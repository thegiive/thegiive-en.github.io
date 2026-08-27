---
layout: post
title: "Pi's 99.93% Cache Hit: $2.65 for 1 Billion Tokens — One More Number to Watch When Choosing Your Agent"
date: 2026-08-25 09:00:00 +0800
permalink: /en/pi-cache-hit-99-93-context-compression-roadmap/
image: /assets/images/pi-cache-hit-99-93-cover.png
image_alt: "Pi agent harness homepage"
author: "Wisely Chen"
category: AI Agent
tags:
  - Pi
  - Cache Hit Rate
  - Agent Harness
  - DeepSeek V4 Flash
  - Context Compression
description: "Someone ran nearly 1 billion input tokens through Pi + DeepSeek V4 Flash with a 99.93% cache hit rate, paying just $2.65. Without cache, the same workload costs ~$132. This article traces Pi's five cache-friendly disciplines back to source code and official docs, then maps five divergent context compression approaches in the Pi ecosystem."
lang: en
---

> **Translation Note**
>
> This article is translated from the Chinese original. The author writes primarily in Traditional Chinese, and the original version is the canonical source — including the latest updates, comments, and follow-up discussions.
>
> **Read the Chinese original:** [Pi 99.93% Cache Hit：10 億 token 只花 $2.65，Agent 選型要多盯一個數字](https://ai-coding.wiselychen.com/pi-cache-hit-99-93-context-compression-roadmap/)
>
> If you spot any translation errors or have feedback, please refer to the Chinese version as the source of truth.

---

## A Guaranteed-Boring Preface

Lately I've been studying Pi's harness design, and the deeper I dig, the more it feels like cracking open an undergrad OS textbook — cache locality, write-ahead logs, garbage collection, eviction policies, all the classics. The only difference is that instead of managing rows and pages, you're managing conversations and decisions. (For CS majors, "the dinosaur book" refers to the iconic OS textbook whose cover features a different dinosaur on every edition. Citing it in the age of AI carries a certain self-aware irony — maybe we're the dinosaurs now.)

## First Time Coding with an Open-Source Model — It Didn't Go Well

Another guaranteed-boring IT day.

A while back I tried using Codex with an open-source LLM for coding for the first time. Honestly, it wasn't great: changes that should have been made were missed, edits contradicted each other, and anything beyond a short session led to the model forgetting what we'd discussed earlier. I didn't rush to a conclusion because I wasn't sure where the blame lay — the model itself, the prompting, or the harness.

Then I switched to Pi, which the community had been championing. Same model, night and day. On the same tasks, it suddenly remembered which files it had read, what it had changed, and where it got stuck; long sessions didn't visibly degrade. And costs — when running through the API — were absurdly low.

This wasn't an isolated experience. Someone ran Pi with DeepSeek V4 Flash, churning through nearly 1 billion input tokens with a 99.93% cache hit rate — bill: $2.65. The same volume without caching would have cost roughly $132. The same model on other harnesses typically sees cache hit rates between 94% and 97%; Pi holds above 99%. Those few percentage points, at scale, translate to exponential cost differences.

So what this article does is straightforward: take "what exactly is Pi doing right," break it down, and trace each piece back to source code and official documentation. No reading-group impressions.

## First, Understand Prefix Cache: Change One Comma, Recompute Everything After It

