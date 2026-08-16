---
layout: post
title: "YouTube Transcript: Two Days with Qwen3.8-27B — RTX 5090 SGLang / vLLM / llama.cpp Benchmarks"
date: 2026-08-16 11:00:00 +0800
permalink: /en/youtube-qwen38-27b-rtx5090-sglang-vllm-mtp-transcript/
image: /assets/images/youtube-qwen38-27b-rtx5090-mtp-thumbnail.jpg
image_alt: "Qwen3.8-27B on RTX 5090: SGLang vs vLLM MTP benchmark"
author: "Wisely Chen"
category: AI-Agent
tags:
  - Qwen 3.8
  - RTX 5090
  - SGLang
  - vLLM
  - MTP
  - speculative decoding
  - NVFP4
  - Local LLM
  - KV cache
description: "Qwen3.8-27B went open weights two days ago with SWE-bench Pro 61.7, beating Opus 4.6 Max's 53.4. Those are Alibaba's numbers — these are mine. I put it on a 32GB RTX 5090 and benchmarked SGLang and vLLM to answer one practical question: how many people can a single consumer card actually serve? Answer: four, at 425 tok/s total throughput, roughly 106 tok/s each. MTP is a free lunch — vLLM with MTP-2 is 73.5% faster single-user — but the same MTP head only gains 20% on the llama.cpp route. Full launch parameters included."
lang: en
---

