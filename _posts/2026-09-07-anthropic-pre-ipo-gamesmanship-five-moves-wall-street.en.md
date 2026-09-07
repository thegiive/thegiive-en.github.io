---
layout: post
title: "Anthropic vs OpenAI Compute Showdown: From Cornfields to a $2T IPO"
date: 2026-09-07 09:00:00 +0800
permalink: /en/anthropic-pre-ipo-gamesmanship-five-moves-wall-street/
image: /assets/images/anthropic-ipo-gamesmanship-cover.png
image_alt: "Bar chart comparing Anthropic and OpenAI compute power in GW"
author: "Wisely Chen"
category: AI Industry Analysis
tags:
  - Anthropic
  - OpenAI
  - compute
  - GW
  - IPO
  - Gavin Baker
  - AWS Rainier
  - Trainium2
  - SpaceX Colossus
  - Google TPU
  - SemiAnalysis
  - infrastructure
description: "Everyone is comparing Fable vs Astra benchmarks. But the real race is compute. At the end of 2025, Anthropic had only 70% of OpenAI's running power (1.4 GW vs 1.9 GW). By mid-2026, spend-implied averages show Anthropic at ~2.7 GW vs OpenAI's ~2.4 GW. The Information reports Anthropic signed at least 14.8 GW in compute contracts over the past eleven months. This article breaks down the compute race across three layers — running power, signed pipeline, and annual spend — then connects it to Anthropic's potential $2T IPO."
lang: en
---

