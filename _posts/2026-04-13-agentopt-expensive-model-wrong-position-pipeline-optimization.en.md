---
layout: post
title: "\"Opus Is Too Smart, So It Shouldn't Be Doing the Planning\" — A Paper That Flips the Agent Ops Paradigm"
date: 2026-04-13 08:00:00 +0800
permalink: /en/agentopt-expensive-model-wrong-position-pipeline-optimization/
image: /assets/images/agentopt-paper-cover.png
description: "Columbia's AgentOpt paper ran 9 models in 81 combinations and proved it: Ministral 8B as Planner + Opus as Solver hits 74.27% accuracy, while Opus as Planner sits at 31.71%. Putting the most expensive model in the Planner seat is the worst move — because it's so strong it skips the tools and answers raw. Anthropic's own Advisor Tool is course-correcting in the same direction: cheap models run the main loop, Opus steps back as on-call advisor. The unit of agent pipeline optimization isn't single-model capability — it's how the model combo fits the specific task."
lang: en
---

> 📝 Originally written in Chinese. [Read the original →](https://ai-coding.wiselychen.com/agentopt-expensive-model-wrong-position-pipeline-optimization/)

![AgentOpt Paper: Client-Side Optimization for LLM-Based Agent](/assets/images/agentopt-paper-cover.png)

{% include youtube.html id="i6bXmJM5MHI" %}

**Author:** Wisely Chen
**Date:** April 13, 2026
**Series:** Agent Ops
**Keywords:** AgentOpt, Agent Pipeline Optimization, Planner-Solver, Opus, Ministral 8B, Advisor Tool, Anthropic, LLM Routing, Multi-Agent Architecture, Cost Optimization

---

For a long time I've been telling people: use Opus as the Planner, then let Sonnet or Haiku do the executing. The logic sounds reasonable, right? The smartest brain handles planning, the cheap hands and feet handle execution. Clean division of labor, controlled cost.

I used this myself. I even wrote several posts evangelizing it.

Then I read Columbia DAPLab's [AgentOpt paper](https://arxiv.org/abs/2604.06296).

**I got slapped in the face. Hard.**

There's a pair of numbers in the paper. Same task, same agent pipeline architecture — only "who plans, who solves" got swapped:

- **Ministral 8B (Planner) + Opus (Solver): 74.27%**
- **Opus (Planner) + any model (Solver): 31.71%**

A 42-percentage-point gap. And the winning combo's Planner is one of the cheapest models on the market — about **one-thirty-third** the price of Opus.

Nine models, 81 combinations, large-scale experiments. The result demolishes an assumption I'd held without question:

> **"Put the strongest model in the most important seat and you'll get the best result."**

Not necessarily. Sometimes the opposite is true. And it's not a small gap — it's a 2x difference.

---

## The most counterintuitive result first

The AgentOpt team tested every pairwise combination of 9 models on HotpotQA (a multi-hop QA benchmark) — one as Planner (decompose the question), one as Solver (use search tools to find the answer).

The top and bottom of the rankings:

| Combination | Accuracy |
|------|--------|
| **Ministral 8B (Planner) + Claude Opus 4.6 (Solver)** | **74.27%** |
| Claude Opus 4.6 (Planner) + any Solver | **31–32%** (the bottom 11 are all Opus-as-Planner) |

What is Ministral 8B? It's Mistral AI's 8B-parameter on-device small model, priced at $0.15/M tokens — about **1/33rd** the cost of Opus.

But as a Planner, it beats Opus by more than 2x.

---

## Why is the strongest model the worst here?

The team dug into the execution logs and found the cause:

**Opus is too strong — too strong to delegate.**

In the 9 combinations where Opus was the Planner, 7 surfaced `role2_never_called` — the Solver was never invoked at all. Opus saw the multi-hop question and just answered it directly, completely skipping the search tool.

The catch: this task is designed to require external lookup for an accurate answer. Opus confidently "answered raw" — and the result was, predictably, ugly.

Ministral 8B, on the other hand: it knew it couldn't answer this kind of complex question on its own, so it dutifully decomposed the question and handed it off to the Solver to look up. Which is exactly what a Planner is supposed to do.

**The weak model's "incompetence" turned into an architectural advantage.**

It reminds me of a management paradox you see in companies all the time: the most capable individual contributor gets promoted into management, then breaks the team because they "do everything themselves." A good manager isn't the strongest executor — it's the one who knows how to delegate.

---

## This isn't a one-off: a systemic finding across benchmarks

AgentOpt didn't just test HotpotQA. They ran full experiments on four benchmarks:

### 1. HotpotQA (multi-hop QA) — Planner + Solver architecture

Best combo: Ministral 8B + Opus = **74.27%**
Worst cluster: Opus + anything = **~31%**

**Takeaway:** The Planner role doesn't need "smart." It needs "follows the script."

### 2. MathQA (math reasoning) — Answerer + Critic architecture

Best combo: Opus (Answerer) + Haiku 4.5 (Critic) = **98.84%**

Interesting finding: once the Answerer is strong enough (Opus), **the Critic's model barely matters** — across 9 different Critics, the accuracy spread is only 2.9 percentage points.

**Takeaway:** Not every role deserves expensive money. Some roles inherently have low leverage.

### 3. GPQA Diamond (graduate-level science QA) — single model

The one benchmark where "just use the strongest model" wins. Opus solo gets 74.75%.

**Takeaway:** When the task doesn't involve tool use or role delegation, model capability ranking does predict performance.

### 4. BFCL v3 (multi-turn function calling) — single model

Three models tied at 70% (Opus, Qwen3 Next, and one other), but cost differs by **32x**.

**Takeaway:** When tied on accuracy, pick the cheap one.

---

## How wasteful is "just use the best model"?

The paper provides cost comparison data:

| Benchmark | High-cost combo | Budget combo (similar accuracy) | Cost gap |
|------|-----------|---------------------|---------|
| HotpotQA | Opus + Opus (~73%) | Qwen3 Next + gpt-oss-120b (71.3%) | **21x** |
| MathQA | Opus + Opus (~98.5%) | Ministral + Claude 3 Haiku (94.0%) | **118x** |
| BFCL | Opus (72%) | Qwen3 Next (71%) | **32x** |

**At similar accuracy, the gap between best and worst model combos can be 13–32x in cost.**

For any team running agent pipelines, this is real money. If you run thousands of pipeline executions a day, picking the wrong combo means tens of thousands of dollars wasted per month.

---

## Why you can't pick model combinations by intuition

The paper tested 8 search algorithms to find the best combination. One of them is **LM Proposal** — letting a strong model like GPT-4.1 "recommend" the best combo.

The result?

- GPQA (where intuition works): decent
- **HotpotQA (where intuition fails): only 34%**
- **BFCL: only 45%**

Even the strongest language model can't predict which combination is best. Because role interactions in a pipeline are non-linear — you can't infer "model A is great in role Y of pipeline X" from "model A is great on benchmark X."

Another interesting failure: **Hill Climbing**. On HotpotQA, it found the best combo only **52% of the time**. Why? Because the best combo (Ministral 8B + Opus) behaves nothing like its "neighbor" combos — local search starting from any adjacent combination can't reach the global optimum.

**The winner is Arm Elimination**, a confidence-interval-based round-by-round elimination strategy that maintains near-brute-force accuracy while saving 40–60% of the search budget.

---

## Anthropic itself is course-correcting

Reading this, you might be thinking — like I did — about Anthropic's own Claude Code architecture.

I analyzed in [Anthropic's Official Reveal: Why Is Claude Code So Good?](https://ai-coding.wiselychen.com/anthropic-dual-agent-architecture-claude-code/) that Claude Code uses a dual-agent architecture — Initializer Agent plans the task list, Coding Agent implements step by step. In `opusplan` mode, it explicitly uses Opus for planning and Sonnet for execution.

**But right around the time the AgentOpt paper came out, Anthropic shipped something new: the [Advisor Tool](https://www.inside.com.tw/article/41043-anthropic-claude-advisor-tool-opus-executor-agent-cost).**

This design completely flips Opus's role —

The old way: **Opus is the brain (Planner), cheap models are the hands (Executor)**
The Advisor Tool way: **Cheap models (Haiku / Sonnet) run the main loop as the executor, Opus steps back as an on-call "advisor," only invoked at critical moments**

Concretely:
- The executor model (Haiku 4.5 or Sonnet 4.6) drives the entire task loop, advancing step by step
- When the executor decides "I can't handle this step," it makes an API call to Opus 4.6 as Advisor
- Opus reads the full conversation, returns a strategic recommendation (typically 400–700 tokens)
- The executor takes the recommendation and continues — Opus doesn't take over control

According to TestingCatalog's tests, **Haiku paired with an Opus advisor significantly outperforms Haiku running alone, while total cost remains lower than just using Sonnet.**

What does this mean? Anthropic itself is acknowledging the problem AgentOpt identified:

1. **Opus shouldn't permanently sit in the Planner seat.** Letting it always be in control may, like the HotpotQA experiment showed, cause it to "grab the work."
2. **On-demand invocation beats always-on.** The Advisor Tool's `max_uses` parameter caps how many times Opus gets called — you can set "ask the advisor at most 3 times for the whole task."
3. **The executor's judgment of "when to call the advisor" is the key.** This perfectly matches AgentOpt's finding: pipeline performance isn't decided by the strength of any single role — it's decided by how the roles interact.

There's a fascinating fault-tolerance design: if the advisor call fails, the executor doesn't abort — it just continues. That's an admission that **a lot of the time, the executor doesn't actually need the advisor.**

---

AgentOpt's research and Anthropic's Advisor Tool point to the same conclusion: **the optimal model assignment depends on the specific task type — not "put the strongest one up front."**

- **Tasks requiring tool use** (like HotpotQA): a weak model as Planner may be better, because it won't "grab the work"
- **Pure reasoning tasks** (like GPQA): a strong model alone is enough
- **Tasks requiring critique** (like MathQA): the Critic role's model choice barely affects the result
- **Long, multi-step tasks** (like SWE-bench): cheap model on the main loop + strong model as on-demand advisor is likely the best balance

Future agent frameworks shouldn't hardcode "this role always uses this model." They should allocate dynamically based on the task — and ideally, let the model itself decide "when to call the advisor."

---

## What you can do now

### 1. Audit your agent pipeline costs

If you're running a multi-agent pipeline, spend 30 minutes listing:

- What model does each role use?
- How many tokens does each role burn per month?
- Are any roles where swapping in a cheap model wouldn't change the outcome?

The MathQA case told us: the Critic role's model choice barely affects final accuracy. Your pipeline probably has these "fake high-value" roles too.

### 2. Watch for the "capability overshoot" trap

Opus's failure on HotpotQA wasn't that it was bad — it was that it was too good. Good enough to skip the architectural step that mattered.

In your pipeline, check: **is any role's model "too smart" — smart enough to bypass the workflow it's supposed to follow?**

This is the same problem in AI agents and human organizations.

### 3. Use systematic methods to find the best combo

AgentOpt is [open source](https://agentoptimizer.github.io/agentopt/), supporting LangGraph, AutoGen, CrewAI, and other major frameworks. The core API is simple:

```python
selector = ArmEliminationModelSelector(
    agent=MyAgent,
    models={"planner": [...], "solver": [...]},
    eval_fn=eval_fn,
    dataset=dataset,
    model_prices=model_prices,
)
results = selector.select_best(parallel=True)
```

It works by intercepting LLM API calls at the HTTP layer, using caching to avoid redundant computation, then using Arm Elimination to progressively eliminate weak combinations. If your pipeline has 3+ roles, brute force search costs explode exponentially — that's where this kind of tool earns its keep.

### 4. Don't trust intuition — including AI's intuition

One of the paper's most brutal conclusions: **letting an LLM recommend its own model combinations is barely better than random.** GPT-4.1 on HotpotQA recommended combinations that hit only 34% accuracy — half of Ministral 8B + Opus's 74.27%.

If even a frontier model can't guess right, you and I picking by gut feel is even more unreliable.

---

## One-line summary

> **The unit of agent pipeline optimization isn't "single-model capability" — it's "how well a model combination fits a specific task."** Pick the wrong combo and you might pay 32x for half the result.

This paper made me rethink something: when we build agent systems, we spend tons of time on prompt engineering, tool design, memory management — but we may have never seriously asked, "does this role actually need this model?"

That might be the most overlooked but highest-ROI optimization lever we have.

---

## References

- **AgentOpt paper:** [AgentOpt v0.1 Technical Report: Client-Side Optimization for LLM-Based Agent](https://arxiv.org/abs/2604.06296)
- **Project page:** [AgentOpt GitHub](https://agentoptimizer.github.io/agentopt/)
- **DAPLab Blog:** [Why Your Agent Needs a Model Combo Optimizer, Not Just a Model](https://daplab.cs.columbia.edu/general/2026/03/22/why-your-agent-needs-a-model-combo-optimizer.html)
- **Anthropic Advisor Tool:** [Anthropic Turns Opus Into an "Advisor": A New Architecture for AI Agents](https://www.inside.com.tw/article/41043-anthropic-claude-advisor-tool-opus-executor-agent-cost)
- **Further reading:** [Anthropic's Official Reveal: Why Is Claude Code So Good?](https://ai-coding.wiselychen.com/anthropic-dual-agent-architecture-claude-code/)

---

## FAQ

**Q: Is this research saying small models are always better Planners than large ones?**

No. On GPQA (pure science QA), Opus solo is the best choice. AgentOpt's conclusion is "there is no universal best combination" — you have to test against your specific task. Ministral 8B beating Opus as Planner on HotpotQA happened because that task requires the Planner to decompose the problem instead of answering it. Different task, different result.

**Q: My pipeline only has 2 roles — is AgentOpt worth it?**

If you have 9 candidate models × 2 roles = 81 combinations, brute-force search costs $50–120 (depending on task complexity). Arm Elimination drops that to $30–70. If your pipeline burns more than $500/month in API costs, paying once to find the best combo is worth it.

**Q: How is this different from an LLM Router (like RouteLLM)?**

An LLM Router routes individual queries — "should this question go to Opus or Haiku?" AgentOpt does combinatorial optimization on the entire pipeline — "what's the best combination of Planner, Solver, and Critic models?" Different layers of the same problem. A Router saves money on individual calls; AgentOpt saves money on the whole pipeline.
