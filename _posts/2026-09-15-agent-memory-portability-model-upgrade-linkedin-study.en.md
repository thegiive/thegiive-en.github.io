---
layout: post
title: "Does Your Agent's Memory Survive a Model Upgrade? A LinkedIn Study Says the Average Hides Two Opposite Effects: +9.91 and −13.28"
date: 2026-09-15 09:00:00 +0800
permalink: /en/agent-memory-portability-model-upgrade-linkedin-study/
image: /assets/images/agent-memory-portability-model-upgrade-cover.png
image_alt: "Study design diagram showing synthetic histories converted into four memory formats with migration and evaluation paths"
author: "Wisely Chen"
category: AI Agent
tags:
  - Agent Memory
  - Memory Portability
  - Model Upgrade
  - RAG
  - Knowledge Graph
  - Embedding Migration
description: "Fable 5.1 and GPT-6 Astra just launched. Many agents have already swapped their underlying model, but their memory stores were written by the old one. A LinkedIn paper (arXiv 2609.05339) ran a controlled experiment: 48 synthetic histories, four memory formats, Llama-3.1-8B and Qwen2.5-7B swapping writer and reader roles. Natural-language notes show only −1.47pp average swap penalty — but split by direction it's +9.91 and −13.28. Eighty percent of notes loss happens at write time. Mixed embeddings lose 58% of upgrade gains. Notes-only repair fails 0/48; raw-history repair recovers 34. This post breaks down the experimental design, four key tables, and what I'm changing in my own 263-page wiki."
lang: en
---