What does a 99.93% cache hit rate actually mean? Let's start with the arithmetic at the bottom layer. I unpacked the mechanism in detail in [Understanding Cache Mechanics: From Gemma4 to Claude Code, Saving 80% of Tokens](https://ai-coding.wiselychen.com/kv-cache-gemma4-claude-code-save-80-percent-token/) (why KV cache is prefix-structured, Claude Code's caching engineering), and in [DeepSeek V4 Flash's Disk KV Cache](https://ai-coding.wiselychen.com/deepseek-v4-flash-disk-kv-cache-50x-economics/) I showed how the math works out to a 4x bill difference between 90% and 99% hit rates. I won't repeat all of that here — just the part that matters for Pi.

LLMs process information in strict linear order: if the first half of this request's prompt is **byte-for-byte identical** to last time, that portion can skip computation — the model serves pre-computed results straight from the KV cache, at zero or near-zero cost. That's a "prefix cache hit."

But change one character in the prefix — even adding a single comma — and everything from that point to the end is treated as brand-new input, billed at full price. One full-price recomputation can easily run tens of thousands of tokens.

So the entire optimization boils down to a single question: **How stable is the prefix of the prompt you're sending to the model?**

Pi holds at 99.93%. Other harnesses commonly land between 94% and 97%. Those few missing percentage points, at volume, create an exponential gap — $2.65 versus $132 are two points on the same curve.

The community's first instinct was: is Pi using some black magic, some proprietary caching algorithm? After going through Pi's source code and official documentation end to end, the answer is embarrassingly mundane: **It doesn't use any new caching technique. It's all engineering discipline — don't touch the prefix.**

## Pi's Six Disciplines: Every One Is About "Touching the Prefix Less"

A kitchen metaphor ties them together: the prefix is the kitchen's pots, pans, stove layout, and fridge placement. No matter what new orders come in each evening, the head chef never rearranges the entire kitchen for one new dish. Rearranging the layout = changing the prefix = full-price recomputation. The only thing the chef does is bring new ingredients in through the back door.

Pi's six disciplines all protect the kitchen from being rearranged:

**One: Only 4 default tools.** The system prompt construction code is blunt about it — default tools are just read, bash, edit, and write. No grep, no find, no ls — when needed, the agent runs them through bash. Tool definitions and descriptions sit at the very front of the prefix; every additional tool is another chunk of "prefix that might change." Many harnesses declare every tool upfront. Pi goes the opposite direction: one fewer tool means one more degree of prefix stability, and that stability compounds **on every single request**.

**Two: Thin system prompt.** The default prompt is just a few dozen lines. A lot of developers these days love writing multi-thousand-word prompts covering every conceivable behavior rule. But system prompts, tool definitions, and AGENTS.md — change one character in any of them and the cache prefix breaks. Pi's design choice isn't to make the prefix "comprehensive"; it's to make it "rarely change."

**Three: Sessions are append-only.** Each turn appends new messages after the existing context, verbatim. It doesn't reconstruct a fresh "system config + conversation history" prompt every turn. This is the easiest discipline to overlook, but it's the physical foundation for 99%+: the hundreds of thousands of tokens that came before are bolted to the floor like a kitchen counter, locked in cache. The only thing the model has to pay full price for is the sliver of new instructions appended at the end.

**Four: Skills loaded on demand.** In the default state, the system prompt contains only each skill's name plus a one-line description — like a recipe shelf that shows only the table of contents, not every cookbook opened flat and taking up space. Only when the agent actually needs a skill does it use read to pull in the full text. This way, unused skills don't bloat the prefix. Compare this to harnesses that stuff thousands of tokens of instructions at startup — the savings aren't about "having versus not having," they're about "how clean the prefix stays."

**Five: Compaction runs in a separate routing session — it doesn't pollute the main line.** This is the easiest one to miss. Most harnesses handle compaction by telling the LLM to "summarize when context is full" — that one-off summary request goes straight into the main session, and the prefix breaks on the spot. Pi's official documentation states explicitly: compression and branch summary requests use a **new routing session** (a separate process), and when the provider supports it, **prompt cache write is actively disabled**. The one-off summary artifact is used once and discarded — it never contaminates the main kitchen.

**Six: Switching branches has a second type of summary, equally non-polluting.** When you use /tree to jump to another branch, Pi can ask whether you want to condense the branch you're leaving into a context summary to carry over (you can turn this off). Same structured format, same file tracking — same routing session, same cache-write disabled. So "memory" in Pi isn't a linear conversation summary; it's a **structured state snapshot stored at every fork in the tree**.

Add all six together and the source of 99.93% becomes clear. Notice: not a single one is a "caching technique." They're all discipline.

## The Session Tree: A Free Time Machine

Many people assume agent conversations are a straight line, like a chatbot. Pi's underlying data structure is actually **a tree**: storage is JSONL, one entry per line, linked by id and parentId. Every conversation turn, tool call, or attempted code change is an independent node on the tree.

This answers the most common objection to append-only: if you only add and never delete, doesn't the AI get permanently stuck in a wrong context when it goes down a bad path (say, writing hundreds of lines of scraping code before discovering the library doesn't support the feature)?

