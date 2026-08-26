---
layout: post
title: "Sovereign AI Isn't a Slogan: Sequoia's Four-Tier Ladder, the Open 60 vs Closed 62 Math, and Where You Should Stop"
date: 2026-08-26 09:00:00 +0800
permalink: /en/not-your-weights-not-your-product/
image: /assets/images/not-your-weights-logo.png
image_alt: "Sequoia 'Owning Your Intelligence' event branding"
author: "Wisely Chen"
category: AI Industry Analysis
tags:
  - Sovereign AI
  - Sequoia
  - Sonya Huang
  - open-weight models
  - post-training
  - Kimi K3
description: "At a Sequoia event in August, Sonya Huang translated the Bitcoin world's 2017 slogan into the AI era: not your weights, not your product. The slogan rests on exactly one hard number — Kimi K3 and GLM-5.3 score 60 on the Artificial Analysis Intelligence Index, Fable 5 scores 62. Starting from that 2-point gap, this piece works out the real cost structure of building your own, and the positional interest a VC has in pushing this thesis."
lang: en
---

> **Translation Note**
>
> This article is translated from the Chinese original. The author writes primarily in Traditional Chinese, and the original version is the canonical source — including the latest updates, comments, and follow-up discussions.
>
> **Read the Chinese original:** [Sovereign AI 不是口號：Sequoia 的四級路徑、開源 60 vs 封閉 62 的算術、以及你該停在第幾級](https://ai-coding.wiselychen.com/not-your-weights-not-your-product/)
>
> If you spot any translation errors or have feedback, please refer to the Chinese version as the source of truth.

---

Eighty founders of portfolio companies sat in the audience at a Sequoia event when Sonya Huang took that 2017 Bitcoin slogan — "not your keys, not your bitcoin" — and swapped one noun:

> "Not your weights, not your product. I think that for product to be truly yours, I think it's reasonable to think that you need to be able to control and custody your own weights."

If your product is truly yours, the weights should be in your hands.

Two weeks before she said this, Jensen Huang used his first-ever X post to publicly push for the US to keep open-weight models. She described the reaction as a "near unanimous wave of support."

The slogan landed. But a slogan is not arithmetic.

[Full talk](https://www.youtube.com/watch?v=bMMv0bZzONg).

---

## The entire thesis rests on one number

"Your own model can beat the API" — before August 2026, that was a cost-saving play. You knew it would be dumber, but it was cheaper.

This year it became a performance play. And the number propping up that shift is just one:

| Model | AA Intelligence Index |
|------|:--:|
| Opus 5 | 63 |
| Fable 5 | 62 |
| Kimi K3 | 60 |
| GLM-5.3 | 60 |
| Sonnet 5 | 55 |

The open-source camp is **2 points** behind the closed frontier.

Sonya on stage: "this is new for 2026." The "new" she's talking about isn't that open models got stronger — they've always been closing the gap. What's new is: **a 2-point gap, in your domain, on your data, can be closed.**

What it takes to close it, laid out piece by piece:

- A continuously running eval suite — re-run every time the base model ships a new version
- Domain data: expert trajectories, synthetic data, RL environments, at least two of the three
- Inference infra: GPUs, vLLM or an equivalent stack, drift monitoring
- A small team: Harvey's research team is 7 people, the smallest sample in the public record

Seven people, legal domain, publishing research. This is the strongest evidence for the thesis.

But it's a sample, not a denominator.

---

## The half-sentence the slogan leaves out

Let's be precise about what "performance" means. Sonya isn't talking about tokens per second or throughput. She's talking about **task performance in a specific domain** — "in certain domains you may be able to get better performance by tuning models on your own data." Latency and speed are a separate line item in her framework, not the "performance" here.

She explicitly calls the performance rationale "relatively newer." In past years, companies chose open models for cost. This year, an additional reason appeared: they might be stronger in a specific domain. Her examples are all task-level: Harvey's coding tab autocomplete running on its own model (high-frequency, latency P0), a security company doing bespoke post-training on its own model, a biotech firm self-hosting because of the value of its private data.

This is the opposite of what most people assume when they hear "build your own." Most people think cost savings. But in Sonya's own ordering, cost ranks first because it's the most common reason — not because it's the newest.

**The genuinely new reason is performance, and performance is precisely the most expensive part of self-hosting.**

Note this is domain performance, not throughput. If a company's only motivation for building is tokens per second, that's an infra problem — buy more GPUs, done, no 7-person team needed. What requires a 7-person team is this: a model post-trained on your domain data understands your business better than the closed frontier on the API. Proving that "understands better" requires evals, and evals are the most expensive part.

The cost-saving path already existed — single-GPU models like DeepSeek V4 Flash or Qwen3.8 27B, the math is clear in the [intelligence-per-dollar](https://ai-coding.wiselychen.com/fable-5-enterprise-adoption-ceiling-intelligence-per-dollar/) table. The performance path only opened this year, and its bill isn't metered tokens — it's capex.

A token bill can be cut every month. GPUs and a training team can't.

---

## An unflattering observation

Sonya is a VC.

When a VC pushes "build your own," the portfolio companies get two direct benefits: one, there are more rounds to invest in — infra, fine-tuning, data tooling, RL environments, each one a new company; two, there's more reason to stay in the portfolio — self-hosting is a multi-year commitment, the API is a monthly subscription.

This isn't saying she's wrong. Harvey's 7-person team is real. Kimi K3's 60 is real.

But when a VC tells you "your product should belong to you," it's worth asking: **if every company self-hosts, what does Sequoia's portfolio look like?**

More infra companies, more fine-tuning tools, more data pipelines. Fewer asset-light application companies running on a monthly API subscription.

For a VC, that's a good business. For the portfolio companies, the premise is that you can actually close that 2-point gap.

---

## For individual developers, the slogan flips

For an enterprise CTO, "not your weights" means: high-frequency features, private data, latency requirements — none of it should be outsourced to an API.

For an individual developer, the implication is the reverse.

You don't have a data moat. You don't have a 7-person research team. What you do "own" is your harness — prompts, context management, evals, workflows. Swap the model, and all of it stays.

That's the core of the [three harness migrations](https://ai-coding.wiselychen.com/agent-harness-three-migrations-mechanism/) piece. For individual developers, **the harness is your weights.**

Same slogan, two directions:

| | Enterprise CTO | Individual developer |
|---|---|---|
| Action | Move high-frequency features to open base + post-training | Build the harness layer well; treat the model as a replaceable part |
| Cost | Capex (GPUs + team) | Time (writing evals, tuning the harness) |
| Moat | Domain data + own model | Workflows + own harness |
| Risk | Base model generation swap, re-run evals | Model upgrade, re-tune the harness |

### Self-hosting isn't binary — it's a four-tier ladder

The table above assumes one thing: the enterprise sovereignty path is building your own model. But Sonya's talk actually lays out a four-tier ladder, and self-hosting is just one rung:

| Tier | What to do | What you need | Cloud / on-prem |
|------|---------|---------|----------|
| 1 | Use out-of-the-box models as-is; the API is enough | A subscription | Cloud |
| 2 | Harness, model routing: prompts, context, evals, workflows | Time, no GPUs | Mostly cloud, start experimenting with hybrid |
| 3 | Online feedback loop: customer interaction data continuously flowing back | Tier 2 + data feedback infrastructure | Hybrid |
| 4 | Post-training: domain data + RL environments + inference infra | GPUs + a training team (Harvey's 7 people sit here) | Hybrid |

Sonya's words: "some companies find they can get good performance with out-of-the-box models. Others are finding strong performance gains from post-training... And then finally, setting that machine up so that live customer data is actually creating a feedback loop."

The key to this ladder: **every tier has a verifiable output (an eval score), and a company can stop at any tier — no one-jump to full self-hosting required.**

In the same week, 老鄭's AI Forward (AI 前瞻) long-form on AI sovereignty turned the same thesis into an actionable checklist: structuring business context, closing process loops, private evals, model redundancy — all of it sits at tiers 2 and 3, **not a single item requires GPUs**. His framing is blunter: the moat isn't whether you use AI, it's whether anything gets left behind after each use — data, processes, evals, knowledge bases, judgments reusable next time. If nothing is left behind, the tokens you burned were fireworks.

Put the two theses together and sovereignty is actually two independent questions:

- **Do the models belong to you?** — Tier 4, and the answer is the capex math.
- **Does the learning loop above the model belong to you?** — Tiers 2 and 3, and the answer is that much lighter but equally concrete checklist.

For companies that can't clear the capex hurdle, the practical option is: push tiers 2 and 3 until your eval scores prove you're beating raw API usage, then decide whether to enter tier 4. It's not "build vs. don't build." It's "which tier do you stop at."

---

## Reality Check

This thesis has a narrower scope of application than the slogan suggests.

**The 2-point gap is a snapshot from August 2026.** Kimi K3 and GLM-5.3's 60 is Artificial Analysis's composite score — not your score on your domain. One generation of base model swap and your eval suite needs re-running, your post-training data re-labeled. This isn't build-once-and-done.

**The 7-person-team evidence applies to the legal domain.** I called it "the strongest evidence for the thesis" earlier, but it's evidence for the legal domain — documents, case law, contracts. The data structure is relatively clean, easier for domain post-training than a manufacturer's sensor data or a bank's risk models. If your domain data lacks that structure, 7 people may not be enough — or you may not need 7 people at all. Sonya herself says: "every company is very very different."

**The VC's positional interest is real.** She credits Anthropic and OpenAI as good partners, but companies still want "their own set of independent legs." Good partners are also partners you can choose not to depend on. A VC pushing this thesis and the portfolio companies actually needing it are two separate facts.

But it did one thing right: **it turned "build your own" from a cost decision into a performance decision.** The old argument for self-hosting was cost — and once open models caught up, that argument got weaker, not stronger, because there are more cheap models. This year's new argument is "it might be stronger in your domain," and that argument only holds when the open baseline is high enough.

Kimi K3's 60 is the evidence that it is.

---

## Key takeaways

**One. Translate the slogan into arithmetic.** Can your data and pipeline close the gap between the open baseline (60) and your requirements? Is the total cost of closing it — GPUs, team, eval maintenance — worth it compared to the API bill? Answer both before you build.

**Two. The performance reason is new; the cost reason is old.** If cost savings is your only reason to build, first ask: are DeepSeek V4 Flash or Qwen3.8 27B already good enough? If yes, you don't need a 7-person team — you need the table from the [intelligence-per-dollar](https://en.wiselychen.com/en/fable-5-enterprise-adoption-ceiling-intelligence-per-dollar/) piece.

**Three. An individual developer's weights are the harness, not the model.** Prompts, context management, evals, workflows — the things that survive a model swap. Time spent on this layer beats chasing the new model every month. The enterprise equivalent is the learning loop: business context, private evals, model redundancy — the model can churn; the standards and the judgments must be permanent.

**Four. When a VC says "your product should belong to you," ask: after self-hosting, which new companies appear in their portfolio?** If it's infra, fine-tuning, data pipelines — then the thesis is both an opportunity and a cost for you.