> **Translation Note**
>
> This article is translated from the Chinese original. The author writes primarily in Traditional Chinese, and the original version is the canonical source — including the latest updates, comments, and follow-up discussions.
>
> **Read the Chinese original:** [Anthropic vs OpenAI 算力對決：從玉米田到 $2T IPO，誰的電表轉得比較快](https://ai-coding.wiselychen.com/anthropic-pre-ipo-gamesmanship-five-moves-wall-street/)
>
> If you spot any translation errors or have feedback, please refer to the Chinese version as the source of truth.

When Fable 5 first launched, power users knew how strong it was.

But after a while, many felt it had gotten weaker. That probably wasn't a model downgrade — it was inference demand outgrowing the available compute.

Everyone is comparing Fable vs Astra on benchmarks. But the real competition is compute distribution. At the end of 2025, Anthropic genuinely had only 70% of OpenAI's capacity. That gap is closing faster than anyone expected.

Neither company has ever published its compute numbers. All we can do is estimate from the outside — power consumption, spending, supply chain intel, chipmaker earnings calls. What follows is what those puzzle pieces add up to.

One caveat upfront: power consumption does not equal compute. A 1 GW datacenter full of H100s and a 1 GW datacenter of Trainium2 chips produce very different FLOPS. Cooling architecture, PUE, chip generation, training vs inference — each changes how watts translate to actual compute. The GW numbers below measure "scale on the power meter," not "FLOPS produced." It's the coarsest but most reliable proxy available from the outside.

---

## Layer 1: Running Power (GW)

| Timeframe | OpenAI | Anthropic | Anthropic as % |
|-----------|:------:|:---------:|:--------------:|
| End of 2025 | ~1.9 GW | ~1.4 GW | ~74% |
| Early 2026 | ~2 GW | ~2 GW | ~100% |
| 2026 average (spend-implied) | ~2.4 GW | ~2.7 GW | ~113% |

Sources: End-of-2025 figures from [Epoch AI report](https://x.com/Saemin4655/status/2088824774416072859) (OpenAI ~1.7M H100-equivalents at 11% of global share, Anthropic ~1M at 6%). 2026 figures back-calculated from compute spending.

Epoch's facility-level numbers are more conservative — Anthropic ~1.45 GW, OpenAI ~0.42 GW — but they acknowledge incomplete coverage, as cloud-rented capacity isn't on their list.

Bottom line: **In 2026, Anthropic's combined training-plus-inference running power is no longer significantly below OpenAI's.** Most methodologies put them in the same ballpark, within plus or minus 20-30%, shifting month to month.

---

## How Did They Catch Up So Fast?

Three pieces snapped together.

**1. AWS Project Rainier.** In October 2025, [Andy Jassy posted](https://x.com/ajassy/status/1983616724642730217): a cornfield in New Carlisle, Indiana had been turned into AWS's largest-ever AI compute cluster, running Trainium2 chips. His words:

> "It is 70% larger than any AI computing platform in AWS history, with nearly 500K Trainium2 chips, and is now fully operational with Anthropic actively using it to train and run inference for its industry-leading AI model, Claude."

500K Trainium2 chips online, targeting one million by year-end. Single-site capacity around 900 MW. The post got 1.57 million views. The most-liked replies weren't about technology — they were from local farmers complaining about losing their farmland.

**2. Google TPU.** This one isn't "future" — it's "always been there." [SemiAnalysis noted](https://x.com/SemiAnalysis_/status/2075203875062120785) that five or more generations of Claude were trained on TPUs. In 2026, 1 GW of Ironwood TPU is being deployed, with Q4 shipments accelerating. Broadcom CEO Hock Tan laid out the roadmap in the September [earnings call](https://x.com/FirstSquawk/status/2095260321397268918): add 5 GW of TPU v8i in 2027, another 10 GW increment in 2028. Anthropic will become Broadcom's largest XPU customer by 2027.

**3. SpaceX Colossus 1.** The most dramatic piece.

Elon Musk built Colossus 1 in Memphis — roughly 300 MW, 220,000 GPUs (H100/H200/GB200), [completed in 122 days](https://x.com/SemiAnalysis_/status/2087667981031506158). It was built to train Grok.

Then [in May 2026, Anthropic leased the entire cluster](https://x.com/MilkRoadAI/status/2052986277977366547).

[Reports indicate](https://x.com/negligible_cap/status/2065502888973967710) SpaceX decided to lease it out after running into technical difficulties training Grok on it. In February, Musk was publicly saying Anthropic "hates Western civilization." By May, they'd rented his entire datacenter. One observer [summed it up](https://x.com/itsak1to/status/2094885764408434947): Musk built the datacenter to beat Claude. Now Claude runs on it.

---

## Layer 2: Signed Pipeline

### OpenAI

Stargate remains the headline: $500 billion, 10 GW-class narrative. In April 2026 they said the 10 GW target had been locked down ahead of schedule. Partners include Microsoft, Oracle, CoreWeave, AWS Trainium, AMD, plus their in-house Jalapeño ASIC. Long-term commitments have been aggregated to the trillion-dollar range, mixing in years of inference minimums.

### Anthropic

Measured AI estimates the U.S. pipeline at roughly 5 GW by end of 2026, ~9.5 GW by 2027, and over 15 GW by 2028 — across 15 campuses, four chip types, five contract structures.

Known major deals:

| Partner | Scale |
|---------|-------|
| AWS (Rainier) | Up to 5 GW |
| Google / Broadcom | Multi-GW TPU (ramping from 2027) |
| Azure | ~$30B |
| Nscale | ~$45B / 6 years |
| SpaceX Colossus 1 | ~300 MW |
| Fluidstack, CoreWeave | Additional capacity |

In spring 2026, some observers believed Anthropic's locked-in GW already exceeded OpenAI's publicly announced commitments.

Then came the September 6 bombshell. [The Information reported](https://x.com/Techmeme/status/2096682533106917555) that Anthropic had signed at least 14.8 GW in compute contracts since October of last year, with a potential ten-year spend of $517 billion. This excludes the 1-2 GW already in place. One driver: Claude Code and Cowork demand grew so fast that [Anthropic itself was caught off guard, scrambling for capacity](https://x.com/choblin29/status/2096638464792130011).

14.8 GW aligns closely with Measured AI's earlier 15 GW pipeline estimate. The difference: now there's independent cross-confirmation from The Information.

Long-term, OpenAI's commitments are larger. But Anthropic isn't watching through binoculars — they're on the same track, separated by car lengths, not laps.

---

## Layer 3: Annual Compute Spending

Pipeline is promises. Power bills are facts. SemiAnalysis estimates for 2026 compute spend:

| Category | Anthropic 2026E | OpenAI 2026E |
|----------|:------:|:------:|
| Training spend | ~$25B | ~$32B |
| Inference spend | ~$24.1B | ~$22.6B |
| **Total compute spend** | **~$49.1B** | **~$54.6B** |

OpenAI itself has publicly stated its 2026 compute spend is roughly $50B, consistent with SemiAnalysis's $54.6B estimate.

Annual cash burn is nearly matched. OpenAI leans heavier on training; Anthropic leans heavier on inference.

Stack all three layers: running power in the same ballpark, pipeline scale where Anthropic doesn't trail by much, annual spending nearly even. The perception that "Anthropic's compute is far behind OpenAI" was a 2025 fact, not a 2026 one.

---

## Why Compute Matters: The Infrastructure Layer of the IPO

Compute isn't just a technical spec. Anthropic is preparing what could be the [largest tech IPO in history](https://x.com/kimmonismus/status/2087806611918073940), with investors discussing a $2T valuation. Atreides Management CIO [Gavin Baker's September 5 post](https://x.com/GavinSBaker/status/2096257640884027500) (820K views, self-labeled "pure speculation") outlined several strategic moves: shifting ARR from gross to net, removing Meta's $5B+ from the top line, timing model releases to the listing window. But the most significant piece for the compute narrative came from The Information the same day: IPO investors are demanding Anthropic disclose [revenue per token and revenue per GW](https://x.com/theinformation/status/2096697449133809907). Compute is becoming a variable in the valuation formula — Anthropic's IPO story isn't "our model is the smartest," it's "we have infrastructure at OpenAI's scale, and it's printing money."

---

## Counterarguments: Why Compute Parity Doesn't Mean the Valuation Makes Sense

**1. Pipeline is promises, not power.** 5 GW, 15 GW — those are contract numbers, not energized datacenters. Getting from signature to power-on requires land, grid connections, cooling, and labor. Every step can delay. [SemiAnalysis says Colossus 1's 300 MW was built in 122 days](https://x.com/SemiAnalysis_/status/2087667981031506158), but that's SpaceX speed, not the industry norm.

**2. Burning compute doesn't mean earning it back.** [Annualized revenue surged from $9B to $65B](https://x.com/StockSavvyShay/status/2089448527701369088), but compute spending is also $49.1B. Even with [inference margins of 50-65%](/barclays-ai-profit-chain-cloud-tax-inference-margin/), training spend is pure burn. A meaningful chunk of Q2's operating profit came from a one-time SpaceX compute deal.

**3. [Polymarket odds for an IPO by end of October have dropped from 90% to 62%.](https://x.com/oddsqcom/status/2096531774428082543)** Astra's community reception was better than expected. Baker himself acknowledged: "I think that Astra was probably better than Anthropic was expecting." Model leadership isn't a sufficient condition for an IPO; the timing window may be narrower than it looks.

---

## Reality Check

The compute data in this article comes with several important caveats.

**GW figures vary wildly depending on methodology.** Epoch AI's facility-level numbers (Anthropic 1.45 GW, OpenAI 0.42 GW) and spend-implied numbers (Anthropic 2.7 GW, OpenAI 2.4 GW) differ by several multiples. The gap comes from cloud-rented capacity, shared clusters, and training vs inference allocation. No single number is "correct" — each is an estimate under a specific methodology.

**SemiAnalysis spending figures are sell-side estimates, not audited numbers.** Neither company is public yet. The real compute spend is not disclosed. $49.1B and $54.6B are derived from supply chain intelligence.

**Baker's post is speculation, which he labeled "pure speculation" himself,** and one of his August claims was [directly denied by Anthropic's head of communications](https://x.com/sashadem/status/2088427297217110188).

But taken together, these data points accomplish one thing: **they transform "Anthropic has less compute than OpenAI" from an intuition into a question with verifiable numbers.** The answer: definitely behind in 2025, at parity by 2026. That doesn't make $2T reasonable, but it removes "not enough compute" as a valid argument against Anthropic.

---

## Key Insights

**1. The compute map is being redrawn faster than expected.** At the end of 2025, Anthropic had only 70% of OpenAI's capacity. In under a year, through Rainier (Trainium2), Google TPU, and the SpaceX Colossus lease, they've reached parity. If you're using vague notions of a "compute gap" to decide which API to bet on, the numbers have changed.

**2. The training/inference spend ratio reveals strategic direction.** Anthropic's inference spend ($24.1B) slightly exceeds OpenAI's ($22.6B); training is the reverse ($25B vs $32B). If post-IPO Anthropic redirects inference profits into training, inference API users won't face immediate price hikes — but long-term capacity priority may tilt toward high-value enterprise clients.

**3. Your API provider's S-1 will be the first public compute cost structure.** Once Anthropic goes public, its compute spending, inference margins, and customer concentration all become quarterly public filings. For the first time, you'll be able to calculate from public documents how much of your API bill is margin vs compute cost. The [Barclays profit chain framework](/barclays-ai-profit-chain-cloud-tax-inference-margin/) will move from estimates to verifiable numbers. Your API-vs-self-hosted spreadsheet is about to get a lot more precise.

---

## FAQ

**Q: Has Anthropic's compute really caught up to OpenAI?**

It depends on the methodology. At the end of 2025, Anthropic's running power was about 1.4 GW vs OpenAI's 1.9 GW — roughly 70% (source: Epoch AI report). By 2026, spend-implied annual average running power puts Anthropic at ~2.7 GW, slightly above OpenAI's ~2.4 GW. Epoch AI's more conservative facility-level numbers (Anthropic ~1.45 GW, OpenAI ~0.42 GW) show different proportions, but they acknowledge incomplete coverage. The conclusion: by 2026, both are in the same league, within plus-or-minus 20-30%, with the exact ranking shifting by month and methodology. "Anthropic has far less compute than OpenAI" is no longer a current fact.

**Q: Where does Anthropic's compute come from? How did they catch up?**

Three sources. First, AWS Project Rainier — a Trainium2 cluster in New Carlisle, Indiana with 500K chips online (announced by AWS CEO Andy Jassy in October 2025), targeting one million by year-end, single-site capacity around 900 MW. Second, Google TPU (Tensor Processing Unit) — five or more generations of Claude were trained on TPUs; 1 GW of Ironwood is deploying in 2026 with 5 GW more in 2027. Third, SpaceX Colossus 1 — roughly 300 MW with 220,000 NVIDIA GPUs, originally built by Elon Musk to train Grok, leased to Anthropic in May 2026. Together, these three pieces brought Anthropic from 70% to parity within a year.

**Q: How much are Anthropic and OpenAI each spending on compute in 2026?**

Per SemiAnalysis estimates, Anthropic's total 2026 compute spend is approximately $49.1B (training ~$25B, inference ~$24.1B). OpenAI's is approximately $54.6B (training ~$32B, inference ~$22.6B). OpenAI has publicly stated its 2026 compute spend is around $50B, consistent with the estimate. Annual cash burn is nearly matched; OpenAI leans heavier on training, Anthropic on inference. These are sell-side estimates, not audited figures — neither company is public yet.

**Q: What does compute parity have to do with Anthropic's $2T IPO valuation?**

Compute is the infrastructure layer beneath the IPO valuation story. When investors evaluate whether $2T is reasonable, "does this company have enough compute to sustain revenue growth" is a key question. If Anthropic's compute were clearly inferior to OpenAI's, high growth would lack infrastructure backing. With compute at parity, that objection is neutralized. But compute parity doesn't validate the valuation on its own — it also depends on whether inference margins can hold (Barclays estimates 50-65%), whether the $30T TAM (Total Addressable Market) assumption holds, and the timing risk from competitors releasing new models in rapid succession.

**Q: Did Fable 5 really get weaker over time because of compute constraints?**

Very possibly. The Information reported on September 6 that Claude Code and Cowork demand grew so fast that Anthropic itself was caught off guard, forcing an emergency compute scramble. When inference demand surges but compute capacity doesn't scale proportionally, model response quality degrades — not because the model itself was downgraded, but because inference resources are being diluted. Anthropic signing at least 14.8 GW (gigawatts) in compute contracts over the past eleven months is part of closing that gap. This also explains why compute distribution — not model benchmarks — is the more reliable indicator of an AI lab's long-term strength.

---

## Sources

- [Epoch AI: global AI compute distribution (via @Saemin4655, 2026-08-16)](https://x.com/Saemin4655/status/2088824774416072859)
- [Andy Jassy: Project Rainier announcement (2025-10-29)](https://x.com/ajassy/status/1983616724642730217)
- [SemiAnalysis: SpaceX Colossus build speed (2026-08-12)](https://x.com/SemiAnalysis_/status/2087667981031506158)
- [Milk Road AI: Anthropic-SpaceX Colossus deal (2026-05-09)](https://x.com/MilkRoadAI/status/2052986277977366547)
- [SpaceX rented Colossus after Grok training trouble (2026-06-12)](https://x.com/negligible_cap/status/2065502888973967710)
- [Gavin Baker original post (2026-09-05)](https://x.com/GavinSBaker/status/2096257640884027500)
- [Baker on All-In Podcast (2026-08-18)](https://x.com/GavinSBaker/status/2089729629695234462)
- [Glenn Solomon on Krishna Rao](https://x.com/glennsolomon/status/2096275708066840729)
- [FT via @kimmonismus: $2T valuation talk (2026-08-13)](https://x.com/kimmonismus/status/2087806611918073940)
- [Anthropic revenue trajectory (2026-08-17)](https://x.com/StockSavvyShay/status/2089448527701369088)
- [Polymarket: IPO odds 90% → 62% (2026-09-06)](https://x.com/oddsqcom/status/2096531774428082543)
- [Sasha de Marigny denial (2026-08-15)](https://x.com/sashadem/status/2088427297217110188)
- [WSJ: OpenAI + Anthropic combined spend (via Beth Kindig, 2026-04-22)](https://x.com/Beth_Kindig/status/2047014766678253791)
- [This blog: Barclays profit chain](/barclays-ai-profit-chain-cloud-tax-inference-margin/)
- [The Information: Anthropic 14.8 GW / $517B compute deals (via Techmeme, 2026-09-06)](https://x.com/Techmeme/status/2096682533106917555)
- [The Information: IPO investors demand revenue per token & GW (2026-09-06)](https://x.com/theinformation/status/2096697449133809907)
- [Choblin: Claude Code demand forced compute scramble (2026-09-06)](https://x.com/choblin29/status/2096638464792130011)
- [SemiAnalysis: Anthropic trained 5+ Claude releases on TPUs (2026-07-09)](https://x.com/SemiAnalysis_/status/2075203875062120785)
- [Broadcom Q3 FY26 earnings: Hock Tan on Anthropic TPU roadmap (via First Squawk, 2026-09-02)](https://x.com/FirstSquawk/status/2095260321397268918)
- [This blog: Not your weights, not your product](/not-your-weights-not-your-product/)