No. You don't need to manually delete previous wrong conversations or save-as a new file — use /tree to go back to the node before the wrong turn and grow a new branch from there. And because everything before that node **still has a perfectly unchanged prefix**, the cache remains valid. The cost of a wrong turn is limited to the wrong branch itself.

**Window management comes down to two numbers.** The model window has a fixed reservation of 16,384 tokens for the LLM's response — non-negotiable breathing room that prevents the AI from being cut off mid-sentence (compression triggers when context exceeds "window minus 16,384"). Another 20,000 tokens are reserved as the "recent messages" allowance during compression — ensuring the AI maintains clear short-term working memory of the current task while summarizing sprawling history. Both are adjustable in settings, so Pi's context management isn't "deal with it when it's full" — it leaves a buffer before taking action.

## Five-Step Compression: Cut Points Must Preserve Action Integrity

When /context fills up (or you manually /compact), here's what actually happens:

1. **Find the cut point.** Count backward from the newest messages until you hit 20,000 tokens and draw a line. The cut point is extremely particular: it can only fall on a user message, assistant message, bash execution, or custom message — **never in the middle of a code read or tool execution result**. Results must stay on the same side as their tool call. Everything before the line gets summarized; everything after is preserved verbatim.
2. **Generate a structured summary.** Old messages are serialized (tool results trimmed to 2,000 characters, since read and bash output are usually the largest chunks in context), then the LLM fills in a fixed format: Goal, Constraints, Progress (Done / In Progress / Blocked), Key Decisions, Next Steps, Critical Context, plus two lists — files read and files modified.
3. **The summary is written back as a new entry.** It doesn't modify old data; it appends a CompactionEntry. The next request sent to the LLM is: system prompt + summary + messages after the cut point. The compressed old messages haven't disappeared — they're still sitting intact in the JSONL file, just no longer in the submitted context.
4. **Repeated compression is iterative.** On the second compression, the previous summary is passed in as input, and the new summary is "old summary + newly old messages" compressed again. File tracking is cumulative too — each round merges the read/modified lists from the previous summary, so after five rounds, "which files this session touched" never gets lost.
5. **The summary request itself doesn't enter the main session's cache.** It uses a new routing session, with cache write disabled when the provider supports it — this is what the fifth discipline looks like in practice.

One easy-to-miss detail: if a single turn by itself exceeds 20,000 tokens (say, a read of several thousand lines), the cut point falls in the middle of that turn, and Pi generates two summaries that get merged — one for the prior history and one for the beginning of that turn. So you never end up with "an entire turn compressed away with the agent having zero knowledge it happened."

## The Rashomon of Agent Memory: Five Approaches, No Consensus

Built-in compaction is the most passive option: it only acts when context fills up. Across the open-source ecosystem, "how should AI remember" has no consensus. I've cataloged five approaches (some names can't be found as exact GitHub projects, so I'll describe the ideas): pai-acp lets the agent decide which history to pre-compress; pi-smart-compact keeps only a sticky note — goal, modified files, unfinished items; pi-context treats context like Git with rollback, diff, and revert; Hypa filters at the gate so irrelevant information never enters in the first place; pi-press computes summaries in the background, ready to swap in when compaction actually triggers.

All five sound reasonable in their own way. But there's a massive trade-off hiding here — pi-vcc, the most-starred on GitHub (267 stars), takes a sixth path: no LLM summarization at all, pure syntactic extraction and formatting, claiming lossless compression. The thesis: "Summarization always loses information, so don't summarize — just reorganize."

Lossless? Sounds too perfect — if LLM summaries inevitably lose detail and hallucinate, why doesn't everyone just install pi-vcc and throw out the other five?

**Because it preserves actions but loses semantics.** Pure syntactic extraction knows what the code was changed to, but "why was this line changed in the first place" — the decision logic, the discussion process — all gone.

And don't get greedy. If you get greedy and install gate filtering, sticky notes, AND pi-vcc lossless extraction into the same agent — the AI will descend into severe logical confusion. These plugins fight furiously under the hood over "who gets to decide what information stays," constantly overwriting each other's state, and eventually wrecking everything.