> **Translation Note**
>
> This article is translated from the Chinese original. The author writes primarily in Traditional Chinese, and the original version is the canonical source — including the latest updates, comments, and follow-up discussions.
>
> **Read the Chinese original:** [換了模型，agent 的記憶還在嗎](https://ai-coding.wiselychen.com/agent-memory-portability-model-upgrade-linkedin-study/)
>
> If you spot any translation errors or have feedback, please refer to the Chinese version as the source of truth.

Same agent memory. When Llama reads notes written by Qwen, accuracy is 9.91 percentage points higher than when Llama reads its own notes. Reverse the direction — Qwen reading Llama's notes — and accuracy drops 13.28 percentage points below Qwen reading its own.

Average the two directions: −1.47. If you only look at the average, the conclusion is "swapping models doesn't matter."

These are the core numbers from [Does Your Agent's Memory Survive a Model Upgrade?](https://arxiv.org/abs/2609.05339), posted to arXiv on September 4. Both authors use LinkedIn email addresses; no university or lab affiliation on the front page. The question the paper asks is engineering, not science: models get swapped every few months, memory stores accumulate over many months — after a swap, can the old memories still be used, how much is lost, and can it be repaired?

This month is exactly when that question matters most. Fable 5.1 and GPT-6 Astra launched back to back. Many agents have already swapped their underlying model, but their memories were written by the old one.

---

## 30-Second Overview

| Item | Details |
|------|---------|
| Paper | Does Your Agent's Memory Survive a Model Upgrade? A Controlled Study of Memory Portability, [arXiv 2609.05339](https://arxiv.org/abs/2609.05339) |
| Authors | Ankit Goyal, Jaideep Ray — LinkedIn email addresses |
| Models | Llama-3.1-8B-Instruct and Qwen2.5-7B-Instruct-1M, locally served fixed versions, swapping writer and reader roles |
| Data | 48 synthetic histories, 160 questions each, answers are random codes, exact-match scoring — no LLM judge |
| Four memory formats | LC-RAW (full raw history in context), RAG (chunked vector retrieval), NOTES (model-compressed natural-language notes, 128 KiB cap), KG-fixed (fixed-schema subject-predicate-object triples) |
| Embedding migration | bge-large-en v1.0 to v1.5, both 1024-dimensional — mixing them in one index doesn't error |
| Scale | ~2,900 model responses total; repair experiments: 1,440 evaluations, 98.2 GPU-hours, ~$221 |
| Code | Available on request — no public repo |

The paper decomposes agent memory into four roles: the actor that generates experiences, the writer that stores them as memory, the reader that later answers questions from memory, and the embedder that handles vector retrieval. An upgrade might swap any of these. The experimental design swaps exactly one at a time, holding everything else fixed.

---

## What Previous Benchmarks Missed

Previous memory benchmarks — LoCoMo, LongMemEval, BEAM — all test whether a single system "remembers." They share a hidden assumption: the model that wrote the memory is the same model that reads it.

This paper asks a different question. It fixes the history and swaps the model underneath: Llama's memories read by Qwen, compared to Qwen reading its own memories. The difference — which the paper calls Retained Performance After Swap — measures migration loss, not reader capability.

Another design choice: fully synthetic data. All 48 histories are script-generated with randomized entity names and value codes. The model can't know any answer from pretraining; chance accuracy is near zero. The trade-off is unrealistic conversations. The payoff is that every point of loss can be traced to a specific pipeline stage.

One more thing, rare in agent papers: before running the main experiments, the authors locked four hypotheses, a 5-percentage-point threshold, and multiple-testing corrections using a signed git tag. Two of the four hypotheses didn't pass their threshold. The paper reports them anyway.

---

## Building Intuition: Who Wrote the Handover Notes?

Imagine inheriting a project from a colleague who just left. They left two things: a set of "key notes" they wrote up, and the full Slack history plus commit log.

How useful the notes are depends on what they included when writing them. The API rate limit they didn't think was important? You can read the notes ten times and you won't find it. Rewriting the notes in your own words won't make it appear either. The only way is to go back to the raw Slack thread.

If they'd left a filled-in form instead — with fixed fields for owner, deadline, dependencies, status — you wouldn't need to guess their writing style. The fields are predetermined; they just filled in values.

The paper's four formats map to these three things. NOTES are the handover notes. LC-RAW and RAG are the raw Slack history. KG-fixed is the filled-in form. The experiment measures: when someone new reads them, which format loses the most?

---

## Table 1: The Average Hides Two Opposite Effects

| Format | Reader | Self-written | Cross-written | Difference (pp) |
|--------|--------|-------------|---------------|-----------------|
| NOTES | Llama | 0.3762 | 0.4753 | +9.91 |
| NOTES | Qwen | 0.4719 | 0.3391 | −13.28 |
| KG-fixed | Llama | 0.8456 | 0.8445 | −0.11 |
| KG-fixed | Qwen | 0.9878 | 0.9880 | +0.02 |

Fixed-schema swaps move only +0.0004 across both readers. Natural-language notes depend entirely on direction: Qwen's notes read by Llama score 9.91pp above Llama's self-written baseline; Llama's notes read by Qwen score 13.28pp below Qwen's baseline.

The paper's phrasing is direct:

> "Do not average away the migration direction."

The −1.47 average doesn't mean "notes are interchangeable." It means two large effects in opposite directions happen to cancel. This is also why pre-registered hypothesis H1 didn't pass its threshold: H1 tested symmetric-average penalty, estimated at −1.47, far from the 5pp threshold. If the authors had only reported pre-registered results, the headline would have been "notes are portable." Splitting by direction revealed the problem.

Why this direction? During calibration, one number stands out: given the same budget cap, Qwen's notes retain 85.6% of the required evidence, while Llama retains only 64.5% — and Llama uses more space. The writer decides what goes into the notes and what doesn't. Notes left by a low-retention writer can't be compensated for by any reader. The paper's practical rule:

> "The practical rule is to test each writer→reader direction and treat the note-writing model's quality as an inseparable part of the memory design."

The note-writing model is part of the memory design. Swap the writer, and you've swapped the memory.

---

## Table 2: Where the Loss Happens

The paper ran a diagnostic experiment: normal reading, then directly feeding the reader the stored items containing the answer, then directly feeding the raw events. The differences between the three scores reveal how much each stage loses.

| Format | Total loss | Write-time loss | Retrieval loss | Reader residual |
|--------|-----------|-----------------|----------------|-----------------|
| NOTES | 0.584 | 0.467 (80%) | 0.036 (6%) | 0.081 (14%) |
| RAG | 0.450 | 0.005 (1%) | 0.364 (81%) | 0.081 (18%) |

The two formats have completely opposite loss profiles. Notes lose 80% of their accuracy at write time; the reader accounts for only 14%. RAG loses 81% at retrieval; the stored chunks themselves lose almost nothing — feed the correct chunks directly to the reader and accuracy jumps from 0.53–0.56 to 0.88–0.95.

This gives you a debugging order. When an agent starts "forgetting" after a model swap, the engineer's instinct is to blame the new model. The paper says: don't touch the reader first. For notes, measure evidence coverage at write time. For RAG, measure retrieval recall. Evidence the reader never receives can't be answered by any reader, no matter how capable.

The paper also tested an obvious fix: have the new model rewrite the old notes in its own style. Accuracy actually dropped 0.012 to 0.042. The problem isn't stylistic incompatibility — it's that the content was never written down in the first place.

---

## Table 3: Mixed Embeddings Are the Worst Option

| Index | Accuracy | vs. old index (pp) |
|-------|----------|---------------------|
| Old embedding (v1.0) | 0.4257 | — |
| Full re-embedding (v1.5) | 0.5447 | +11.90 |
| Half old, half new mixed | 0.4753 | +4.96 |
| Ideal routing (oracle knows which index has the answer) | 0.5960 | +17.03 |

Full re-embedding gains 11.90 percentage points. The half-old-half-new mixed index gains only 4.96 — losing nearly 60% of the upgrade benefit. This is pre-registered hypothesis H2, estimated at +6.95, which passed.

The dangerous part is that it doesn't error out. v1.0 and v1.5 both output 1024-dimensional vectors. Stuff them into the same index and the database returns results as usual — but old vectors and new vectors live in different spaces, and similarity rankings go haywire. The paper recommends building a completely new index and testing before cutover. If you must migrate incrementally, keep two separate vector spaces and route queries by index version.

Worth noting: this RAG setup had problems before the embedding swap — the retriever only found the right evidence about 60% of the time, regardless of reader. The paper explicitly states this was a deliberately minimal single-stage dense retriever with no reranker, and does not represent RAG's ceiling.

---

## Table 4: Notes-Only Repair Fails; Raw History Is the Second Chance

| Repair method | Cases reaching 90% | Median cost per case |
|---------------|-------------------|---------------------|
| NOTES, rewrite from existing notes | 0/48 (both directions) | — |
| NOTES, rebuild from raw history, Qwen repairs | 34/48 | $0.76 |
| NOTES, rebuild from raw history, Llama repairs | 0/48 | — |
| RAG, full re-embedding | 48/48 | $0.013 |
| KG-fixed, rebuild from schema | 91–96/96 | ~zero |

Rewriting from existing notes alone: zero out of 48 histories reach the 90% recovery target, across both directions and all three thresholds. Evidence already dropped from the notes doesn't come back no matter how many times you rewrite.

Rebuilding from raw history, with Qwen as the repair model: 34 out of 48 reach 90%, at a median cost of $0.76 per case. Switch to Llama as the repair model: zero recoveries, all hitting output length limits. So raw history is necessary but not sufficient — the repair model also needs to complete the rebuild within its context and output constraints.

Structured formats are much cheaper to repair. RAG re-embedding recovers all 48, at ~$0.013 each. Fixed-schema rebuilds recover 91–96 out of 96, at near-zero cost.

> "Do not discard raw history just because a compact store exists."

The paper doesn't dodge the cost: keeping raw history carries privacy, security, and retention obligations. Their recommendation: encrypt, restrict access, set an explicit deletion schedule.

---

## What This Changes for Me

I maintain a blog knowledge base called llm-wiki: 173 raw-article markdowns plus 263 machine-generated summary pages, 26 concept pages, and 14 entity pages. I checked the frontmatter: not a single one of the 263 summary pages records which model wrote it. These summaries were created between mid-July and mid-September. I switched models during that period, but which page was written by which model? No record.

Mapped to this paper, those summary pages are exactly the least-portable NOTES format, and I don't even know which writer produced which page — so I can't test migration direction at all. The good news: I kept the raw articles. Every summary's `sources` field points back to the raw directory, so Table 4's repair path is open to me.

My previous approach: swap models, keep the summaries, they're readable enough. After reading this paper, four things to change:

- **Add a writer field to every summary page.** The paper's playbook item #4: record provenance — model, prompt, schema, embedding, chunking config. This is the cheapest step, and without it, the other three steps are impossible.
- **Test on swap, and test each direction.** The paper's appendix has a 20-question quick screen with 0.860 correlation to the full evaluation, but ~0.10 error — good enough to catch obviously bad migrations, not for sign-off. For me: after a model swap, randomly sample twenty summaries, generate questions from the raw articles, and quiz the new model.
- **Never delete raw articles.** I was already doing this, but the reason changed. Before: convenience for lookups. Now: it's the only repair path.
- **After upgrading the model, regenerate summaries from raw articles.** The paper's loss decomposition shows 80% of NOTES loss happens at write time. The bottleneck is the writer, not the reader. Information discarded during the old model's compression can't be read back by any new model, no matter how capable. So treat summaries as cache: raw/articles/ is the asset, wiki/summaries/ is the cache. On model upgrade, rerun generation from raw, letting the new model compress with its own logic. This beats "letting the new model read old summaries" or "restyling old summaries" — Table 4 shows notes-only rewrite fails all 48 cases, while raw-history rebuild recovers 34.

For anyone building an enterprise agent platform, the same logic applies in a different position. Claude's memory export/import, Pinecone and Weaviate's index migration docs all exist — what's missing is measuring how many points you lose. Before a model swap, run "new model reads old memories" against "new model reads its own memories," test both directions. For embedding swaps, rebuild the entire index — don't mix incrementally. When choosing memory format, anything that can be stored as fixed fields shouldn't be stored as free text.

---

## The Counter-Argument: This May Prove Less Than It Appears

The strongest objection is scale. Llama-3.1-8B and Qwen2.5-7B are not what anyone uses as an agent backbone in 2026. The asymmetry comes from Llama's 64.5% evidence retention when writing notes — a frontier model's retention rate might approach 100%, and the +9.91/−13.28 gap might vanish entirely. The paper's own limitations section acknowledges: larger models, subjective real conversations, and histories that don't fit in context may have different portability profiles.

Second, the synthetic histories use random codes as answers, testing whether exact identifiers survive. Real agent memory's value often lies in fuzzy preferences and context — that kind of "loss" is hard to measure with exact matching.

Third, fixed-schema stability was tested on a schema the authors designed for their synthetic task. The paper explicitly says this doesn't generalize to knowledge graphs broadly. In the real world, what the schema should look like is itself a hard problem, and KG-fixed was excluded from the diagnostic experiment because feeding its items individually actually performed worse than normal retrieval.

Fourth, two of the four pre-registered hypotheses didn't pass. The headline "asymmetry" finding came from splitting directions post-hoc, outside the pre-registered tests.

My response: point one is a real limitation, which is why this article cites the method — "test each direction separately" — rather than the specific numbers. The method doesn't depend on model size. Points two and three narrow the scope of the conclusions, not their direction. Point four is actually a strength: they locked hypotheses, reported failures, and labeled post-hoc analysis as post-hoc. That's more honest than most agent papers.

---

## Reality Check

This is an experiment on 7B-class models with 48 synthetic histories, two authors, one cross-family direction, and code available on request. Every number in this paper holds only within its specific setup.

The RAG section's 40% retrieval miss rate was measured with a deliberately minimal setup. Swap in a pipeline with a reranker and the mixed-index penalty might not reach 60% — the paper didn't test it. Raw-history repair succeeded in only one direction; Llama as repair model failed completely. So the value of "keep raw data" also depends on whether your repair model can actually finish the job.

My own part needs to be clear too: the four changes I listed for llm-wiki are derived from the paper. I haven't implemented any of them yet. The 263 summary pages lacking writer records is a verified fact, but how much accuracy those summaries actually lose after a model swap — I have no data. I'll update after running the sample test.

But the paper did one thing right. It turned "swapping models" from a deployment action into a measurable migration, and decomposed the loss into write, retrieval, and read stages. That's more useful than yet another memory framework claiming two more points on LoCoMo, because those scores all assume the model never changes.

---

## Key Insights

**A model upgrade is a memory migration, not a parts swap.** The paper's conclusion opens with this. Before swapping, run "new model reads old memories" against "new model reads its own memories," testing both directions. Looking at the average alone will hide large effects in opposite directions.

**Eighty percent of notes loss happens at write time — measure the writer's evidence retention first.** When an agent starts forgetting after a swap, don't blame the reader first. For notes, check write-time evidence coverage. For RAG, check retrieval recall. Restyling old notes with the new model doesn't help.

**When upgrading embedding versions, rebuild everything — don't mix.** Half-old-half-new loses nearly 60% of the upgrade gains, and it doesn't error. If you must migrate incrementally, keep two separate indexes and route queries by version.

**Raw data is the only repair path, but confirm the repair model can finish the job.** Notes-only rewrite fails all 48 cases. Raw-history rebuild recovers 34, but a different repair model recovers zero. Keep raw data, but also write deletion schedules and access controls into the design.

**Summaries are cache, not assets.** Eighty percent of loss happens at write time; the writing model matters more than the reading model. After a model upgrade, regenerate summaries from raw sources — don't let the new model read old summaries. Raw articles are the asset; summaries can always be rebuilt.

**Recording provenance is the cheapest step.** Which model, which prompt, which embedding wrote each piece of memory — record it now. Without it, none of the above five steps are possible.

---

## FAQ

**Q: What does this paper test?**

It tests agent memory portability: after swapping the underlying model, how much accuracy do old memories lose when read by the new model, and can they be repaired? The method fixes 48 synthetic histories and decomposes memory into writer, reader, and embedder roles, swapping one at a time. Four memory formats are compared: full raw history, chunked vector retrieval (RAG), model-compressed natural-language notes, and fixed-schema knowledge graphs. Models are Llama-3.1-8B-Instruct and Qwen2.5-7B-Instruct-1M, swapping both directions.

**Q: The paper says "average shows no difference" is misleading — why?**

The average swap penalty for natural-language notes is only −1.47 percentage points, but split by direction: Qwen writing for Llama to read is +9.91, Llama writing for Qwen to read is −13.28 — two large effects canceling out. The cause: different writers retain different amounts of evidence. Given the same budget, Qwen's notes retain 85.6% of required evidence while Llama retains only 64.5%. Migration must be tested per direction, not averaged.

**Q: Can old and new embedding vectors coexist in the same index?**

The paper tested this and the answer is no. Upgrading bge-large-en from v1.0 to v1.5, full re-embedding gains 11.90 percentage points while a half-old-half-new mixed index gains only 4.96 — losing nearly 60% of the upgrade benefit. Both versions output 1024-dimensional vectors, so mixing doesn't cause errors, but the vectors occupy different spaces and similarity rankings break. Recommendation: build a fresh index, test, then cut over. For incremental migration, keep two separate indexes and route queries by version.

**Q: Can the new model recover lost memories by rewriting old notes?**

No. The diagnostic experiment shows 80% of NOTES loss occurs at write time, with the reader accounting for only 14%. Having the new model restyle old notes actually reduces accuracy by 0.012 to 0.042. Notes-only repair fails all 48 histories — none reach the 90% recovery target. Rebuilding from raw history is the only viable path: Qwen as repair model recovers 34/48 at a median cost of $0.76 per case.

**Q: Do these findings hold for frontier models?**

Unknown. The paper tested only 7B and 8B models, 48 synthetic histories, and one cross-family direction. The authors acknowledge in their limitations section that larger models and real conversations may show different portability profiles. The asymmetry stems from weak writers retaining less evidence — frontier models may not have this problem. What transfers regardless of model size: the method of testing each direction separately, decomposing loss into write/retrieval/read stages, and recording each memory's writer provenance.

---

## Sources

- Paper: [Does Your Agent's Memory Survive a Model Upgrade? A Controlled Study of Memory Portability](https://arxiv.org/abs/2609.05339), arXiv 2609.05339, 2026-09-04
- Related posts on this blog: [Agent Memory Benchmark Rashomon](https://ai-coding.wiselychen.com/agent-memory-benchmark-rashomon-filesystem/), [MemHarness Memory Reconstruction](https://ai-coding.wiselychen.com/agent-memory-reconstruction-memharness/), [LifeMem Workflow Clustering](https://ai-coding.wiselychen.com/lifemem-lifelong-experience-reuse-workflow-clustering/), [NVIDIA Cross-Model KV Cache Transfer](https://ai-coding.wiselychen.com/cross-model-kv-cache-transfer-nvidia-ridge-regression/)