> **Translation Note**
>
> This article is translated from the Chinese original. The author writes primarily in Traditional Chinese, and the original version is the canonical source — including the latest updates, comments, and follow-up discussions.
>
> **Read the Chinese original:** [YouTube 逐字稿：千問3.8-27B 用了兩天說說感覺——RTX 5090 SGLang/vLLM/llama.cpp 實測](https://ai-coding.wiselychen.com/youtube-qwen38-27b-rtx5090-sglang-vllm-mtp-transcript/)
>
> If you spot any translation errors or have feedback, please refer to the Chinese version as the source of truth.

**Author:** Wisely Chen
**Date:** August 2026
**Series:** AI Coding Field Notes — YouTube Transcripts
**Keywords:** Qwen3.8-27B, RTX 5090, SGLang, vLLM, MTP, Speculative Decoding, NVFP4, local LLM, KV cache, cloud-local hybrid

---

## What This Episode Covers

Two days ago Qwen3.8-27B went open weights, with official benchmarks showing SWE-bench Pro 61.7 — beating Opus 4.6 Max's 53.4. Those benchmarks are Alibaba's. This episode is about mine: I put the model on a 32GB RTX 5090, ran SGLang and vLLM each through a full round of testing, and answered one very practical question — **how many people can a single consumer card actually feed?**

The answer: four people, 425 tok/s total throughput, roughly 106 tok/s each. MTP is a free lunch: vLLM with MTP-2 is 73.5% faster single-user — but the same MTP on the llama.cpp route only gains 20%, and this episode cross-references a community GGUF test on that. All launch parameters are in this post.

---

**Length:** ~17 minutes

<iframe width="560" height="315" src="https://www.youtube.com/embed/F67glW4YKAI" title="Qwen3.8-27B RTX 5090 benchmark" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

### Timestamps

- 0:00 Opening: Qwen3.8-27B release background
- 2:07 SWE-bench Pro 61.7 beats Opus 4.6 Max; architecture carries over from 3.6
- 5:33 Hands-on coding impressions: harness impact > model impact
- 7:01 Test environment: SGLang / vLLM / llama.cpp
- 7:35 Why NVFP4? Community says 1-5% quality drop
- 8:25 vLLM + MTP testing
- 9:09 The 4-user concurrency sweet spot: ~106 tok/s each
- 10:17 SGLang vs vLLM framework differences
- 10:46 A local model at 90% of Opus 4.6's capability
- 11:45 DeepSeek V4 Flash + DGX Spark comparison
- 14:57 Cloud-local hybrid deployment architecture
- 17:07 Closing: unbeatable below consumer GPU tier, trades blows with cloud

---

## Full Transcript

### Opening

Hi everyone, back again this week.

Two days ago Qwen3.8-27B went open weights. I wrote a benchmark breakdown on the blog: SWE-bench Pro 61.7, beating Opus 4.6 Max's 53.4. Those are the official numbers — today is about my numbers. Yesterday I put it on my 32GB RTX 5090 and ran SGLang and vLLM each through a round of testing. The question is simple: **how many people can this card serve at the same time?**

### Environment: Squeezing It into 32GB with NVFP4

The model is RadixArk's Qwen3.8-27B-NVFP4 quantized build — 27B is the only way it fits in 32GB — running SGLang with MTP speculative decoding, generating 2 draft tokens at a time.

Why be this stingy about every megabyte? Because it's a dense model. Someone in the community ran the math: DeepSeek V4 Flash has 284B parameters but only activates 13 billion per token; this 27B puts every parameter on the field for every single token, so the actual compute per token is more than double V4 Flash's. **Dense model speed doesn't come free — you have to engineer it out.** Once running, the GPU is down to 1.2GB free, and every other service has to shut down first.

### Single User: 113.75 tok/s

Single-user test: SGLang averages 113.75 tok/s. You simply cannot read as fast as it outputs. The control group: same model, running on a 96GB unified-memory MacBook, a community tester got only 5 tokens per second. **Fitting in memory is not the same as actually running.**

But a whole 5090 for one person is a bit of a waste. The real point is the next round.

### Four Users Concurrent: This Card's Sweet Spot

Four people come in at once, each generating 2,048 tokens: everything finishes in 19 seconds, total throughput 425.44 tok/s, each person getting 106 to 108.

**Four people using it simultaneously, and each one gets nearly the same speed as running solo.** Single user 113, four users 106 each, nobody queuing. That's the sweet spot.

### Eight Users: Orders Accepted, Kitchen Overwhelmed

So I got greedy and went to eight. All eight requests completed, but total time was 54 seconds and throughput dropped to 303 — lower than the 4-user run. Completion times split visibly into three waves: 18 seconds, 37 seconds, 54 seconds. The KV pool ran out; only 3-4 people were actually generating at any moment while the rest queued. **You can take eight orders — you just can't cook for eight people.**

### vLLM and MTP: One Launch Flag, 73.5% Difference

Same model, moved to vLLM. First, a pothole: version 0.20.2 fails to load with an `lm_head.input_scale` error; upgrading to 0.27.1 fixes it.

A/B/C test where the only variable is MTP: without it, single user gets 57.83 tok/s; with MTP-2, 100.33. **That's 73.5% faster single-user and 67.2% more total throughput at four users — from one line in the launch command.**

But this gain depends heavily on your framework. A community tester went the llama.cpp/GGUF route on the same 5090: MTP only gained 20% on short context, and actually slowed things down past 100K context. His conclusion was "MTP doesn't do much on a 5090." Same MTP head — native serving implementation gets 73%, llama.cpp gets 20% and sometimes goes negative. **Whether MTP works is not a question for the model. It's a question of how well your framework implements it.**

### SGLang vs vLLM

Both running MTP-2: SGLang does 425 at four users, vLLM does 351 — 17% lower. If you're chasing raw speed, pick SGLang. If your ecosystem lives on vLLM, vLLM is fine — what I'm running in production right now is exactly the vLLM MTP-2 four-user configuration.

One more community observation: someone rented a 96GB RTX Pro 6000 to run vLLM with full BF16 weights and found single-user inference barely faster than much cheaper machines. That's because vLLM cares about throughput, not single-stream latency. **The value of these serving frameworks is redeemed through concurrency** — complain about single-user speed and you're missing the point; put four people on it and it pays for itself.

### Reality Check

Three things I have to say honestly. First, the SGLang tests used 2,048 tokens per person while vLLM used 512 — not a strict A/B, so treat the trends as comparable but don't memorize the percentages as gospel. Second, this is short-output throughput testing, not an agent workload, and radix cache was off — multi-agent long-context scenarios need separate testing. Third, I tested the quantized build — a community comparison found the Q5 quantized version produced buggy frontend code while the BF16 full-precision build was clean. **This episode answers speed, not quality.**

### Conclusion

One 32GB RTX 5090 is a model server for a four-person team, 106 tok/s each, with an experience barely distinguishable from having the card to yourself. MTP must be on — leaving it off means being 40%+ slower for nothing. And don't get greedy: eight users just makes everyone queue together. The KV pool is the 32GB ceiling.

Two days ago the story was that local deployment is no longer a compromise on benchmarks. Today it lands in serving numbers: the model is strong enough, the card fits it, and four people can use it smoothly. All three launch configurations are below — take them and run.

### How It Feels After Two Days

Honestly, I'm not sure why Alibaba is doing this, and I can't quite see through the business model either. But they keep throwing high-intelligence models that run on consumer GPUs out as open weights, and for the community — for a lot of people — that genuinely helps.

I've spent these two days very happily testing Qwen3.8-27B nonstop. Cloud models are strong today and might suddenly get nerfed tomorrow — you don't control the guardrails, the compute, or the pricing. Local gives you a different kind of stability: no guardrails, fixed hardware, predictable output. These two aren't mutually exclusive — they genuinely complement each other. Routine agentic automation runs local, and with 3.8-27B's intelligence now able to catch complex tasks too, a lot of work can move down from the cloud.

One-sentence verdict: **Qwen3.8-27B — unbeatable below the consumer GPU tier, trades blows one-for-one above it with cloud models.** Also looking forward to Qwen3.8-35B, hopefully soon.

That's all for today. Thank you, everyone.

---

## Appendix: Full Test Data

### Test Environment

- **GPU:** NVIDIA RTX 5090, 32GB VRAM
- **Model:** RadixArk/Qwen3.8-27B-NVFP4 (4-bit quantization, Blackwell hardware acceleration)
- **GPU usage:** ~30,825 MiB / 32,768 MiB (1,284 MiB free — Whisper/embeddings/ComfyUI must be stopped)
- **vLLM version:** 0.27.1 + PyTorch 2.13.0+cu130 + FlashInfer 0.6.16.post3 (0.20.2 fails to load with `lm_head.input_scale` error)

### SGLang Results

| Scenario | Output/User | Completion Time | Total Throughput | Per-User Throughput | MTP Acceptance | Token Pool |
|----------|-------------|-----------------|------------------|--------------------|----------------|------------|
| Single user | 512 tokens ×3 | ~4.5s/run | 113.75 tok/s | 113.75 tok/s | 78–88% | — |
| 4 concurrent | 2,048 tokens | 19.255s | **425.44 tok/s** | 106.37–108.62 | 80–91% | ~64,079 |
| 8 concurrent | 2,048 tokens | ~54s (3 waves: 18s/37s/54s) | 303.4 tok/s | — | — | ~7,703 |

### vLLM A/B/C Test (MTP comparison, 512 tokens per user)

| Config | Single-User tok/s | 4-User Total tok/s | vs No MTP (1U) | vs No MTP (4U) |
|--------|------------------|--------------------|----------------|----------------|
| No MTP | 57.83 | 210.37 | — | — |
| MTP-1 | 85.63 | 305.09 | +48.1% | +45.1% |
| MTP-2 | 100.33 | 351.71 | **+73.5%** | **+67.2%** |

MTP-2 draft acceptance: ~69% average (first draft token 79–80%, second 58–59%, mean accepted length 2.38–2.39).

### SGLang vs vLLM (both MTP-2)

| Framework | Single-User tok/s | 4-User Total tok/s | Note |
|-----------|------------------|--------------------|------|
| SGLang | 113.75 | 425.44 | 4-user test at 2,048 tokens/user |
| vLLM | 100.33 | 351.71 | 4-user test at 512 tokens/user |
| Gap | vLLM 11.8% lower | vLLM 17.3% lower | Different test conditions; trends comparable |

---

## Appendix: Three Launch Configurations

### SGLang, 4 users (recommended)

```bash
sglang serve \
  --trust-remote-code \
  --model-path RadixArk/Qwen3.8-27B-NVFP4 \
  --mem-fraction-static 0.98 \
  --attention-backend flashinfer \
  --chunked-prefill-size 2048 \
  --reasoning-parser qwen3 \
  --tool-call-parser qwen3_coder \
  --mamba-full-memory-ratio 4.59 \
  --host 0.0.0.0 \
  --port 30000 \
  --max-running-requests 4 \
  --speculative-algorithm NEXTN \
  --speculative-num-steps 1 \
  --speculative-eagle-topk 1 \
  --speculative-num-draft-tokens 2 \
  --cuda-graph-max-bs-decode 4 \
  --disable-radix-cache
```

### SGLang, 8 users (accepts more orders; long outputs get batched)

Same as above with two changes: `--max-running-requests 8` and `--cuda-graph-max-bs-decode 8`.

### vLLM MTP-2 (0.27.1, currently running in production)

```bash
vllm serve RadixArk/Qwen3.8-27B-NVFP4 \
  --trust-remote-code \
  --served-model-name Qwen3.8-27B \
  --host 0.0.0.0 \
  --port 30000 \
  --gpu-memory-utilization 0.95 \
  --max-model-len 32768 \
  --max-num-seqs 4 \
  --max-num-batched-tokens 8192 \
  --attention-backend FLASHINFER \
  --reasoning-parser qwen3 \
  --enable-auto-tool-choice \
  --tool-call-parser qwen3_coder \
  --speculative-config '{"method":"mtp","num_speculative_tokens":2}'
```

---

## Further Reading

- [Qwen3.8-27B Open Weights: SWE-bench Pro 61.7 Beats Opus 4.6 Max — 'Just Use On-Prem' Is No Longer a Compromise](/en/qwen-3-8-27b-open-weights-local-security/)
- [Gemma 4's 3x Speedup: Speculative Decoding Isn't New, But Google Shipped the Whole Drafter as Apache 2.0](https://ai-coding.wiselychen.com/gemma4-mtp-drafter-speculative-decoding-open-source/) (Chinese)
- [Running GLM 5.2 on a Single Machine: RTX Pro 6000 and the Hunt for Cost-Effective Tier-1 Local Models](https://ai-coding.wiselychen.com/glm-52-single-machine-rtx-pro-6000-tier1-local/) (Chinese)
- [RTX 5090 Price Up 70% in Three Months: Five Technical Paths to Lower the Local AI Hardware Bar](https://ai-coding.wiselychen.com/memory-price-surge-local-ai-five-paths/) (Chinese)
- [GPU Prices Up Yesterday, DeepSeek API Up Today — Cloud-Local Hybrid as a Hedge](https://ai-coding.wiselychen.com/deepseek-v4-api-price-hike-subsidy-end/) (Chinese)