So when choosing, establish a hierarchy: **Gate filtering at the base layer (less garbage in), sticky notes as the primary compression strategy (preserve working state), version control as the last safety net (rollback capability).** Pre-async and the forgetting approach are tuning layers. Ask five people "what did the agent remember" and you'll get five answers — this is the same unsolved problem I wrote about in [Agent Memory Benchmark's Rashomon Phenomenon](/agent-memory-benchmark-rashomon-filesystem-wiki/). Now the answer has become five approaches, and all five actually work.

## Pi's Philosophy: Keep the Core Light, Let Capabilities Build Upward

Looking back, the six disciplines are really the same decision projected onto different surfaces: Pi's core is deliberately kept thin — it doesn't build in anything that merely "looks useful." But the ecosystem is growing — sub-agents, sandboxes, memory, plan mode, remote sessions keep appearing one by one, and the community conversation becomes "should Pi add this to core?"

That question is slightly mis-framed. The real question is: which layer should this capability live on? I break it into three:

**Foundational capabilities go into Core.** Things that can't run without — session management, context assembly, tool-call loops, compaction. Pi's Core currently does all of this, and does it cleanly.

**Rules and standards — Core defines the floor.** Permission models, remote session protocols, inter-sub-agent communication formats. Leave this layer for each extension to design on its own and the result is a different spec per project, an ecosystem that fragments as it grows. Pi needs to at least define the interfaces and minimum protocols; implementations can let a hundred flowers bloom, but the vocabulary must be unified.

**Gameplay stays with Extensions, always.** Whether memory uses files or a vector store, how many steps plan mode takes, how sub-agents divide labor, which search tool to use — these are preference questions, not capability questions. My position is clear: Pi should never decide these for users, not today, not three years from now.

Going through Pi's official documentation and the READMEs of several major extensions, this layering isn't my invention — the official stance is "features that aren't needed don't enter core." This also explains why the six disciplines can exist: daring to keep only 4 default tools is possible because "more" was never the goal. In one sentence: **If it can be solved with an extension, don't touch the core. Lock down the core's boundaries and leave the gameplay decisions to extensions.** This is also the hardest thing for Pi to hold — feature requests only get louder, and holding the line on "not adding" takes more resolve than shipping features.

## Conclusion: Prefix Stability Is an Iron Law — Pi Just Enforces It Most Thoroughly

Pi has no black magic. Its strength is "less." Fewer tools, thinner prompts, append-only context, on-demand skills, compression that doesn't pollute the cache — all discipline, not technique. Lightweight isn't a feature deficit; it's a deliberate choice for maximum performance. This explains why "switching to Pi suddenly made things so much better": what changed wasn't just the interface — it's that what gets sent into the model on every request became cleaner, so the same model naturally performs better.

But before slapping 99.93% onto your own cost estimates, think about two things.

**First, that number is the product of DeepSeek and Pi together.** 99.93% comes from DeepSeek's underlying architecture combined with Pi's software discipline in perfect harmony. DeepSeek uses disk-based KV cache, so cached entries can survive for a very long time — potentially days. Anthropic, optimizing for response speed, keeps its prompt cache in RAM — expensive, with a TTL of just 5 minutes. Get up to pour yourself a cup of water, come back, and the prefix has evaporated — that next message pays full price for all those tens of thousands of tokens that came before. Switch to Anthropic and the general direction still holds (stable prefixes still save money, rewrite cost after a break is low, breaks happen less often), but that extreme savings margin you probably won't see — the hit-rate ceiling is inherently lower.

**Second, "save tokens" is the most common optimization trap.** qwen-code added a dynamic tool-loading feature — it only sends tool descriptions when the model actually needs them. In a no-cache mindset this sounds perfectly reasonable: fewer tokens sent, less money spent. Result: every request's prefix looked different, and the hit rate cratered from 97.5% to 81.5%. Surface metric: per-request token count dropped 46%. Actual bill: a task that cost $1.05 ballooned to $3.30. Costs more than tripled. Any clever trick that claims to "save tokens" but requires frequent prefix changes will teach you an expensive lesson on the API bill.

