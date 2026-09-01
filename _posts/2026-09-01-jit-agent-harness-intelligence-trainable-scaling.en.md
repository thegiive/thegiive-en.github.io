---
layout: post
title: "NUS Trained a Model Whose Only Job Is Writing Harnesses: JIT-Agent Lets DeepSeek-V4-Flash Beat GPT-5.6 by 9 Points on DeepSearchQA at Half the Token Cost"
date: 2026-09-01 09:00:00 +0800
permalink: /en/jit-agent-harness-intelligence-trainable-scaling/
image: /assets/images/jit-agent-harness-intelligence-cover.png
image_alt: "JIT-Agent paper header showing leaderboard results across four agent benchmarks"
author: "Wisely Chen"
category: AI Industry Analysis
tags:
  - Harness Engineering
  - Agent
  - JIT-Agent
  - Scaling
  - Open Source
description: "NUS LV-Lab's JIT-Agent paper pushes Harness Engineering from craft to science: they trained a 27B model (based on Qwen3.6) to generate task-adaptive agent harnesses on the fly using a four-module protocol (Memory / Planning / Capability Orchestration / Action). Results: DeepSeek-V4-Flash with a JIT-Agent harness scores 85.1 on DeepSearchQA vs GPT-5.6's 76.0 (+9.1); GLM-5.2 gains up to +20.2 points; across nine benchmarks, JIT-Agent matches mature hand-crafted harnesses like Claude Code and OpenCode while using 50% fewer tokens and 36% less cost. This is the first time someone has treated harness-writing ability as an independent, trainable scaling dimension — orthogonal to model scaling."
lang: en
faq:
  - question: "How much compute does JIT-Agent itself require?"
    answer: "Based on Qwen3.6-27B, it needs one A100 or equivalent GPU for inference. Harness generation time isn't explicitly reported, but from token counts it should be seconds to a few minutes — orders of magnitude faster than humans hand-tuning harnesses over hours or days."
  - question: "Can I use JIT-Agent to generate a better harness for Claude Code?"
    answer: "In theory yes, but in practice it doesn't make much sense. Claude Code's harness is deeply integrated into its runtime — you can't easily swap out its memory management or tool filtering logic. JIT-Agent is better suited for scenarios where you're calling an LLM via API and need to build your own agent scaffold."
  - question: "How is this different from the Agents-A1 '35B beats 1T' paper?"
    answer: "Agents-A1 shifted the scaling axis from parameters to training horizon — using longer trajectory data to train a smaller model. JIT-Agent shifts the scaling axis to harness — not changing the model itself, but changing the scaffold around it. Both papers attack the same problem (agent capability doesn't come from model size alone), but from different angles."
---

> **Translation Note**
>
> This article is translated from the Chinese original. The author writes primarily in Traditional Chinese, and the original version is the canonical source — including the latest updates, comments, and follow-up discussions.
>
> **Read the Chinese original:** [NUS 訓練出一顆「專門寫 Harness」的模型](https://ai-coding.wiselychen.com/jit-agent-harness-intelligence-trainable-scaling/)
>
> If you spot any translation errors or have feedback, please refer to the Chinese version as the source of truth.

We've been talking about Harness Engineering on this blog for six months. From the [three migrations of harness center](/agent-harness-three-migrations-mechanism/) to [swapping Kimi K3's harness for a 10-point swing](/local-first-model-needs-local-first-harness/) to [GPT-5.6 Sol gaining 25 points on ARC-AGI-3 with two config changes](/arc-agi-3-harness-retained-reasoning-compaction/) — the conclusion has always been the same: the ceiling on agent capability isn't the model, it's the harness.

Now NUS's LV-Lab has turned that claim into a [paper](https://arxiv.org/abs/2608.25593), and they've done something we haven't: **they trained the ability to write harnesses into a model itself.**

Not a human writing CLAUDE.md. Not a human designing skills. A 27B model that reads a task description and synthesizes a complete agent harness on the fly — how to manage memory, how to decompose planning, which tools to expose, how to run the action loop — then applies it to any off-the-shelf LLM.

How dramatic are the results? DeepSeek-V4-Flash with a JIT-Agent harness scores 85.1 on DeepSearchQA. GPT-5.6 on the same benchmark: 76.0. **A cheap model with an auto-generated harness beat the expensive model by nine points.**

---

## 30-Second Overview

