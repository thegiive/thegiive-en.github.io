---
layout: post
title: "How a Top VC Sees AI in 2026: AGI Has Arrived, but the Adoption Gap Keeps Widening"
date: 2026-09-28 09:00:00 +0800
permalink: /en/sequoia-pat-grady-state-of-ai-bubble-110m-3-4b/
image: /assets/images/sequoia-pat-grady-state-of-ai-cover.png
image_alt: "Cover for the breakdown of Pat Grady's 15-minute state-of-AI video for the Boston College investment committee"
author: "Wisely Chen"
category: AI Industry Analysis
tags:
  - Sequoia
  - Pat Grady
  - AGI
  - Bubble
  - Valuation
  - Jev
  - FDE
  - Own Your Intelligence
  - Diffusion Gap
  - Application Layer
description: "On September 24, Sequoia's Pat Grady recorded a 15-minute state-of-AI briefing for the investment committee of his alma mater, Boston College, and later posted it on X. Three inflection points, AGI has arrived, labs sending deployment squads to compete for tokens, application-layer companies reinventing themselves every four months. The hardest slide shows seven real deals: Sequoia got in at an average of $110M, and within a month the next round priced at an average of $3.4B, roughly 31x. He called it a bubble. This post separates which parts of those 15 minutes are evidence and which are positioning, and explains why that bubble slide doubles as a fundraising deck."
lang: en
---