So the real conclusion fits in one sentence: **Prefix stability is an iron law that every harness should observe today, and Pi is simply the one that enforces it most thoroughly.** When choosing an agent, put cache hit rate right next to model intelligence scores — this number doesn't appear on any leaderboard, you have to pull your own logs: Pi's footer gives it directly (cache read / write / per-turn hit rate), Claude Code's logs live under `~/.claude/projects/`, and my earlier piece [17 Billion Tokens Cost Breakdown](/ai-coding-token-cost-calculation-cache-read/) explains the method. Don't blindly trust anyone else's test numbers, including this article's.

## Cutting-Edge Technology Requires the Most Fundamental Knowledge

Looking back, this entire article really only says one thing: the engineering discipline of agent harnesses is the same old discipline — it never changed. Prefix stability is cache locality. The session tree is a write-ahead log. Compaction is garbage collection. The five compression approaches map to choices among eviction policies. The more I study agent harness design lately, the more it looks like database engineering, like OS design — the only difference is that instead of managing rows and pages, it manages conversations and decisions.

Whether it's done well doesn't depend on how smart the model is. It depends on how well the harness author knows their dinosaur book.

---

## FAQ

**Q: I use Claude (Anthropic API) — does Pi's advantage still apply?**

The direction holds, but the magnitude narrows. Anthropic's prompt cache TTL is 5 minutes — step away for lunch and your prefix is gone; the hit-rate ceiling is inherently lower than DeepSeek's. But the discipline of prefix stability still saves money — rewrite cost after a break is low, and breaks happen less often. Pull your own logs to see actual hit rate; don't directly apply the 99.93% figure.

**Q: "Other harnesses" at 94-97% — which ones specifically?**

My understanding is harnesses in the "many default tools, frequently changing prefix" category (those that declare all tools upfront, rebuild the prompt every turn). This isn't saying any particular one is doing a bad job — it's saying this metric needs to be measured per-harness, not assumed.

**Q: Do /compact and /tree interfere with each other?**

No. Both only append new entries (CompactionEntry or BranchSummaryEntry) without modifying old data. Summaries can reference each other — a branch summary can compress an already-compacted branch, and file tracking is cumulatively merged. The complete history always remains in the JSONL file.

**Q: Can the five compression approaches be stacked?**

Yes, but with a hierarchy: gate filtering at the base, sticky notes as the primary strategy, version control as the safety net, pre-async and forgetting as the tuning layer. Installing two plugins that both "decide what to keep" will cause them to overwrite each other.

**Q: Should I switch from subscription to API?**

Check the API-equivalent cost. Pull your actual API usage logs from the past two weeks and calculate using official pricing: if it exceeds three times your subscription cost, go back and keep paying the monthly fee; under three times, API is not only cheaper but gives you more flexibility. I calculated this in my late-August piece: my API bill was $631-681/month against a $200 subscription — more than three times, so the subscription wins.

---

## Reality Check

- **$2.65 vs $132 is one person's self-reported figure.** There's no information about session count, workload characteristics, or cache write costs. A billion input tokens corresponds to a long chain of extended sessions, not a "turn it on and watch it go" situation.
- **94-97% is an observed "common range for other harnesses," not a benchmark.** Individual harness test numbers weren't published. The direction is almost certainly correct (most harnesses have more default tools and more frequently changing prefixes), but whether "Pi holds 99%+" applies equally on Anthropic has not been answered by anyone.
- **Of the 5 compression approaches, I only traced pi-smart-compact and pi-vcc back to repos.** pai-acp, pi-context, Hypa, and pi-press couldn't be found as exact projects on GitHub — they may be small-scale, private, or not yet open-sourced. The conceptual framework is based on community descriptions, not reading each one's source code.
- **DeepSeek V4 Flash's pricing was raised on August 16.** I wrote about that already. $0.0028/$0.14 is the post-increase pricing; it's unclear which pricing era the $2.65 bill reflects.

---

**Sources**

- [0xEvan: 99.93% cache hit on DeepSeek v4 flash with pi harness](https://x.com/EvanDeKim/status/2086681823216546294) (2026-08-10)
- [Pi Official Docs: Compaction](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/compaction.md)
- [Pi Official Docs: Session Format](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/session-format.md)
- [Pi Source: system-prompt.ts (default tools)](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/src/core/system-prompt.ts)
- [pi-smart-compact](https://github.com/alpertarhan/pi-smart-compact) (30 stars)
- [pi-vcc](https://github.com/sting8k/pi-vcc) (267 stars)