| Item | Details |
|------|---------|
| Paper | [JIT-Agent: Scaling Harness Intelligence via Just-in-Time Harness Evolution](https://arxiv.org/abs/2608.25593) |
| Institution | NUS Learning and Vision Lab (LV-Lab) |
| Date | 2026-08-26 |
| Core claim | Harness intelligence is a trainable, transferable scaling dimension orthogonal to model scaling |
| Method | Train a 27B model (based on Qwen3.6) to generate task-adaptive harnesses on the fly |
| Key numbers | DeepSeek-V4-Flash + JIT: DeepSearchQA 85.1 (GPT-5.6 = 76.0); GLM-5.2 up to +20.2 points; cost -36% |
| Open source | [GitHub](https://github.com/bingreeky/JIT) ・ [Hugging Face](https://huggingface.co/JIT-Agent) |

---

## 1. Why Harness-Writing Deserves to Be "Trained"

The way we write harnesses today is fundamentally a craft. You write a CLAUDE.md, design a few skills, iterate on prompts, test, adjust. This works fine for personal projects, but it has three structural bottlenecks:

1. **Not transferable.** A harness you wrote for Project A almost certainly needs rewriting for Project B. Different tasks, different models, different tool sets — different optimal configurations.
2. **Not scalable.** How many harnesses can one engineer tune per day? 10? 100? Each one requires understanding the task, choosing strategies, and testing outcomes.
3. **No self-improvement.** The harness you write today won't get better because you ran it a thousand times. You have to come back, read the logs, change the config, and re-run.

JIT-Agent breaks all three at once: one model generates harnesses, synthesizing them on the fly for new tasks, distilling results back into its archive during training so similar future tasks get better starting points.

---

## 2. The Four-Module Protocol: h = (M, P, A, F)

The paper's first move wasn't training a model — it was **defining what a harness actually looks like.**

This sounds obvious, but in practice it's chaos. Open different companies' agent frameworks and you'll find some that merge memory with planning, some with no tool orchestration at all, some that cram everything into a single system prompt. Without a unified structure, machine generation is impossible.

JIT-Agent decomposes a harness into four modules with a fixed dependency order:

**M → P → F → A**

| Module | Responsibility | Maps to our practice |
|--------|---------------|---------------------|
| **M (Memory)** | Manages event history and mutable state, produces a compressed "view" | Context management rules in CLAUDE.md, memory systems |
| **P (Planning)** | Produces local directives and subgoals from task description + state + memory view | Task decomposition, plan mode |
| **F (Capability Orchestration)** | Filters which tools to expose from the registry based on current directive and state | Skill design, tool filtering |
| **A (Action)** | Executes the control loop, emitting tool calls or terminal outputs, updating state | The agent's main loop itself |

The elegance is that **every module is a swappable component.** Research-heavy deep search tasks might need a recursive decomposer for P; document processing might need a DAG-based P; simple tasks might not need P at all (set to null). But regardless of what's plugged in, the four-slot interface is fixed.

This is what makes machine generation possible: JIT-Agent doesn't need to write an entire agent framework from scratch. It just picks an implementation for each of four slots and assembles them into a complete harness.

---

## 3. Three Training Stages: Imitate, Repair, Evolve

JIT-Agent is based on Qwen3.6-27B, trained in three stages, each solving a different problem.

### Stage I: Customization (Learn to write harnesses)

A frozen strong teacher model generates protocol-compliant harnesses; JIT-Agent learns to imitate via SFT. Then DPO learns preferences from harness pairs — not just reward, but multi-objective advantage across performance, latency, and cost.

The seed bank starts with just 13 human-written harnesses. Each generation samples 3 task-type-matched references as context.

### Stage II: Repair (Learn to fix broken harnesses)

Some fraction of Stage I harnesses break — compiler errors, interface mismatches, runtime failures. The conventional approach is to discard these. JIT-Agent flips it: failed cases **plus error diagnostics** become training data, teaching the model to fix a broken harness within two rounds.

This is extremely practical. We've all hit the "design looks right, blows up at runtime" scenario when writing harnesses. The difference is humans have to debug manually; JIT-Agent bakes in the debug capability.

### Stage III: Evolution (Self-improvement during training)

This is the most interesting part. JIT-Agent maintains a **continuously expanding harness archive** (starting from 13 seeds):

1. New task arrives; retrieve matched reference harnesses from the archive
2. Sample G candidate harnesses from the current policy
3. Execute all candidates; measure reward / latency / cost
4. Compare against the best incumbent in the archive
5. Update the policy via PPO
6. **Archive entry threshold:** performance at or above the frontier, AND at least one efficiency axis (latency or cost) strictly improved

This evolution loop happens during **training** — Evo-GDPO is an online RL loop requiring sampling, execution, reward computation, and weight updates. After deployment, JIT-Agent uses the archive and learned policy from training to generate harnesses — it's not a "gets better with use" runtime mechanism. But the larger and more diverse the training-time archive, the richer the reference pool at inference.

---

## 4. The Numbers: Cheap Model Beats Expensive Model Across Nine Benchmarks

### Nine-benchmark average

| Configuration | Avg score | Delta from base model |
|--------------|-----------|----------------------|
| GLM-5.2 alone | 74.1 | — |
| **JIT + GLM-5.2** | **81.8** | **+7.7** |
| DeepSeek-V4-Flash alone | 66.7 | — |
| **JIT + DeepSeek-V4-Flash** | **75.5** | **+8.8** |
| GPT-5.6 (estimated) | ~76.5 | — |

### Where JIT + DeepSeek-V4-Flash surpassed GPT-5.6

| Benchmark | JIT + DSV4-Flash | GPT-5.6 | Delta |
|-----------|-----------------|---------|-------|
| DeepSearchQA | **85.1** | 76.0 | +9.1 |
| PinchBench | **92.9** | 84.2 | +8.7 |
| OdysseyBench | **73.0** | 68.7 | +4.3 |

Largest single gain: DeepSeek-V4-Flash on DeepPlanning-Shopping went from 59.1 to **83.9** (+24.8).

### Head-to-head with Claude Code and OpenCode

Using DeepSeek-V4-Flash as backbone:

| Benchmark | Claude Code | OpenCode | JIT-Agent |
|-----------|-----------|----------|-----------|
| DeepSearchQA | 79.6 | 75.9 | **85.1** |
| xBench-DS | 75.0 | 65.0 | **82.0** |
| AgentIF | **66.9** | 48.1 | 63.8 |
| DSQA tokens | 625K | 1,832K | **400K** |
| DSQA cost | $0.088 | $0.258 | **$0.066** |

**The pattern is clear:** JIT-Agent dominates on search and research tasks with dramatically lower token usage and cost. But on open-ended work tasks (AgentIF), Claude Code still leads — 3.1 points isn't huge, but the direction is consistent.

This validates what we've been saying: Claude Code's harness is hand-crafted and deeply optimized for general software development. On its home turf — open-ended tasks requiring heavy judgment calls — auto-generated harnesses can't catch up yet. But on more structured tasks, machine-generated harnesses already win.

---

## 5. How This Paper Connects to Six Months of Harness Engineering Coverage

Let's start with the similarities.

The paper's core formula **h = (M, P, A, F)** maps remarkably well to what we do in practice:

- **M (Memory)** → The context management section of our CLAUDE.md, memory system design, controlling what history enters context
- **P (Planning)** → Plan mode, task decomposition
- **F (Capability Orchestration)** → Skill design, deciding which tools get exposed in which scenarios
- **A (Action)** → The agent's execution loop, which we typically don't touch (provided by Claude Code / Cursor runtimes)

The paper's contribution: **turning what we do by experience into a process with explicit interface definitions that can be machine-generated and auto-optimized.**

Now the differences.

Our harnesses are **persistent** — a single CLAUDE.md might live for months, evolving slowly with the project. JIT-Agent's harnesses are **ephemeral** — generated fresh for each task, discarded after use (only the best get archived as training references).

These two modes don't conflict; they complement each other. Persistent harnesses suit familiar, repetitive workflows. Just-in-time harnesses suit new tasks, new models, new tool sets — scenarios where you don't want to spend three days hand-tuning when JIT-Agent can generate something in two minutes.

---

## 6. An Easy-to-Miss Signal: Which Models Benefit Most from Harness

The ablation results reveal a pattern worth noting:

**Weaker (or efficiency-oriented) models gain more in absolute terms.** DeepSeek-V4-Flash averaged +8.8 points, peaking at +24.8 on a single benchmark. The reason is straightforward — weaker models have larger capability gaps, and a good harness can compensate (e.g., insufficient planning ability gets supplemented by the P module).

**Stronger models reach higher ceilings.** GLM-5.2 already scores 74.1 baseline, but with JIT it pushes to 81.8 and hits 93.9 on DeepSearchQA. For strong models, the harness shifts from "filling gaps" to "reducing waste" — more precise tool filtering and more efficient memory compression keep the model from taking detours.

**The practical takeaway:** Model selection shouldn't be based on benchmark scores alone. A cheap model plus a good harness may be worth more than an expensive model with its default harness. Our [earlier analysis of Kimi K3](/local-first-model-needs-local-first-harness/) reached the same conclusion; now there's larger-scale data to back it up.

---

## 7. What This Paper Doesn't Solve

Reality check — a few limitations.

**1. The AgentIF gap.** JIT-Agent still loses to Claude Code on open-ended work tasks. These require heavy judgment calls — when to ask the user, when to backtrack, when to abandon a path. That kind of decision logic is hard to learn from historical cases in an archive. Claude Code's harness has extensive engineering invested in these edge cases.

**2. Module boundary problems.** Splitting a harness into four pieces is a useful abstraction, but real-world boundaries aren't that clean. Memory compression strategies affect planning quality; tool filtering decisions need planning context. The paper uses the fixed dependency order M→P→F→A to handle this, but in practice these modules often need to feed back into each other.

**3. No per-module ablation.** The paper shows that different tasks produce different module implementations, but doesn't ablate M/P/F/A individually. We don't know whether Memory design or Planning design matters more for any given task.

**4. Cold start for evolution.** The archive starts from 13 seeds. How well it handles cold start for entirely new domains isn't deeply discussed. If your task type diverges significantly from the seed bank, JIT-Agent's first-generation quality may suffer.

---

## 8. What This Means for Practice

**Short term (things you can do now):** Re-examine your harness through the M/P/F/A lens. Does your CLAUDE.md clearly separate memory management strategy (M), task decomposition logic (P), and tool selection rules (F)? Separating them isn't just tidying up — it helps you see which module is your current bottleneck.

**Medium term (once JIT-Agent's open source is running):** In local-first scenarios, JIT-Agent may be most valuable. Run a Qwen3.6-27B for JIT-Agent harness generation, then apply the harness to another local model. The combined cost of two 27B models may be cheaper than a single GPT-5.6 API call.

**Long term (where Harness Engineering is heading):** This paper confirms Harness Intelligence as an **independent scaling dimension.** Model scaling (bigger models), Agent scaling (longer horizons), Knowledge scaling (better knowledge bases), and now Harness scaling (better scaffold generation). Four orthogonal axes — meaning you don't have to bet your entire budget on one line.

Looking back at six months of this blog, from the [Harness Engineering architecture overview](/harness-engineering-architecture-overview/) to [Pi's 99.93% cache hit rate](/pi-cache-hit-99-93-context-compression-roadmap/), we've been saying the same thing: **don't just watch the model — watch the harness.** JIT-Agent's contribution is turning that slogan into a quantifiable, trainable, automatable direction.

---

## Fact Sheet

| Claim | Source | Verification |
|-------|--------|-------------|
| JIT-Agent based on Qwen3.6-27B | arXiv 2608.25593 Section 4 | Paper text |
| Four-module protocol h = (M, P, A, F) | arXiv 2608.25593 Section 3 | Paper text |
| 13 seed harnesses | arXiv 2608.25593 Section 4.1 | Paper text |
| DSV4-Flash + JIT: DeepSearchQA 85.1 vs GPT-5.6 76.0 | arXiv 2608.25593 Table 2 | Paper text |
| GLM-5.2 + JIT peak gain +20.2 | arXiv 2608.25593 Abstract | Paper text |
| Nine-benchmark avg: JIT + DSV4-Flash 75.5 / JIT + GLM-5.2 81.8 | arXiv 2608.25593 Table 2 | Paper text |
| Claude Code comparison: DSQA JIT 85.1 vs CC 79.6 | arXiv 2608.25593 Table 4 | Paper text |
| AgentIF: Claude Code 66.9 vs JIT 63.8 | arXiv 2608.25593 Table 4 | Paper text |
| Tokens: JIT 400K vs CC 625K vs OpenCode 1,832K | arXiv 2608.25593 Table 4 | Paper text |
| Cost: JIT $0.066 vs CC $0.088 vs OpenCode $0.258 | arXiv 2608.25593 Table 4 | Paper text |
| DeepPlanning-Shopping largest single gain +24.8 | arXiv 2608.25593 Table 2 | Paper text |
| Three training stages: Customization / Repair / Evolution | arXiv 2608.25593 Section 4 | Paper text |

---

## FAQ

**Q: How much compute does JIT-Agent itself require?**

Based on Qwen3.6-27B, it needs one A100 or equivalent GPU for inference. Harness generation time isn't explicitly reported in the paper, but from token counts it should take seconds to a few minutes. Compared to humans hand-tuning harnesses over hours or days, that's an order-of-magnitude speedup.

**Q: Can I use JIT-Agent to generate a better harness for Claude Code?**

In theory yes, but in practice it doesn't make much sense. Claude Code's harness is deeply integrated into its runtime — you can't easily swap out its memory management or tool filtering logic. JIT-Agent is better suited for scenarios where you're calling an LLM via API and building your own agent scaffold.

**Q: How is this different from the Agents-A1 "35B beats 1T" paper?**

Agents-A1 shifted the scaling axis from parameters to training horizon — using longer trajectory data to train a smaller model. JIT-Agent shifts the scaling axis to harness — not changing the model itself, but changing the scaffold around it. Both papers attack the same problem (agent capability doesn't come from model size alone) but from different angles.
