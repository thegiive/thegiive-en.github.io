---
layout: post
title: "Fable 5 Is Only 11% of Anthropic's Sales: When the Strongest Model Isn't the Best Seller, It's Time to Calculate Intelligence per Dollar"
date: 2026-08-24 12:00:00 +0800
permalink: /en/fable-5-enterprise-adoption-ceiling-intelligence-per-dollar/
image: /assets/images/fable-5-adoption-ceiling-cover.png
image_alt: "Cloud API output pricing comparison across Anthropic and OpenAI models"
author: "Wisely Chen"
category: AI Industry Analysis
tags:
  - Fable 5
  - Anthropic
  - Intelligence Index
  - open-weight models
  - AI pricing
description: "Ramp's payment data across 70,000 companies shows Fable 5, more than two months after launch, captures only 11.4% of Anthropic model spending and 6% of token usage. This piece breaks down the three forces pushing enterprises away from Fable 5, proposes two new metrics — intelligence per dollar and intelligence per billion parameters — and uses the Artificial Analysis Intelligence Index to compare intelligence density across cloud APIs and open-weight models. The conclusion: the frontier moat is down to two or three points on the Intelligence Index, and the next phase of competition is about delivering sufficient intelligence at the lowest cost."
lang: en
---

> **Translation Note**
>
> This article is translated from the Chinese original. The author writes primarily in Traditional Chinese, and the original version is the canonical source — including the latest updates, comments, and follow-up discussions.
>
> **Read the Chinese original:** [Fable 5 只佔 Anthropic 銷售的 11%](https://ai-coding.wiselychen.com/fable-5-enterprise-adoption-ceiling-intelligence-per-dollar/)
>
> If you spot any translation errors or have feedback, please refer to the Chinese version as the source of truth.

Anthropic's Fable 5 is the strongest model on the market today. That's not in dispute.

But according to a recent FT report citing payment data company [Ramp](https://ramp.com/data/ai-index-august-2026), which tracks about 70,000 companies, Fable 5 — more than two months after launch — accounts for only 11.4% of Anthropic model spending, and just 6% of token usage.

Even more surprising: Opus 5, released in late July at exactly half the price, has already overtaken Fable 5 in enterprise spending.

A company's strongest flagship model, commercially beaten by its own second-tier model that launched less than a month ago. That's a first in the AI industry.

---

## The Numbers First

| Model | Input / MTok | Output / MTok | Share of Anthropic Spend | Token Share |
|------|:--:|:--:|:--:|:--:|
| Fable 5 | $10 | $50 | 11.4% | 6% |
| Opus 5 | $5 | $25 | >11.4% (overtaken) | — |
| Sonnet 5 | $2 | $10 | — | — |
| Haiku 4.5 | $1 | $5 | — | — |

Fable 5 is priced at exactly twice Opus 5. A 6% token share against an 11.4% spend share means every token is indeed expensive — and enterprises won't even buy much of that expensive volume.

Compare with OpenAI: GPT-5.6 Sol captures 25% of token usage and 23% of spending among OpenAI models. Flagship for flagship, Sol's enterprise adoption is more than four times Fable 5's.

---

## Three Forces Pushing Fable 5 Away

**Price.** Fable 5 runs $10/$50 per million tokens; Opus 5 is $5/$25. In agent scenarios where a single task easily burns tens of thousands of output tokens, a 2x monthly bill isn't a technical decision for the CTO — it's a number the CFO understands at a glance.

**Data retention.** Fable 5 requires mandatory 30-day retention of all traffic, overriding existing Zero Data Retention (ZDR) agreements. Opus 5 supports ZDR. For companies under GDPR or HIPAA, this isn't a performance trade-off — it's "the compliance team hasn't finished review, so this model simply cannot be used." [Forrester](https://www.forrester.com/blogs/how-fable-5-and-mythos-5-change-ai-security-data-retention-and-vendor-risk/) calls it an escalation in vendor risk.

**No perceptible performance gap.** Opus 5 trails Fable 5 by half a percentage point on CursorBench 3.2, and actually beats Fable 5 on OSWorld 2.0 using one-third of the budget. Pay 100% more, get 0.5% more capability — the marginal intelligence per extra dollar is nearly zero.

---

## The Flywheel Stalls for the First Time

The growth logic of model companies over the past two years was simple: **stronger model → enterprises upgrade → revenue grows → train the next generation**. From GPT-3.5 to GPT-4, from Claude 2 to Claude 3.5 Sonnet, every turn of the flywheel worked, because each capability jump was visible to the naked eye.

But Fable 5's improvements have entered diminishing-returns territory — not because it isn't strong, but because the part where it's stronger is invisible to most real-world work. Accel partner Miles Clements put it plainly in the FT piece: "Most people don't need to operate at the frontier." He added that the era of enterprises favoring flagship models "was not a sustainable era."

---

## This Isn't Just Anthropic's Problem

Anthropic overall is doing fine — July ARR hit $65B, and enterprise adoption at 43.5% exceeds OpenAI's 39.7%. But this episode reveals a structural shift: **once model capability crosses a certain level, the marginal value of further improvement decays rapidly on the enterprise side.**

Future revenue may come not from the strongest model but from models that are "strong enough, cheap enough." The flagship's role shifts from revenue engine to technology showcase — proof that you have frontier capability, while the actual cash flow comes from the Sonnet and Opus tiers. Ramp's chief economist Ara Kharazian was blunt: Fable 5 "disappointed both in adoption and real-world application given price + data retention requirements."

---

## New Metrics: Intelligence per Dollar, Intelligence per Billion Parameters

If enterprises are buying "best value per dollar" instead of "strongest," the competitive axis shifts from benchmark scores to intelligence density.

The [Artificial Analysis](https://artificialanalysis.ai/evaluations/artificial-analysis-intelligence-index) Intelligence Index (II) is one of the most-cited composite evaluations in the industry, a weighted score across multiple benchmarks. Lining up Anthropic and OpenAI cloud APIs by II:

### Cloud APIs: Intelligence per Dollar

| Model | Intelligence Index | Input $/MTok | Output $/MTok | II / Output $ |
|------|:--:|--:|--:|:--:|
| Opus 5 | 63 | $5 | $25 | 2.52 |
| Fable 5 | 62 | $10 | $50 | 1.24 |
| GPT-5.6 Sol | 61 | $5 | $30 | 2.03 |
| Sonnet 5 | 55 | $2 | $10 | 5.50 |
| GPT-5.6 Terra | 55 | $2 | $12 | 4.58 |
| GPT-5.6 Luna | 52 | $0.20 | $1.20 | 43.33 |
| Haiku 4.5 | 30 | $1 | $5 | 6.00 |

Opus 5 scores one point higher than Fable 5 (63 vs 62) at half the price — Fable 5 is the most expensive model on the table, and not the smartest. GPT-5.6 Luna gets 52 points at $0.20/$1.20, while Haiku 4.5 gets only 30 points at $1/$5 — Luna is five times cheaper with a 73% higher score.

![Cloud API output pricing comparison](/assets/images/fable-5-cloud-api-output-price.png)

### Open-Weight Models: Intelligence per Billion Parameters

Cloud APIs measure intelligence per dollar, but open-weight models run on your own GPUs — there's no per-token bill. For them, cost is a function of model size: how much VRAM the weights occupy (Total B), and how much compute each token burns (Active B). So we switch metrics: II / Total B and II / Active B.

| Model | II | Total | Active | II / Total B | II / Active B |
|------|:--:|------:|------:|:--:|:--:|
| Kimi K3 | 60 | 2,800B | 104B | 0.021 | 0.58 |
| GLM-5.3 | 60 | 753B | 40B | 0.080 | 1.50 |
| Qwen3.8 2.4T | 58 | 2,400B | 95B | 0.024 | 0.61 |
| Qwen3.8 27B | 52 | 27B | 27B | 1.926 | 1.93 |
| Qwen3.6 27B | 46 | 27B | 27B | 1.704 | 1.70 |
| Qwen3.6 35B A3B | 43 | 36B | 3B | 1.194 | 14.33 |
| DeepSeek V4 Flash | 52 | 284B | 13B | 0.183 | 4.00 |
| Gemma 4 31B | 39 | 31B | 31B | 1.258 | 1.26 |
| Gemma 4 26B A4B | 31 | 26B | 4B | 1.192 | 7.75 |

### API vs Open-Weight = Opex vs Capex

These two tables aren't asking you to pick one. They're two different cost structures:

Cloud APIs are **Opex** — pay per use, zero upfront investment, ideal for volatile demand or early-stage work. Open-weight models are **Capex** — buy GPUs, deploy inference infrastructure, high upfront cost but marginal cost trending toward zero, ideal for high, stable volume.

The pragmatic play is to run both: use cloud APIs for prototyping and low-volume tasks, then migrate high-frequency workloads to self-hosted open-weight models once volume stabilizes. It's not about picking a side — it's about which stage you're in.

---

## How to Read the Two Charts

The two scatter plots below visualize the open-weight table. The X axis on both is the Intelligence Index, with a dashed dividing line at II = 40 — to the right is "general-use" territory viable for most scenarios; to the left is "niche" territory, usable only for specific well-defined skills.

**Chart 1: II vs II / Total B (memory efficiency).** The Y axis is how much intelligence you squeeze out of each billion total parameters. Higher means less VRAM for the same score. The top-right corner is the ideal spot — smart and memory-efficient. Qwen3.8 27B (Q38_27) and Qwen3.6 27B (Q36_27), two dense small models, dominate this metric because the entire model is 27B and runs on a single GPU. The giants (K3 at 2,800B, Q38_2.4T at 2,400B) score high but sit at the bottom of the chart on memory efficiency.

![Open-weight models: II vs memory efficiency](/assets/images/fable-5-open-weight-memory-efficiency.png)

**Chart 2: II vs II / Active B (compute efficiency).** The Y axis is intelligence per billion active parameters. MoE models have a structural advantage here — Qwen3.6 35B A3B gets 43 points with only 3B active, hitting an II / Active B of 14.33, far above every other point. Gemma 4 26B A4B reaches 7.75 with 4B active. Note this metric is inherently unfavorable to dense models (active = total), so the two charts must be read together.

![Open-weight models: II vs compute efficiency](/assets/images/fable-5-open-weight-compute-efficiency.png)

The numbers all point to one conclusion: intelligence is being compressed into ever-smaller spaces. Kimi K3 and GLM-5.3 both score 60, just two points behind Fable 5. DeepSeek V4 Flash and Qwen3.8 27B score 52 — not far from Sonnet 5's 55 — but one uses only 13B active parameters and the other runs on a single GPU. Fable 5 isn't just squeezed from above and the middle by its own siblings Opus 5 and Sonnet 5; it's being chased from below by open-weight models. The frontier moat, measured on the Intelligence Index, is down to two or three points.

---

## Advice for Pragmatists

**1. Default to Opus 5 or Sonnet 5; upgrade only when necessary.** This is already what the Ramp data shows most enterprises doing. Most coding agent, document processing, and customer service workloads are handled fine by Opus 5.

**2. Watch Fable 5's 30-day retention policy.** If your organization requires zero data retention, don't touch the Fable 5 API until your compliance team finishes review.

**3. Consider self-hosting open-weight models for high-frequency workloads.** If your API bill has stabilized at five figures a month, migrating high-frequency tasks to models like DeepSeek V4 Flash or Qwen3.8 27B may be more economical. Cloud API and self-hosted inference aren't either-or — it's an Opex-to-Capex ratio question.

**4. Redefine "good."** The highest benchmark score isn't the best choice. The model that buys you the most correct output per dollar on your real tasks is.

---

## A Closing Observation

After all these years of thrashing around in AI, humanity has rediscovered a very old commercial law:

**The best product is not necessarily the best-selling product.**

The Toyota Camry isn't the best car. UNIQLO isn't the best clothing. But they sell the most, because they found the sweet spot between "good enough" and "cheap enough."

Fable 5 faces the same story today. Frontier intelligence still matters — it defines the industry's technical ceiling. But the models that carry the overwhelming majority of commercial revenue will be the ones standing two floors below the ceiling at one-tenth the price.

For model companies, the next phase of competition may no longer be "whose model is strongest," but "who can deliver sufficient intelligence at the lowest cost."

That shift has already begun. Fable 5's 11.4% is the evidence.
