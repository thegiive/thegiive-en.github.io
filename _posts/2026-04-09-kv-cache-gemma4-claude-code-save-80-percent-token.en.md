---
layout: post
title: "Cracking the Cache: From Gemma4 to Claude Code, Save 80% on Tokens"
date: 2026-04-09 08:00:00 +0800
permalink: /en/kv-cache-gemma4-claude-code-save-80-percent-token/
image: /assets/images/kv-cache-prompt-cache-paper-figure2.png
description: "Open Claude Code in the morning, type one sentence — and 2–10% of your monthly quota is gone. I ran an experiment locally with Gemma4 and watched prompt processing drop from 31 seconds to 0.25 seconds — a 100x speedup. Then I dug into the Claude Code source and unpacked Anthropic's multi-layer cache architecture: DYNAMIC_BOUNDARY splitting, two-tier TTL, cache-break detection. From the KV cache fundamentals in Transformers, to the MLSys 2024 Prompt Cache paper, to the daily money-saving habits — once you understand the mechanism, the same plan can stretch 3–5x further."
lang: en
---

> 📝 **Translation Note**
>
> This article is translated from the Chinese original. The author writes primarily in Traditional Chinese, and the original version is the canonical source — including the latest updates, comments, and follow-up discussions.
>
> 👉 **Read the Chinese original:** [搞懂快取機制，從 Gemma4 到 Claude Code 省 80% Token](https://ai-coding.wiselychen.com/kv-cache-gemma4-claude-code-save-80-percent-token/)
>
> If you spot any translation errors or have feedback, please refer to the Chinese version as the source of truth.

![Prompt Cache: Modular Attention Reuse for Low-Latency Inference — Figure 2](/assets/images/kv-cache-prompt-cache-paper-figure2.png)

## Why this article matters

The Claude Code [GitHub Issue #38335](https://github.com/anthropics/claude-code/issues/38335) blew up in the past few days — 478 comments. Max 5x users (paying $200/month) reported their quota burning down abnormally fast. Some saw 5% of their quota vanish in 9 minutes of thinking with no file reads or output; others jumped from 40% to 85% after just a few simple commands. Independent security researchers analyzed it and pointed at the root cause: "Regression and cache re-reads with interrupts melt your usage" — the cache mechanism shipped a regression bug, and tokens that should have been low-cost reuses got recomputed at full price.

Prompt cache hit rate can seriously affect whether you get one more round of AI Coding done. Claude Code author Boris Cherny hinted on X that the reason OpenClaw was banned was that OpenClaw's prompt cache was poorly implemented — he even submitted a Claude API cache optimization PR to OpenClaw himself. For today's token-economy industry, this is a big deal.

I took that question to the local desktop and ran experiments with Gemma4 — and discovered that within the same conversation, some turns took 30 seconds while others took 0.2 seconds. To figure out why, I dug from the Transformer attention mechanism all the way down to the Claude Code source code, and found Anthropic has built an elaborate engineering pipeline around caching. Once you understand the mechanism, you'll know how to stretch the same subscription plan 3–5x further.

This is a follow-up in the "[Claude Code Open-Source Design Details](https://ai-coding.wiselychen.com/claude-code-source-leak-memory-architecture-lessons/)" series. Earlier we unpacked the [System Prompt architecture](https://ai-coding.wiselychen.com/claude-code-system-prompt-source-code-analysis/), [security best practices](https://ai-coding.wiselychen.com/claude-code-security-best-practices-source-code-verified/), and [the four-layer compression mechanism](https://ai-coding.wiselychen.com/claude-code-context-engineering-four-layer-compression/). Today, we'll pull the cache layer out and look at it from an IT architecture perspective.

### Reading guide

This is a long post — pick what interests you:

- **Sections 1–3:** The IT architecture of Prompt Cache (if you want the principles)
- **Sections 4–5:** Local experiments and verification (the show-me-the-data crowd)
- **Sections 6–8:** Claude Code best practices + money-saving tips (if you're in a hurry, skip here)

---

## 1. From an IT architecture lens: what problem is Prompt Cache solving?

Architecture first, then experiments.

Every time you say something to Claude Code, what's happening behind the scenes isn't "AI reads your sentence and answers." A complete API request includes:

- System prompt (~20K tokens)
- Tool definitions (~5K tokens)
- The full conversation history (grows with each turn)
- Your latest sentence (~100 tokens)

So the "fix this bug for me" you typed is only 100 tokens, but the model has to process 25K+ tokens. **Every turn, everything before it gets sent again.**

This isn't unique to Claude Code. All LLM APIs are stateless — the server doesn't remember what you said last turn. Every request resends from scratch.

In a world without caching, every turn is full price:

- Turn 1: System prompt 20K + your question 1K = **21K tokens at full price**
- Turn 2: System prompt 20K + history 1K + new question 1K = **22K tokens at full price**
- Turn 3: System prompt 20K + history 2K + new question 1K = **23K tokens at full price**
- ...
- Turn 10: System prompt 20K + history 9K + new question 1K = **30K tokens at full price**
- 10 turns total: **~255K tokens (full price)** ← O(N²) quadratic growth

**255K tokens — but more than 90% of that is repeated content.** That's the problem Prompt Cache is solving: avoid recomputing the same token sequence over and over.

---

## 2. KV cache: an architectural byproduct of Transformer attention

To understand why Prompt Cache works, you need to look at the Transformer attention mechanism. The core formula: **Attention(Q, K, V) = softmax(Q · Kᵀ / √d) · V**

Q, K, V each play a role:

- **Q (Query)** — the current new token: "What am I looking for?" → different every time, can't be cached
- **K (Key)** — historical tokens: "What do I have here?" (the index) → fixed once computed, cacheable
- **V (Value)** — historical tokens: "What's the actual content?" → fixed once computed, cacheable

**KV cache** means storing the Key and Value for historical tokens so the new token only has to compute its own Q and look up the existing KV.

This works because all current mainstream large models (Claude, GPT, Gemini, Llama, Gemma, Qwen) are **decoder-only architectures** — unidirectional attention, where each token only attends to the tokens before it. Once you compute the KV for an earlier token, it's frozen, and adding more tokens after doesn't disturb it.

Causal masking means:

- T₁ can only see itself
- T₂ can see T₁ and itself
- T₃ can see T₁, T₂, and itself — **T₃'s KV is now fixed forever**
- T₄ sees everything before — **adding T₄ doesn't change T₁₂₃'s KV**

If it were bidirectional attention (like BERT), adding a new token would change the representation of every token, killing the cache. That's also why BERT can't do generative AI.

### Is the cache lossless?

**Completely lossless.** Transformer computation is deterministic — KV loaded from cache and KV computed fresh produce identical results. There's no "the cached version is worse" issue.

### Does the generated output go into the cache?

**No.** The KV for output tokens the model produces is discarded after the request — because each generation produces different content (when temperature > 0), so storing them wouldn't help reuse.

But there's a subtle elegance: in the next turn, the previous turn's output gets concatenated back into the prompt and becomes part of the input — so it naturally gets covered by the cache:

- **Turn 1:** Input = system prompt + user "hi". Output = assistant "hello!" ← not cached
- **Turn 2:** Input = system prompt + user "hi" + assistant "hello!" + user "fix my code". The whole prefix is read from cache, **only the final new question is computed at full price.**

The longer the conversation, the higher the cache coverage ratio, the smaller the per-turn computational increment.

---

## 3. The two-layer architecture of Prompt Cache: API cache + inference cache

From an IT architecture standpoint, Prompt Cache actually has two layers. Take the Gemini API as an example — it provides both:

| | API Cache (Context Caching) | Inference Cache |
|---|---|---|
| **Trigger** | Manually marked with `cache_control` | Automatic prefix matching |
| **Lifetime** | 1–24 hours, user-configured | 5 minutes, ephemeral |
| **Best for** | Fixed documents, repeated Q&A | Multi-turn conversation, incremental appending |
| **Discount** | 75% off (Claude / Gemini) | 50–75% off |
| **User intervention** | Explicit marking required | Fully transparent |
| **Local (Ollama)** | N/A | Automatic (but unreliable) |

**Claude Code uses both layers:**

- Explicit `cache_control` marks lock down the system prompt and tools → API cache
- Multi-turn conversation history matches automatically by prefix → inference cache

The two together produce that 90% cache hit rate.

### The academic foundation: the Prompt Cache paper

This mechanism didn't materialize from thin air. There's a 2023 MLSys paper, [Prompt Cache: Modular Attention Reuse for Low-Latency Inference](https://arxiv.org/abs/2311.04934) (In Gim et al.), which proposed a key idea: precompute attention states for reusable text fragments in a prompt (system prompts, templates, documents) and store them server-side, then serve them directly on subsequent requests.

The paper's results:

- **GPU inference latency reduced 8x**
- **CPU inference latency reduced 60x**
- No model parameter changes needed — purely an inference-layer optimization
- Especially effective for document Q&A and recommendation systems

The paper's core idea — modularize repeating prompt fragments, precompute KV, and cache them — is exactly the academic foundation of Claude API's `cache_control` and Gemini's Context Caching. Claude Code's `DYNAMIC_BOUNDARY` design is essentially doing what the paper describes: "split the prompt into cacheable modules."

### The cache is a prefix match: it's a chain — break it anywhere and everything after is dead

This is the most important architectural constraint: **the cache can only match from the beginning.** Any single byte difference in the middle invalidates everything after.

Three cost scenarios:

- **Most expensive:** Block 3 (global static) invalidated → everything is recomputed from scratch — Block 3, Block 4, all messages.
- **Medium:** Block 4 (CLAUDE.md) changed → Block 3 still reusable, but Block 4 and all messages need recomputation.
- **Cheapest:** Just appending new messages → Block 3, Block 4, history all reused, only the new message is computed.

### With cache vs. without cache: cost models compared

Assume system prompt is 20K tokens, each conversation turn adds ~1K tokens.

**Without cache (full price every turn):**

- Turn 1: 21K, Turn 5: 25K, Turn 10: 30K
- 10 turns total: **~255K tokens (full price)** ← O(N²) quadratic growth

**With cache (prefix at 1/10 price):**

- Turn 1: equivalent of 26K (first write, 25% premium)
- Turn 2: equivalent of 3.1K, Turn 5: 3.5K, Turn 10: 3.9K
- 10 turns total: **~60K equivalent tokens** ← approximately O(N) linear growth

**Side by side: 255K vs 60K, the cache saves 76%.** Quadratic growth becomes linear — that's the architectural value of Prompt Cache.

---

## 4. Experimental verification: a 100x speedup with local Gemma4

Enough principles. Time for experiments. I ran Gemma 4 locally with Ollama (8B total params, 9.6GB model), on a 16GB Apple Silicon Mac, and wrote a test script for multi-turn conversation: feed it a 670-token article, then ask 5 follow-up questions in sequence.

Each turn the API returns two key metrics: prompt processing time (digesting input) and generation time (producing the answer):

| Turn | Prompt processing | Generation | Total |
|------|------------|------|--------|
| Turn 1 (feed article) | 24,458ms | 5,095ms (68 tok) | 34s |
| Turn 2 (follow-up 1) | 31,036ms | 22,653ms (365 tok) | 58s |
| **Turn 3 (follow-up 2)** | **253ms** | 2,511ms (46 tok) | 3.8s |
| Turn 4 (follow-up 3) | 203ms | 2,029ms (36 tok) | 3.0s |
| Turn 5 (follow-up 4) | 165ms | 1,870ms (37 tok) | 2.4s |
| Turn 6 (follow-up 5) | 176ms | 1,235ms (26 tok) | 1.8s |

**From Turn 2 to Turn 3, prompt processing dropped from 31 seconds to 0.25 seconds — a 100x speedup.** Meanwhile generation speed stayed steady at 13–20 tok/s, completely unaffected.

This tells us the speedup happens only during the "digest input" phase — exactly matching the KV cache mechanism described above. Turns 1–2 are computing the KV tensors for 670+ tokens layer by layer (60 layers × 670 tokens × 2). From Turn 3 onward, everything is loaded from memory.

### Control group: why don't small models feel it?

I swapped in Qwen3.5 (0.8B, ~1GB) for the same test:

| Turn | Prompt processing |
|------|------------|
| Turn 1 (feed article) | 566ms |
| Turn 2 (follow-up 1) | 173ms |
| Turn 3 (follow-up 2) | 182ms |
| Turn 4 (follow-up 3) | 212ms |
| Turn 5 (follow-up 4) | 227ms |
| Turn 6 (follow-up 5) | 240ms |

Steady ~200ms throughout, no drama. The reason is simple: the small model's KV computation only takes 200ms to begin with — there's not much for the cache to save.

**The bigger the model, the more expensive KV computation is, and the bigger the cache payoff:**

| | Gemma 4 (4.5B active) | Qwen3.5 (0.8B) |
|---|---|---|
| **Cache miss** | ~25,000ms | ~566ms |
| **Cache hit** | ~170ms | ~173ms |
| **Speedup** | 148x | 3.3x |
| **Hit speed** | 3,000–5,000 tok/s | 3,200–3,900 tok/s |

Notice the hit speeds are nearly identical for both models — both reading from memory, the bottleneck is no longer compute, it's I/O.

### Ollama's caching problem: impressive results, unreliable behavior

The experiments also revealed that Ollama's caching is probabilistic — running the same prompt twice yields different cache-hit turns, and after several consecutive hits the cache may suddenly miss (KV evicted under memory pressure).

A stark contrast to Claude API's deterministic caching.

---

## 5. Claude Code's cache engineering: a source-level walkthrough

After using Claude Code to read its own source, I found Anthropic has done a lot of fine-grained engineering around caching — far more than just "automatic caching."

In the previous [System Prompt source code analysis](https://ai-coding.wiselychen.com/claude-code-system-prompt-source-code-analysis/), we mentioned a hidden boundary called `SYSTEM_PROMPT_DYNAMIC_BOUNDARY`. Now let's look at its full design at the cache layer.

### The multi-layer prompt structure

Each API call, what Claude Code sends is a carefully assembled multi-layer structure:

**system (system prompt, ~20K tokens):**

- Block 1: billing attribution header → not cached
- Block 2: CLI prefix → not cached
- Block 3: static instructions (behavior rules, etc.) → **global cache** (shared across all users worldwide)
- ── DYNAMIC_BOUNDARY ──
- Block 4: dynamic content (CLAUDE.md etc.) → **org cache**

**tools (tool schema):** frozen within a session

**messages (conversation history):** `cache_control` is placed on the last message

This design directly maps to the context management discussed in the [four-layer compression mechanism](https://ai-coding.wiselychen.com/claude-code-context-engineering-four-layer-compression/). Caching and compression work together — compression controls context size, caching controls the cost of repeated computation.

Key functions (with source locations):

- `getSystemPrompt()` (prompts.ts:444) — assembles the system prompt
- `splitSysPromptPrefix()` (api.ts:321) — splits at DYNAMIC_BOUNDARY
- `buildSystemPromptBlocks()` (claude.ts:3214) — adds cache_control marks
- `addCacheBreakpoints()` (claude.ts:3064) — places cache breakpoint on last message

### Two TTL tiers

- **Default 5 minutes** — all users
- **Extended 1 hour** — Pro/Max subscribers (not over quota), Anthropic employees

Source code, claude.ts:408–413:

```typescript
userEligible =
  process.env.USER_TYPE === 'ant' ||
  (isClaudeAISubscriber() && !currentLimits.isUsingOverage)
```

### Cache-break detection

Claude Code monitors `cache_read_input_tokens` for each call. If it drops by >5% and the absolute value is >2000 tokens compared to the previous call, it flags this as a break and analyzes the cause: did the system prompt change? Did tools change? TTL expired? Model switched?

### Sub-agent cache isolation

In the [Anthropic dual-agent architecture](https://ai-coding.wiselychen.com/anthropic-dual-agent-architecture-claude-code/) post, we analyzed sub-agent design. Here's a cache-angle observation: **sub-agents almost never reuse the main thread's cache.**

In the source, cache state is tracked separately by `querySource` + `agentId`. Sub-agents differ from the main thread in three critical ways:

1. **Different toolset** — main thread has the full toolset, the Explore agent only a subset. Different tool schema → different cache prefix → everything after the tools section can't be reused.
2. **Completely independent message history** — sub-agents have their own conversation context.
3. **Possibly different model** — sub-agents may use Haiku/Sonnet while the main thread uses Opus. Different model = different weights = completely different KV tensors = zero reuse.

The result:

- **Main thread (Opus):** Block 3 reused, Block 4 + tools reused, messages reused — its own cache chain stays intact
- **Sub-agent (Haiku):** Block 3 not reusable (different model), tools not reusable (different toolset), messages not reusable (independent conversation) — almost starting from zero each time

So every sub-agent you spawn is essentially a "mini cold start." If your CLAUDE.md says "use sub-agents in parallel for everything," be aware each agent has its own independent cache cost.

---

## 6. Claude Code cache best practices

With architecture and mechanism understood, here's the practical playbook.

### Core principle: don't touch the prefix, only append at the end

**Cache-protecting (green light):**

- **Continuous conversation** — prefix stays stable, incremental caching, keep one session going
- **btw** — using `btw` to share a session lets you share the cache
- **CLAUDE.md** — clean it up periodically, but never in the middle of a working session

**Cache-destroying (red light):**

- **Open a new session** — cold cache, ~20K tokens recomputed at full price
- **Modify CLAUDE.md** — Block 4 onward all invalidated, set it once and leave it
- **Add or remove MCP tools** — tool schema change = cache break, configure before the session and disable unused MCP
- **Switch models** — full invalidation, switch in phases, not frequently
- **/compact** — message history changed = break, only use when conversation exceeds 100K
- **Idle past TTL** — cache expired; Pro/Max users, send something within 1 hour

### Quantifying the cache difference

Assume system prompt is 20K tokens, 10 conversation turns:

- **One continuous session:** 1 full-price + 9 at 1/10 = **1.9 full-price units**
- **Open new session each time:** 10 full-price = **10 full-price units**

**5x difference.** For Pro/Max subscribers, this means the same plan can do 3–5x more work.

### The hidden cost of switching models

Switching models is a complete invalidation — Opus and Sonnet have different weights, so KV tensors aren't interchangeable. One model switch means 50K tokens of context recomputed at full price. If you switch back within the TTL you may still hit the old cache (`promptCacheBreakDetection.ts` tracks `modelChanged`).

**Recommendation: don't switch models in the last half hour before clocking out.** Switch by phase, not frequently.

---

## 7. Advanced trick: cache keep-alive

Pro/Max users have a 1-hour TTL. Lunch takes 1.5 hours and your cache expires; a long meeting and your cache expires.

**Principle:** TTL refreshes on each cache read. Just send a request matching the prefix before expiry, and the cache lives indefinitely.

**Concept:** Use tmux or iTerm2 AppleScript to send a prompt to the Claude Code terminal every 55 minutes:

```
Did I disconnect? If not, just say ok.
```

Does this consume some tokens? Yes. But way cheaper than a 20K-token cold-start full-price recomputation — you trade 1 token of output for 20K tokens of cache value.

A note: Anthropic has abuse-detection mechanisms. Automating keep-alive too mechanically (precisely every 60 seconds, no other activity) theoretically risks being flagged. A 55-minute manual click is fine — the key is "reasonable use." I don't recommend full automation.

---

## 8. Connecting to previous articles

This article's cache mechanism, together with the prior Claude Code series posts, forms a complete architectural picture:

| Article | Layer | What it solves |
|------|--------|-------------|
| [System Prompt source analysis](https://ai-coding.wiselychen.com/claude-code-system-prompt-source-code-analysis/) | Context layer | How prompts are assembled, how DYNAMIC_BOUNDARY splits |
| [Security best practices](https://ai-coding.wiselychen.com/claude-code-security-best-practices-source-code-verified/) | Control layer | Permissions, sandboxing, hook governance |
| [Four-layer compression](https://ai-coding.wiselychen.com/claude-code-context-engineering-four-layer-compression/) | Context layer | What to do when context gets too long |
| **This post: Cache mechanism** | **Cache layer** | **How to save on repeated computation** |

In the architecture/governance/engineering practice posts, I've decomposed Claude Code into six layers: context, control, tools, execution, cache, validation. This post pulls the cache layer out by itself — from the underlying KV cache principles, to Anthropic's engineering implementation, to the daily habits you should adopt when using Claude Code.

---

## Closing

Caching isn't black magic — it's economics:

- **KV cache:** trade memory for compute, historical tokens computed once aren't recomputed
- **Prefix matching:** trade structural rigidity for reusability, fixed prefix is what enables reuse
- **TTL management:** trade discipline for savings, keeping the conversation continuous *is* saving money

From the 100x speedup of local Gemma 4 experiments to the elegance of `DYNAMIC_BOUNDARY` in the Claude Code source — once you understand the principles, you don't need any plugin or tool. Just a few good habits — keep your sessions continuous, don't randomly modify CLAUDE.md, don't open new windows on a whim — and your Claude Code subscription will go 3–5x further.

---

## References

**Paper:**

- In Gim, Guojun Chen, Seung-seob Lee, Nikhil Sarda, Anurag Khandelwal, Lin Zhong. [Prompt Cache: Modular Attention Reuse for Low-Latency Inference](https://arxiv.org/abs/2311.04934). MLSys 2024.

**Series:**

- [Claude Code 510K-Line Source Leak: Architecture Lessons](https://ai-coding.wiselychen.com/claude-code-source-leak-memory-architecture-lessons/)
- [Decoding Claude Code's System Prompt Source](https://ai-coding.wiselychen.com/claude-code-system-prompt-source-code-analysis/) (EP1)
- [Claude Code Security Best Practices: 14 Rules](https://ai-coding.wiselychen.com/claude-code-security-best-practices-source-code-verified/) (EP2)
- [Claude Code Context Engineering: Four-Layer Compression](https://ai-coding.wiselychen.com/claude-code-context-engineering-four-layer-compression/) (EP3)
- [Anthropic's Official Reveal: Dual-Agent Architecture](https://ai-coding.wiselychen.com/anthropic-dual-agent-architecture-claude-code/)