> **Translation Note**
>
> This article is translated from the Chinese original. The author writes primarily in Traditional Chinese, and the original version is the canonical source — including the latest updates, comments, and follow-up discussions.
>
> **Read the Chinese original:** [頂級 VC 怎麼看 2026 年現在的 AI：AGI 已至，但是落地的落差越來越大](https://ai-coding.wiselychen.com/sequoia-pat-grady-state-of-ai-bubble-110m-3-4b/)
>
> If you spot any translation errors or have feedback, please refer to the Chinese version as the source of truth.

Seven deals. Sequoia got in at an average post-money of $110M. Within a month, the capital partners came in at an average price of $3.4B.

Roughly 31x. In one month.

That's the second-to-last slide of a 15-minute Loom Pat Grady recorded on September 24. He's one of Sequoia's leaders, and the video was originally made for the investment committee of his alma mater, Boston College — which is a Sequoia LP. He showed it to his partners, and later posted it on X.

After walking through those numbers, he drew his own conclusion (paraphrased; see the note on sources below):

> This is something we've never seen before, and it's a clear sign of a bubble in the market.

A VC telling his own LPs "this is a bubble" is worth taking apart one slide at a time.

*A note on quotes: my source was a Whisper machine transcript translated into Chinese. There is no verified English transcript, so the quotes in this post are back-translations, not Pat's exact words — except the line from his X post, which is original.*

---

## The 30-Second Overview

| Item | Details |
|------|---------|
| Speaker | Pat Grady, Sequoia |
| Audience | Boston College investment committee (a Sequoia LP) |
| Recorded | 2026-09-24 (per the speaker) |
| Length | 15 minutes |
| Structure | History of tech waves → three inflection points → the good / the bad / the ugly → what the labs believe → diffusion gap → application layer → operations → valuations |
| Hardest number | Seven deals: Sequoia in at an average of $110M, next round within a month at an average of $3.4B |
| Unverifiable numbers | Revenue and growth rates of named and unnamed companies — all verbal claims |

---

## How He Defines "Now"

Pat's timeline is clean. Three points:

- **November 2022**: the power of pretraining
- **Late 2024, OpenAI's o1**: the power of reasoning — System 1 and System 2 thinking
- **Last fall, Claude Code and Opus 4.5**: the power of long-horizon agents

The first two are continuous progress; the third, he argues, is a jump. In January, Sequoia published "2026: This is AGI," co-written by Pat Grady and Sonya Huang, using a functional definition: if it can figure things out on its own, plan, use tools, and loop until the goal is reached, that's AGI. He says people kind of laughed at them back then, and by September it had become consensus.

His analogy is "a faster horse" versus "a car." The AI of the past few years was a faster horse — still software at heart, just more convenient. Now it's a car: a fundamentally different way of getting you somewhere.

His evidence is two consumer products: Instinct (founded by Noah Shinn, valued at $2.5B in August, reportedly in talks at a $10B valuation) and Meta's Muse, launched in early September. His framing is that these two went "…from an app to an assistant" — not handling tasks, but finishing the work.

He compares it to Zoom. Video conferencing had existed for ages, but nobody trusted it; Zoom was the first product to cross the trust threshold and become the default way to meet. **The theory of AI doing things for you has been around for a while. Crossing the trust threshold is what happened this year.**

I agree with half of this framework. The trust-threshold point is right. "The car has arrived," though, extrapolates from two consumer products to the whole economy and skips the enormous stretch of enterprise adoption in between. He admits as much later in the video.

---

## The Labs Have Two Businesses: Selling Tokens, and Sending People to Burn Them

In the section on what the labs believe internally, Pat applies Occam's razor: these people aren't good at public communication, so when what they say sounds calculated or contradictory, they're probably just stating what they really think. AGI is here, next up is ASI, models have spent most of the past year building themselves (RSI, recursive self-improvement), and alignment is a real problem — which is why Dario Amodei published "We Must Pace the Frontier" on September 12.

That section is all secondhand. What I care more about are two sentences at the business layer.

First: token competition is fierce, the number of companies consuming APIs at scale is limited, and the labs keep cutting prices to win them.

Second: the labs have set up "deployment squads" — whole teams of consultants embedded in Fortune 500 companies, building custom applications that consume tokens.

Read the second one again.

This blog has written several pieces on [FDE (Forward Deployed Engineers)](https://ai-coding.wiselychen.com/fde-ai-agent-luo-di-xin-mo-shi-ju-ran-shi-chuan-tong-de-zhu-chang-gong-cheng-shi-part-1/). The core argument: AI adoption in complex domains depends on people embedded on-site, not on SaaS automation. Pat's remark confirms it — even the model labs have to send people into enterprises, because the product doesn't diffuse on its own.

But he also said the quiet part out loud: **deployment squads are another weapon in the token war.** Who an embedded engineer answers to determines what they'll build for you. An FDE whose KPI is "did the customer's problem get solved" and an FDE whose KPI is "how many tokens does the customer burn per month" will propose different architectures for the same requirement. The first will ask "does this decision actually need a large model?" The second rarely does.

---

## The Diffusion Gap: The Real Thesis of the Whole Video

> There's a huge gap between model capability and actual adoption. We call it the "diffusion gap."

This is the load-bearing wall of the whole video, and it's his investment logic for the LPs: lab capability has arrived, enterprises haven't caught up, and the distance in between is the opportunity for application-layer startups.

His list of opportunities:

- **High-value knowledge work**: software engineering, cybersecurity, healthcare, financial services, accounting — each major domain will produce one giant company
- **New systems of record**: the cloud era produced ServiceNow, Workday, and Salesforce; this transition will produce a new generation
- **Frontier science**: he names Chai (Chai Discovery). Chai-2 hit a 16% hit rate in fully de novo antibody design, which the company says is more than 100x better than previous computational methods
- **New kinds of labs**: we don't need another general-purpose lab; we need vertical or new-architecture labs. His example is Jev

This list pulls against the "the car has arrived" claim from earlier. If the car had truly arrived, the diffusion gap should be small. A large diffusion gap means most enterprises are still on horseback. In the first half he says AGI is here; in the second half he says most people aren't using it. Both are true — but the second one is his reason to raise money.

---

## Those Growth Numbers — I Can Only Verify Half

While presenting the growth slides, Pat added in passing that they'd be revising some of these slides later:

- **Instinct**: daily growth still above 10%
- **A high-value knowledge-work company**: revenue around 200M at the end of last year, around 700M this year
- **Another company in the same category**: 2M to 50M
- **Jev**: from zero to $100M in revenue over the past seven days

The last three have no public numbers to check against. Jev does, because [Jev's pricing is public](https://ai-coding.wiselychen.com/jev-typesafe-decision-model-judgment-as-component/): $0.042 per million input tokens, with no charge for output.

Working backward from that price, $100M in revenue over seven days requires roughly 2,380 trillion input tokens — about 340 trillion per day.

I couldn't find any public reference point that would make that volume plausible. What public reporting does show is valuation: TypeSafe is reportedly raising a new round of $1B+ at a $10B valuation. Revenue has not been disclosed.

Possible explanations: he meant annualized run-rate, or committed contract value, or Whisper misheard a word, or it was simply a slip of the tongue. I can't tell which. But there's a direct implication: **the only revenue number on the slides that can be checked against public pricing doesn't check out.** For the other, unnamed numbers, you can only choose to believe them or not.

---

## "Own Your Intelligence": Sequoia's Second Time in Two Months

In the operations section there's a trend called "own your intelligence."

In August, Sonya Huang made the [not your weights, not your product](/en/not-your-weights-not-your-product/) argument at a Sequoia event. Her reason then was domain performance: open-source baselines are high enough now that post-training on your own data may beat the API.

Pat switches the reason this time. He explicitly says it's not because of fear of the foundation model companies, and it's not GDPR:

> …it's simply that the foundation models only occupy a few points on the price–performance Pareto frontier.

This is more precise than the August version, and harder to argue with. Frontier models are the few points at the top-right of the curve — the strongest and the most expensive. Your workloads are spread along the entire curve, and most of them don't need the strongest point. Jev itself is an example of this argument: take a multiple-choice judgment task and hand it to a specialized model that doesn't generate text.

In the August piece I wrote that VCs have a positional interest in pushing this argument — the more portfolio companies build their own stack, the more room there is for startups in the infra, fine-tuning, and data-pipeline layer. The Pareto version hides that interest better, but it's technically correct: **route workloads by how much capability they actually need, instead of sending everything to the single strongest model.** That holds on its own, whether or not you buy the VC pitch.

---

## Reinventing Yourself Every Four Months

The operations section had two more lines I found the most practical for teams like ours in Taiwan.

First: application-layer companies that actually work have to reinvent themselves every four months. The reason is the one from earlier — the technical floor keeps shifting under your feet, and every time it moves, your product assumptions have to be rebuilt.

Second: most companies are adopting a lab-style approach, turning the organization from a "weakest-link game" into a "strongest-link game" — find your best people, protect them, and let them run.

He then mentions that org structures are starting to look more like networks of agents, though "it's not as dramatic as the post Jack Dorsey wrote a few months ago." That post is "From Hierarchy to Intelligence," co-written in April by Jack Dorsey and Sequoia's Roelof Botha, at the same time Block cut about 4,000 jobs. Pat is citing an article his own partner co-wrote. Worth keeping in mind.

---

## The Counterargument: He Called It a Bubble in Front of His LPs — Isn't That Honest?

This is the strongest rebuttal to this post, so let me state it fully first.

A VC, in a briefing to his own investors, voluntarily uses seven of his firm's real deals to say the market is in a bubble. That's the opposite of the usual VC line of "long-term bullish, valuations are reasonable." In his X post he wrote: "This is not a sales pitch, it's just a reflection on what we're seeing." He's willing to end the video by saying "in the end, we don't know what the future holds," and willing to point out, under "the bad," that hyperscalers are starting to lean on debt to fund capex — Epoch AI estimates hyperscaler cash capex will exceed operating cash flow around Q3 2026, so that claim has third-party data behind it. Isn't reading his bubble comment as spin a bit conspiratorial?

My answer: honesty and positional interest are both true at the same time.

Look at how the slide is structured. $110M is the price for the "company-building partner" — that is, Sequoia's price. $3.4B is the "capital partner" price one month later. Where is the bubble? At the $3.4B layer. Where is Sequoia? At the $110M layer.

For an LP, the right reading of this slide isn't "the market is dangerous." It's "the market is dangerous, and we're getting in 31x below the bubble." **It's an honest warning and a very effective fundraising deck at the same time.** The two don't conflict — which is exactly why he can afford to be so candid about it.

So I'm not revising my main argument, just sharpening it: the bubble he describes is real, but the question that slide answers is "which layer should the money sit in," not "should you get in right now."

---

## Whose Decisions Does This Change

**Enterprise CTOs (in my case, in Taiwan).**

Before: wait for the models to stabilize before adopting; sign an annual contract with one API provider; have the vendor or a reseller send people to run a PoC.

After — three adjustments this video supports:

1. **Write "re-evaluate every four months" into your contracts and your architecture.** If application-layer companies themselves have to reinvent every four months, an enterprise shouldn't sign up for an architecture that locks in a model for two years. Model calls should be swappable, and your eval set should be re-runnable whenever you switch models.
2. **When the vendor sends people, ask about their KPI first.** Pat has already said deployment squads are a tool in the token war. That doesn't mean they do bad work, but you should know they'll naturally lean toward using larger models in their architecture choices. For multiple-choice judgment, classification, and routing tasks, ask yourself: "does this need a frontier model?"
3. **Lay your workloads out on the price–performance curve.** List every AI call in your system and tag the capability level each one needs. Most will land in the range that doesn't need the strongest model — and that's where own your intelligence actually saves money.

---

## Reality Check

The evidence behind this post is thinner than it looks.

**The source is a machine transcript of a 15-minute video.** What I had was a Whisper English transcript translated into Traditional Chinese, with no English original to verify against. The quotes here are translations, not his exact words. Names like Instinct and Jev were only confirmed after checking; the transcript itself isn't reliable.

**Most of the revenue numbers can't be verified.** The seven valuation cases, 200M to 700M, 2M to 50M — all unnamed, all his own verbal claims. The only revenue number that can be checked against public pricing, Jev's, doesn't check out. He himself said the slides would be revised.

**"AGI has arrived" is his own definition.** It's the functional definition from Sequoia's January article, not an industry-consensus technical threshold. His claim that it has become consensus isn't backed by data.

**My reading of positional interest is also an inference.** I don't know who the seven companies are, whether Sequoia followed on in the next round, or how LP returns actually work out. 31x is a paper markup, not a realized return.

But the video gets one thing right: **it turns the "diffusion gap" into something you can invest in.** The distance between model capability and enterprise adoption used to be treated as a complaint — "adoption is hard." He redefines it as the size of an opportunity. For people doing FDE work or enterprise adoption, that framework is more useful than any single growth number.

---

## Key Insights

**1. When you hear "it's a bubble," first ask which layer the speaker sits in.** The gap between $110M and $3.4B is a spread for Sequoia and a risk for investors at the $3.4B layer. Same slide — whether it's a warning or an opportunity depends on where you are.

**2. When the vendor's deployment squad shows up, ask about their KPI.** Pat has publicly said they're a tool in the token war. They can help you adopt, but the question "does this decision need a large model?" is yours to ask.

**3. Put every AI call on the price–performance curve.** That's the only version of own your intelligence that holds up technically, whether or not you buy the VC pitch.

**4. Your architecture should survive a floor change every four months.** Swappable model calls, re-runnable evals. That's a survival requirement for application-layer companies, and it applies to internal enterprise systems too.

---

## FAQ

**Q: What is the core argument of Pat Grady's state-of-AI video?**

On September 24, 2026, Sequoia's Pat Grady recorded a 15-minute video for the Boston College investment committee. His core argument: the arrival of long-horizon agents means AGI is here, but there's a huge "diffusion gap" between model capability and actual enterprise adoption, and that gap is the opportunity for application-layer startups. He also used seven real deals to point to a valuation bubble: Sequoia got in at an average of $110M, and the next round within a month priced at an average of $3.4B.

**Q: What is the diffusion gap?**

The diffusion gap is Pat Grady's term for the distance between what AI models can already do and what enterprises and individuals actually use them for. On the lab side, models can prove math theorems and run long-horizon agents, but most enterprises haven't really adopted them. Sequoia treats this distance as the market size for application-layer companies and expects each major domain of high-value knowledge work (software, cybersecurity, healthcare, finance, accounting) to produce one giant company.

**Q: What does "own your intelligence" mean, and how is it different from just using an API?**

Pat Grady's argument is that frontier foundation models occupy only a few points on the price–performance Pareto frontier — the strongest, most expensive segment. Enterprise workloads are spread along the entire curve, and for most of those positions, a self-post-trained open-source model is a better fit than an off-the-shelf frontier API. This points in the same direction as Sequoia's Sonya Huang's "not your weights, not your product" talk in August, but the reason shifts from "domain performance" to "covering the cost curve."

**Q: How is a lab's "deployment squad" different from an FDE?**

Pat Grady describes labs sending whole teams of consultants into Fortune 500 companies to build custom applications that consume tokens, and says outright that this is a tool in the token war. In form, it's the same as an FDE (Forward Deployed Engineer) — people embedded on-site. The difference is the KPI: an FDE measured on solving the customer's problem will first ask "does this need a large model?", while a deployment team measured on token consumption naturally leans toward larger models in its architecture choices. Enterprises accepting vendor-embedded teams should clarify those teams' performance metrics first.

**Q: Is the claim that Jev made $100M in revenue in seven days credible?**

It can't be verified at this point. Pat Grady said Jev went from zero to $100M in revenue over the past seven days, but TypeSafe hasn't disclosed revenue. Working backward from Jev's public pricing ($0.042 per million input tokens, no charge for output), $100M in seven days would require roughly 2,380 trillion input tokens — about 340 trillion per day — and there's no public reference point that supports that volume. What public reporting confirms is on the valuation side: TypeSafe is reportedly raising $1B+ at a $10B valuation. The number could be annualized, contract value, a transcription error, or a slip of the tongue.

---

## Sources

- Pat Grady's X post (embedding the Loom video "AI for BC IC 2026-09"): https://x.com/gradypb/status/2103536289438130670
- [Sequoia Partner Pat Grady Releases 15-Minute Video on AI Prepared for Boston College Investment Committee (ABAB News)](https://www.ababnews.com/news/ebb234fe-5607-4d08-af61-2bc1c7b5cc98)
- [2026: This is AGI (Sequoia Capital)](https://sequoiacap.com/article/2026-this-is-agi)
- [Rival AI agents, Instinct and Meta's Muse, both add the ability to make calls (TechCrunch)](https://techcrunch.com/2026/09/17/rival-ai-agents-instinct-and-metas-muse-both-add-the-ability-to-make-calls/)
- [TypeSafe AI reportedly raising $1B at $10B valuation (Sovereign Magazine)](https://www.sovereignmagazine.com/article/typesafe-jev-reported-10-billion-valuation)
- [Hyperscaler Capex to Exceed Cash Flow by Q3 2026 (Epoch AI)](https://epoch.ai/data-insights/hyperscaler-capex-vs-cash-flow)
- [Dario Amodei — We Must Pace the Frontier](https://darioamodei.com/post/we-must-pace-the-frontier)
- [Chai Discovery Releases All-Atom Foundation Model for Zero-Shot Antibody Design (BiopharmaTrend)](https://www.biopharmatrend.com/news/chai-discovery-releases-foundation-model-for-zero-shot-antibody-design-with-1620-hit-rates-1309/)
- [Block — From Hierarchy to Intelligence](https://block.xyz/inside/from-hierarchy-to-intelligence)
- [Jack Dorsey says AI should replace the middle manager after Block cuts 4,000 jobs (CoinDesk)](https://www.coindesk.com/tech/2026/04/01/jack-dorsey-says-ai-should-replace-corporate-hierarchy-after-block-cuts-4-000-jobs)

## Related Posts

- [Sovereign AI Isn't a Slogan: Sequoia's Four-Tier Ladder, the Open 60 vs Closed 62 Math, and Where You Should Stop](/en/not-your-weights-not-your-product/)
- [Jev Sells a Single AI Judgment as a Component: 190 Earnings-Report Tests (Chinese)](https://ai-coding.wiselychen.com/jev-typesafe-decision-model-judgment-as-component/)
- [The FDE Model Explained: Why 95% of Enterprise AI Agent Deployments Fail (Chinese)](https://ai-coding.wiselychen.com/fde-ai-agent-luo-di-xin-mo-shi-ju-ran-shi-chuan-tong-de-zhu-chang-gong-cheng-shi-part-1/)
