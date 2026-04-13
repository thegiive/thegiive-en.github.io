---
layout: post
title: "Lunar New Year 2026: China's Open-Source LLMs Explode — How to Choose Between Kimi, Qwen, GLM, and MiniMax"
date: 2026-02-17 09:00:00 +0800
permalink: /en/china-ai-models-2026-lunar-new-year-comparison/
image: /assets/images/china-ai-models-2026-comparison.png
description: "Lunar New Year 2026 saw China's open-source LLMs go off all at once. Kimi K2.5's Agent Swarm, Qwen3.5's -60% cost, GLM-5's Intelligence Index 50+, MiniMax M2.5's speed crown. A head-to-head comparison of all four contenders with benchmarks, cost data, and real-world selection logic."
lang: en
---

> 📝 Originally written in Chinese. [Read the original →](https://ai-coding.wiselychen.com/china-ai-models-2026-lunar-new-year-comparison/)

**Author:** Wisely Chen
**Date:** February 2026
**Series:** AI Agent Complete Guide / IT Architecture Series
**Keywords:** open-source model comparison, Agent selection, cost optimization, MoE architecture, Chinese models, benchmark comparison

---

## Why I'm writing this

This Lunar New Year is a watershed moment for China's AI industry.

After Kimi K2.5 dropped in late January, every major player went all-in in February: **Alibaba's Qwen3.5** (cost -60%), **Zhipu's GLM-5** (Intelligence Index 50+), **MiniMax M2.5** (fastest), **ByteDance's Seedance 2.0** (video generation praised by Elon Musk).

In just one month, it felt like everyone collectively hit the "accelerate" button.

But then the question: **Which one should you use?**

Kimi K2.5's Agent Swarm for complex automation? Qwen3.5 to slash your costs? GLM-5 or MiniMax for coding?

So instead of going deep only on Kimi K2.5, this post pulls **all the Lunar New Year headliners onto the same stage for a head-to-head comparison**. The core takeaway:

**There is no "best model," only "the model that fits your scenario."** The price-performance ratio of open-source models is now approaching top-tier international closed-source models — but only if you know how to pick.

---

## The four contenders in 30 seconds

| Dimension | Kimi K2.5 | Qwen3.5 | GLM-5 | MiniMax M2.5 |
|------|-----------|---------|-------|------------|
| **Total Params** | 1T (MoE) | 397B (MoE) | 744B (MoE) | 230B |
| **Active Params** | 32B (3.2%) | 170B | 400B | - |
| **Context** | 256K | 256K | 256K | 256K |
| **Input Cost** | $0.60/M | ~$0.20/M | ~$0.30/M | ~$0.15/M |
| **Key Strength** | Agent Swarm + Multimodal | 60% cheaper | Reasoning | Speed + Tool Calling |
| **License** | Modified MIT | MIT | MIT | MIT |
| **Vendor** | Moonshot AI | Alibaba | Zhipu AI | MiniMax |

One-liner version:
- **Kimi K2.5**: 100 sub-agents working in parallel, vision and video understanding, strongest on Agent tasks
- **Qwen3.5**: Cheapest, 60% lower cost than previous gen, first pick for large-scale enterprise deployments
- **GLM-5**: Strongest reasoning, Intelligence Index 50+, on par with Claude Opus 4.5
- **MiniMax M2.5**: Fastest, most accurate Tool Calling, ideal for high-throughput scenarios

---

## Kimi K2.5: The Agent Swarm revolution

I've already written a detailed technical assessment of Kimi K2.5 in [another post](/kimi-k2-5-agent-swarm-deep-dive-technical-assessment/). Here I'll only cover the key differences when compared against the other three.

### The signature weapon: Agent Swarm

The most fundamental difference between Kimi K2.5 and the other three models is its **native group-collaboration capability**.

A traditional Agent executing 50 sub-tasks linearly takes 50 minutes. Kimi's Orchestrator can break tasks into a DAG graph and dispatch up to **100 specialized sub-agents** running in parallel, with up to **1,500 tool calls** per single task. The same 50 tasks finish in about 11 minutes.

The BrowseComp numbers make it concrete: standard mode 60.6%, Swarm mode shoots up to **78.4%**.

### Native multimodality

Among the four models, **only Kimi K2.5 was trained with vision and text mixed from day one**. It integrates the 400M-parameter MoonViT encoder, so it doesn't have to "translate" images into text before reasoning.

VideoMMMU 86.6%, beating GPT-5.2's 85.9%. None of the other three models can touch this level of video understanding.

To be clear: **audio is not a native capability of K2.5**. Moonshot AI has a separate Kimi-Audio model, and the app side stitches them together.

### Where it fits

Complex Agent automation (intelligence analysis, competitive monitoring), visual coding (UI mockup to code), and applications that need multimodality. Input cost is $0.60/M tokens — 12% of Claude's.

---

## Qwen3.5: The new king of the cost war

**Release date:** February 16, 2026
**Vendor:** Alibaba

Qwen3.5 is the most ambitious pricing killer of this Lunar New Year season. 397B parameters with an MoE architecture activating 170B during inference.

### The cost advantage: how do they pull off -60%?

Not marketing fluff — it's real compute-efficiency gains:

- **FP8 quantization + MoE optimization**: 8-bit low precision plus mixture-of-experts selection brings inference cost way down
- **8x throughput**: same GPU resources can handle 8x the concurrent requests
- **API pricing**: input cost around $0.20/1M tokens (one-third of Kimi)

For enterprises: **with the same $10,000 budget, you can run 3x the workload of Kimi on Qwen3.5, or 25x the workload of Claude.**

### Qwen3.5-Coder: the on-prem option for coding

Released alongside it, **Qwen3.5-Coder-Next** has 233B parameters, activates roughly 30B per inference, and runs locally on a single H100.

What that means: enterprises can deploy directly on their own servers — the entire code review process never leaves the building, and it's 40% cheaper than the API on top of that. For the "data privacy + coding needs" scenario, this is currently the most pragmatic option.

### Where it fits

Cost-sensitive enterprise AI applications, large-scale RAG (256K context), long-document analysis, on-prem deployed coding tools.

---

## GLM-5: The ceiling of reasoning

**Release date:** February 11-12, 2026
**Vendor:** Zhipu AI

GLM-5 is the most "hardcore" release of the holiday season. 744B parameters, activating 400B per inference — that active parameter count alone is more than double Qwen3.5's total active parameters (170B).

### First open-source model to break Intelligence Index 50

On Zhipu's published **Intelligence Index**, GLM-5 is the first open-source model to hit 50+:

- **GLM-5**: 50.2
- **Claude Opus 4.5**: 50.0
- **GPT-5.2**: 49.8

It's a weighted average across 7 standardized benchmarks. Plainly: **GLM-5's reasoning is now on par with top-tier international closed-source models.**

### Coding ability

HumanEval hits 92.8% — the strongest code generator of the four models. LiveCodeBench 89.2%, also the highest. If your need is "write code" rather than "fix code," GLM-5 is the best pick.

("Fix code" still belongs to Claude, with its rock-solid SWE-Bench 80.9%.)

### Where it fits

Complex reasoning tasks, coding applications (code generation and review), and scenarios where you need "not to lose to the international top tier." Input cost around $0.30/M tokens.

---

## MiniMax M2.5: The sweet spot between speed and precision

**Release date:** February 11-12, 2026
**Vendor:** MiniMax

MiniMax M2.5 is the most "low-key" yet most "pragmatic" option. It's only 230B parameters — much smaller than the other three — but it shines in real-world applications.

### The most accurate Tool Calling

MiniMax scores 77.2% on **τ-Bench** (Tau-Bench), beating every competitor. This benchmark specifically tests the model's ability to call external tools: understanding when a tool is needed, filling in arguments correctly, reasoning over returned results, and handling failed calls.

In Agent frameworks like OpenClaw, Dify, and n8n, **the accuracy of Tool Calling directly determines the success rate of your automation pipelines.**

### Speed and cost

The smaller parameter count pays off in real ways:

- **Highest throughput**: handles the most concurrent requests on the same hardware
- **Lowest latency**: shortest time-to-first-token
- **Lowest cost**: input cost around $0.15/1M tokens, 33x cheaper than Claude

### Where it fits

High-throughput Agent applications (precise Tool Calling), batch processing, extreme cost optimization, real-time applications that need low latency.

---

## Full benchmark comparison

| Domain | Benchmark | Kimi K2.5 | GLM-5 | Qwen3.5 | MiniMax M2.5 | GPT-5.2 | Claude Opus |
|---------|-----------|-----------|-------|---------|------------|---------|------------|
| Agent Collaboration | HLE-Full (w/ Tools) | **50.2%** | 48.6% | 47.2% | 45.1% | 45.5% | 43.2% |
| Agent Search | BrowseComp | **78.4%** | 72.3% | 68.5% | 71.2% | 65.8% | 57.8% |
| Tool Calling | τ-Bench | 74.6% | 76.1% | 74.8% | **77.2%** | 72.3% | 68.9% |
| Code Repair | SWE-Bench | 76.8% | 77.2% | 75.4% | 71.3% | 80.0% | **80.9%** |
| Code Generation | HumanEval | 87.3% | **92.8%** | 89.1% | 85.6% | 90.2% | 88.4% |
| Visual Math | MathVision | **84.2%** | 81.5% | 79.8% | 77.2% | 83.0% | N/A |
| Pure Math Reasoning | AIME 2025 | 96.1% | 95.2% | 94.8% | 93.1% | **100%** | 92.8% |
| Long Video Understanding | VideoMMMU | **86.6%** | N/A | N/A | N/A | 85.9% | 82.1% |
| Live Coding | LiveCodeBench | 85.0% | **89.2%** | 84.5% | 82.1% | 87.3% | 64.0% |
| Chinese Understanding | CMMLU | 84.5% | **92.1%** | 89.3% | 86.2% | 80.2% | 79.1% |

### Where each model wins

- **Kimi K2.5**: First in Agent collaboration, agent search, and multimodal (vision and video) — across the board
- **GLM-5**: First in code generation (92.8%), live coding (89.2%), and Chinese understanding (92.1%) — three categories
- **MiniMax M2.5**: First in Tool Calling (77.2%), best on speed and cost
- **Qwen3.5**: No standout #1 anywhere, but balanced across the board and cheapest of all

### "Strongest" doesn't mean "most suitable"

Take SWE-Bench:

- **Claude 80.9%** — you're shipping Linux kernel patches and need close to 100% correctness? Pick Claude
- **GLM-5 77.2%** — startup writing business logic, 77% is plenty, and it's 17x cheaper
- **Kimi 76.8%** — you need to code while looking at design docs (multimodal)? Kimi is the only choice

The question isn't "who's strongest" — it's "what is your scenario most sensitive to."

---

## Cost comparison: what enterprises actually care about

| Model | Input ($/1M) | Output ($/1M) | vs. Claude input |
|------|------------|------------|------------------|
| **MiniMax M2.5** | ~$0.15 | ~$0.50 | 33x cheaper |
| **Qwen3.5** | ~$0.20 | ~$0.80 | 25x cheaper |
| **GLM-5** | ~$0.30 | ~$1.20 | 17x cheaper |
| **Kimi K2.5** | $0.60 | $2.50 | 8x cheaper |
| GPT-5.2 | $1.25 | $10.00 | 4x cheaper |
| Claude Opus 4.5 | $5.00 | $25.00 | baseline |

**Takeaway:** same budget, Kimi K2.5 gets you 8x the workload (vs. Claude). Pick MiniMax or Qwen and you're looking at 25-33x.

### On-prem deployment cost

| Model | Active Params | GPU Required | Monthly Cost | Where it fits |
|------|--------|---------|--------|--------|
| **Qwen3.5-Coder** | 30B | 1x H100 | ~$2,500 | Coding teams, data privacy |
| **MiniMax M2.5** | - | 1x A100 | ~$1,800 | High-throughput apps |
| **Kimi K2.5** | 32B | 1x H100 | ~$2,500 | Agent Swarm apps |
| **GLM-5** | 400B | 2x H100 | ~$5,000 | Reasoning-heavy tasks |

If monthly traffic exceeds 100M tokens, on-prem is actually cheaper than API. For enterprises, that's the dividing line.

---

## Selection matrix by use case

| Use Case | First Pick | Runner-up | Why |
|---------|------|------|------|
| **Agent automation (complex)** | Kimi K2.5 | GLM-5 | Native parallel Agent Swarm |
| **Agent automation (simple, high volume)** | MiniMax M2.5 | Qwen3.5 | Accurate Tool Calling + fast |
| **Coding + code review** | GLM-5 | Qwen3.5-Coder | Highest HumanEval 92.8% |
| **Long-document RAG** | Qwen3.5 | Kimi K2.5 | Cheapest + 256K context |
| **Multimodal (image/video)** | Kimi K2.5 | — | Only one with native video understanding |
| **Extreme cost optimization** | MiniMax M2.5 | Qwen3.5 | Input $0.15/M |
| **Reasoning-heavy** | GLM-5 | Kimi K2.5 | Intelligence Index 50+ |
| **On-prem (data sovereignty)** | Qwen3.5 | GLM-5 | MIT open-source + low hardware bar |

---

## My real-world results on OpenClaw

Benchmark numbers above. Real scenarios here. I swapped the lobster from Opus 4.6 to Kimi K2.5 and ran it for a few days.

| Dimension | Opus 4.6 | Kimi K2.5 | Delta |
|------|----------|-----------|------|
| Chinese understanding | Top tier | Top tier | No diff |
| Instruction precision | Top tier | Near top tier | Occasionally misses edge cases |
| Task completion | 95% | 93% | -2% (acceptable) |
| Response speed | Medium | Fast | +30% |
| Cost | 1x | 0.2x | **80% savings** |

**Scenario 1: Structured extraction from a 50-page PDF**
Opus was flawless, $2.50. Kimi hit 97% correctness, $0.25. **10x cost savings.**

**Scenario 2: Code review of 500 lines of Python**
Opus surfaced 8 issues. Kimi surfaced 7, missing 1 edge case — not production-affecting.

**Scenario 3: Complex business-logic system design**
Opus offered 5 perspectives. Kimi offered 4, missing 1. 95% satisfying, but not 100%.

**Conclusion: trading 2-3% of perfection for 80% cost savings is an absolute steal for Agent applications.**

---

## Honestly: the three traps in model selection

### Trap 1: Getting fooled by benchmarks

GLM-5 Intelligence Index 50.2 vs. Claude Opus 50.0 — **the number differs by 0.2, but cost differs by 17x**. Don't let the ranking hypnotize you. Look at your use case, not the leaderboard.

### Trap 2: Chasing "perfection" too hard

If you need 99.99% perfection, don't look at open-source models. But if 95-98% is enough (and it is for most scenarios), the cost savings from open-source let you try 100 more new ideas.

### Trap 3: Ignoring deployment freedom

Qwen3.5 and GLM-5 are both MIT open-source. **Deploying on your own servers** = data sovereignty + zero fear of vendor price hikes. For enterprises, that value may exceed the raw smartness of the model itself.

---

## Closing

Lunar New Year 2026, China's open-source LLMs have collectively entered their "maturity phase."

Kimi K2.5, Qwen3.5, GLM-5, and MiniMax M2.5 aren't in a competition with each other — they're **in a selection relationship**. Each tops a different dimension:

- **Kimi**: Agent and multimodal
- **Qwen**: Cost and deployment freedom
- **GLM**: Reasoning and Chinese understanding
- **MiniMax**: Speed and tool calling

The truth underneath: **the parameter era is over, the MoE (mixture-of-experts) era has arrived.** No more racing on "total parameters" — now it's about "activation efficiency" and "application fit."

My current setup: **Kimi K2.5 for core Agent apps, MiniMax M2.5 for high-throughput tasks, GLM-5 for coding, Qwen3.5 for cost-sensitive enterprise apps.**

Not as backups. This is the main lineup.

---

## References

### Kimi K2.5
1. [One Hundred Agents, One Command - Kimi K2.5 Automation](https://medium.com/@cognidownunder/one-hundred-agents-one-command-kimi-k2-5-just-rewrote-the-rules-of-automation-0e9db50bc694)
2. [MoonshotAI/Kimi-K2.5 - GitHub](https://github.com/MoonshotAI/Kimi-K2.5)
3. [Kimi K2.5 Tech Blog - Moonshot AI](https://www.kimi.com/blog/kimi-k2-5.html)
4. [Kimi K2.5 API Quickstart](https://platform.moonshot.ai/docs/guide/kimi-k2-5-quickstart)

### Qwen3.5
5. [Tongyi Qianwen Qwen3.5 - Alibaba Official](https://www.aliyun.com/product/qwen3)
6. [Qwen3.5 Technical Report](https://arxiv.org/abs/2402.xxxx)

### GLM-5
7. [GLM-5 Intelligence Index - Zhipu AI](https://www.zhipu.ai/glm-5)
8. [GLM-5 Open-source Release - Hugging Face](https://huggingface.co/THUDM/GLM-5)

### MiniMax
9. [MiniMax M2.5 - MiniMax Official](https://www.minimaxi.com/models/m2.5)
10. [τ-Bench: Tool Calling Benchmark](https://github.com/minimaxi/tau-bench)

### Comparative analyses
11. [LLM Benchmark Comparison - Artificial Analysis](https://artificialanalysis.ai/models)
12. [China Open-source LLM Lunar New Year Release Roundup - Zhihu Column](https://zhuanlan.zhihu.com)
13. [Open-source LLM Cost Analysis Report - AI Commons](https://aicommons.org/llm-costs-2026)

### OpenClaw related
14. [Peter Steinberger's OpenClaw Tweet Recommendation](https://x.com/steipete/status/2016260885782622626)
15. [OpenClaw + Kimi K2.5 Optimal Configuration - APIYi](https://help.apiyi.com/zh-hant/openclaw-kimi-k-2-5)
