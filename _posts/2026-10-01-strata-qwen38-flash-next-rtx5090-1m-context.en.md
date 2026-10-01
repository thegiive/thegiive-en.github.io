---
layout: post
title: "Running Qwen3.8-Flash-Next on an RTX 5090 + 64GB: Strata Hands-On, IQ3_XXS vs IQ3_S"
date: 2026-10-01 09:00:00 +0800
last_modified_at: 2026-10-01 16:00:00 +0800
permalink: /en/strata-qwen38-flash-next-rtx5090-1m-context/
image: /assets/images/strata-qwen38-flash-next-rtx5090-cover.png
image_alt: "Strata splits a 125B model across VRAM, RAM and SSD on one RTX 5090: 101.9 tok/s, a near-1M-token document test passed, 5.8 GiB of RAM left"
author: "Wisely Chen"
category: AI Architecture
tags:
  - Qwen3.8-Flash-Next
  - Strata
  - RTX 5090
  - Local LLM
  - MoE
  - Quantization
  - FreeToken
  - Open Source
description: "Qwen3.8-Flash-Next beats the Qwen3.8-27B I use every day on most rows of Qwen's own benchmark table (DeepSWE 1.1: 58.7 vs 42.2), but a 125B backbone plus a huge n-gram table left my RTX 5090 + 64GB RAM stuck between wanting to try it and not being able to run it. Even FreeToken's setup blew past my memory budget. The open-source engine Strata fit it in with four tricks: low-bit quantization, an expert cache, the n-gram table on SSD, and MTP. IQ3_XXS stretched context to 1M and generated 101.9 tok/s on a short prompt; IQ3_S pulled back to 262K, ran real agent requests at a median of 106.8 tok/s, and made fewer everyday mistakes. Here are both runs, and why I stayed on IQ3_S."
lang: en
---

> **Translation Note**
>
> This article is translated from the Chinese original. The author writes primarily in Traditional Chinese, and the original version is the canonical source — including the latest updates, comments, and follow-up discussions.
>
> **Read the Chinese original:** [Qwen3.8-Flash-Next 跑在 RTX 5090 + 64GB：Strata 實測 IQ3_XXS vs IQ3_S](https://ai-coding.wiselychen.com/strata-qwen38-flash-next-rtx5090-1m-context/)
>
> If you spot any translation errors or have feedback, please refer to the Chinese version as the source of truth.

Time for another guaranteed-boring IT day.

I've wanted to try Qwen3.8-Flash-Next for a while. On the official benchmarks, it beats the 27B I use every day on a lot of rows:

| Benchmark | Qwen3.8-Flash-Next | Qwen3.8-27B |
|------|------:|------:|
| DeepSWE 1.1 | 58.7 | 42.2 |
| JobBench | 55.7 | 33.4 |
| SWE-bench Multilingual | 81.0 | 73.8 |
| Toolathlon Verified | 73.5 | 67.1 |
| SWE-bench Pro | 62.5 | 61.7 |
| GPQA Diamond | 91.7 | 89.2 |

The biggest gaps are all agent work: fixing repos, running tools, finishing a whole task end to end.

And yet my RTX 5090 + 64GB RAM was stuck right where I wanted to try it and couldn't run it.

---

## The 30-Second Overview

| Item | Numbers |
|------|------|
| Model | Qwen3.8-Flash-Next, released August 26, Qwen Community License |
| Parameters | 125B total, only 6B active per token, plus a 51B n-gram embedding table and a 4B MTP module |
| MoE layout | 512 experts per layer; each token uses 10 routed experts plus 1 shared expert |
| Context | 262,144 tokens native; Qwen says it can be extended to 1M |
| Engine | Strata, open-sourced by Niko1221 (MIT); first commit September 24, already at 0.1.30 by September 30 |
| Quantization | IQ3_XXS (RAM+VRAM requirement 47.0GB), IQ3_S (54.8GB) |
| My machine | Core Ultra 7 265K, 64GB DDR5-4800, RTX 5090 32GB, CUDA 12.9 |

---

## Where It Got Stuck: 125B, Neither Big Nor Small

A quantized Qwen3.8-27B fits on the card and runs smoothly day to day. On this same 5090 in August, [Qwen3.8-27B](/en/youtube-qwen38-27b-rtx5090-sglang-vllm-mtp-transcript/) did 113.75 tok/s for a single user and 425.44 tok/s total throughput with four users in parallel.

Flash-Next is a 125B backbone with about 6B active per token, plus a huge n-gram vector table. The whole thing doesn't fit on a 32GB card.

For squeezing MoE models onto gaming GPUs, the newest academic answer is FreeToken. It went up on arXiv on August 17, led by UC Berkeley, with Ion Stoica, Song Han and Matei Zaharia among the authors. The paper claims a single 5090 runs the 284B DeepSeek V4 Flash at 22–25 tok/s. It supports Flash-Next too, but it won't run on my machine:

- It keeps every expert in host RAM; the 5090 box in the paper has 192GB of DDR5
- For Flash-Next it only takes FP8 or NVFP4, and the NVFP4 checkpoint is still 135GB
- That n-gram lookup table alone needs 47.7 GiB pinned in host RAM

32GB of VRAM plus 64GB of RAM is 96GB total. A 135GB model doesn't fit from the start.

Just add RAM? In August I wrote about [32GB of DDR5 going from about NT$3,000 to nearly NT$13,000](https://ai-coding.wiselychen.com/memory-price-surge-local-ai-five-paths/) here in Taiwan. Spending real money on an upgrade just to try one model — I couldn't bring myself to do it.

Until recently, when I found Strata.

---

## How Strata Makes It Run

![Strata's GitHub README: one 12–24GB NVIDIA card plus 64GB of RAM runs a 125B model](/assets/images/strata-github-readme-1.jpg)

Strata is an open-source inference engine built by one very capable developer, aimed at consumer PCs with RTX 20- to 50-series cards. The first line of its README says it: one 12–24GB NVIDIA card plus 64GB of RAM, running a 125B model. It has 2.9k stars on GitHub (as of October 1). Reading through it, I realized I'm not the only one who wants big models without rebuilding the whole PC.

It combines four techniques so the GPU, CPU, RAM and SSD share the work.

### 1. Low-Bit Quantization

First, shrink the weights with IQ3 formats. Both are 3-bit quantizations:

- **IQ3_XXS**: about 3.06 bits per weight by llama.cpp's definition — the most aggressive. RAM+VRAM requirement: 47.0GB
- **IQ3_S**: about 3.44 bits per weight, keeping a bit more precision. RAM+VRAM requirement: 54.8GB

These are ISTA-DASLab's retuned versions, where different layers get different precision. Strata's author describes IQ3_S as the best quality, matching the original model on the published tests.

### 2. Expert Cache and CPU/GPU Split

The full model has 24,576 experts (48 layers × 512), and each token calls only 10 of them.

- **All expert weights live in RAM**
- **The frequently used ones are cached in VRAM**
- **During generation, the GPU computes the cached experts while the CPU handles the misses at the same time**

The cache also adapts to actual usage, so the experts sitting in VRAM track whatever you're working on right now. In the log from the 0.1.29 upgrade on 9/30, the GPU expert cache hit rate was 93.4%: 93 out of 100 calls were handled on the graphics card.

This corrects the conclusion I reached in July in [MoE Offload, Fully Explained](https://ai-coding.wiselychen.com/moe-offload-deepseek-v3-v4-local-inference-optimization/):

> MoE offload 場景下，速度 = f(VRAM 能留多少 expert)。大 VRAM 不是為了塞下模型，是為了少搬 PCIe。

In English: with MoE offload, speed is a function of how many experts VRAM can hold, and big VRAM isn't for fitting the model, it's for moving less over PCIe. Back then the assumption was llama.cpp's approach: missed experts get moved across PCIe so the GPU can compute them. Strata doesn't move them; the CPU computes them in place. The first half still holds — more VRAM means more experts stay resident. The second half needs a rewrite: big VRAM isn't just about moving less over PCIe, it's about calling on the CPU less.

### 3. The n-gram Table Lives on SSD

Flash-Next has a 29GB n-gram lookup table. It stores vectors for sequences of tokens the model has learned, and what comes out of a lookup still goes back into the model for further computation.

FreeToken pins the entire table in RAM. Strata leaves it on the SSD and reads only the rows it needs. On a 64GB machine, that one decision is the line between runs and doesn't run.

### 4. MTP Speculative Decoding

An old friend. A small model guesses the next few tokens and the main model verifies them in one pass; correct guesses let it advance several steps at once. The author says it's 1.6–1.8x faster. In my run last night, it guessed 191 tokens and 76 were accepted, about 40%.

### The Value Is in the Integration

Hybrid CPU/GPU inference, quantization, speculative decoding — each has prior art. Strata's value is engineering them together around Flash-Next's specific traits, so one home computer's hardware shares the load.

More importantly, it hits a real pain point for a lot of people: a good graphics card, only 64GB of RAM, and no appetite for upgrading the whole machine for one model.

---

## My Tests

Same machine as always: RTX 5090 32GB, 64GB DDR5-4800, Core Ultra 7 265K.

### IQ3_XXS: Pushing Context All the Way to 1M

I started with IQ3_XXS on Strata 0.1.30 and set the context limit to 1M:

- `--rope-scaling yarn --rope-scale 4`: stretch the native 262K by 4x with YaRN
- `--kv q4_0`: drop the KV cache from int8 to 4-bit, or it won't fit in 64GB of RAM

On a short prompt generating 512 tokens, it ran at 101.9 tok/s.

Then a long-document test, known as needle-in-a-haystack: take a pile of unrelated filler text, insert a line like "the verification code is violet-harbor-731" at the 10%, 50% and 90% positions, and ask the model for all three codes. If it didn't actually read the document, it can't answer.

| Document length | Three codes | Time to answer | Implied read speed |
|---|---|---:|---:|
| 300K tokens (299,513) | All correct | 76 s | about 4,000 tokens/s |
| 999K tokens (999,490) | All correct | 344 s, nearly six minutes | down to about 2,900 tokens/s |

A 1M-token input really did go through. The cost: only 5.8 GiB of RAM left during that run.

Once it was wired into OpenClaw, real requests ran at an actual context of about 185K–188K, with prefill at 3,278–3,642 tok/s and long-output generation at 87.2–89.6 tok/s.

### IQ3_S: Pulling Context Back to 262K

IQ3_S takes 7.8GB more than IQ3_XXS. With only 5.8 GiB of RAM left during the 1M test, those 7.8GB don't fit. So I set the context limit back to 262K, switched the KV cache back to int8, and turned YaRN off.

After the switch, 32.81 GiB of experts stay resident in RAM, and the GPU cache has 9,494 slots.

In a little over an hour, it served 41 real agent requests:

- **Decode (generation) median: 106.8 tok/s**, range 31.8 to 170.6
- **Prefill (reading the prompt): roughly 3K–4K tokens per second** (3,160 to 4,250)
- Each prompt was 50K–67K tokens, about 64K

So these speeds come from real requests of about 64K tokens, not a benchmark at a full 262K. And the first 49,152 tokens were reused from the previous turn, so you can't treat the whole prompt as freshly read.

For now I'm staying on IQ3_S + 262K. 1M was the highlight of this round of testing, but for day-to-day OpenClaw work I'd rather keep a little more weight precision and memory headroom.

---

## Why I Ended Up on IQ3_S

Using both for real, I could feel the difference.

In OpenClaw, IQ3_XXS hit errors more often when writing code or reading screenshots. Because the agent retries automatically, things rarely failed outright, but there were error messages. After switching to IQ3_S and pulling context back, those problems dropped a lot.

That's my own usage observation. The two periods had different context limits, KV precision and tasks, so I can't attribute all of the improvement to IQ3_S.

Third-party numbers point the same way. On X, Nigel Hungerford-Symes ([@VectorCrossProd](https://x.com/VectorCrossProd/status/2105124854169227592)) ran the same 291 MMLU-Pro questions against Strata's IQ3_S and two larger Unsloth quantizations:

```
IQ3_S on Strata, 84 GB:     65.3%
UD-Q4_K_XL, 111 GB:         68.4%
UD-Q6_K_XL, 169 GB:        68.7%
```

He added in [the next post](https://x.com/VectorCrossProd/status/2105124856941650123):

> The 3.5-bit does cost ~3 pts, and it's real (p=0.02 vs Q6).

Going down to 3.5 bits costs about 3 points, and it holds up statistically. IQ3_S is already Strata's highest-precision option; IQ3_XXS compresses harder, so common sense says it loses more — but nobody has measured it.

For me the choice was simple: **rather than stretching context to the max, I care more about making fewer mistakes and redoing less work every day.**

---

## Isn't 27B Good Enough?

On the official table, Flash-Next is clearly stronger than 27B; the biggest gap, DeepSWE 1.1, is 16.5 points. But on my 5090, two things have to come off that:

- **The quantization discount**: the official scores are for the original model, and I'm running 3-bit. In the MMLU-Pro example above, IQ3_S lost about 3 points.
- **One request at a time**: Strata's README is explicit that it answers one request at a time. 27B on SGLang serves four people at once, with 425.44 tok/s total throughput.

So if the question is "set up a shared coding server for a small team," the answer is still 27B dense. It lives entirely in VRAM, uses no RAM for weights, and runs requests in parallel.

Where Strata + Flash-Next fits is **a single-user personal agent machine**: one person, one agent, wanting a stronger model, and when needed, dropping an entire repo into context in one go.

---

## The Bottleneck Moved from the GPU to Memory

Strata pushed down the VRAM requirement, and the pressure moved to DRAM.

Planning models for a 5090 used to start with "what fits in 32GB of VRAM," and the answer was a 27B dense model. Now there's another path, but the first question is "is there enough RAM for all the experts":

- **RAM decides whether it runs at all**: IQ3_XXS needs 47.0GB and IQ3_S needs 54.8GB for the experts
- **VRAM decides how fast**: the more hot experts it can cache, the fewer times it calls on the CPU

And RAM happens to be the thing whose price climbed the most this past half year. 64GB is "just enough" for this model, not "comfortable": IQ3_XXS at 1M leaves 5.8 GiB, and IQ3_S's extra 7.8GB means giving up 1M.

---

## The Power of Open Source

Looking back at the whole thing, not a single piece came from the same company:

- Qwen released the 125B weights
- ISTA-DASLab compressed it to 3 bits
- One person spent a week writing Strata, standing on llama.cpp's shoulders, MIT-licensed
- On X, someone ran 291 MMLU-Pro questions on their own to check quality for everyone, and someone else casually [sent a PR](https://x.com/khu/status/2105391806020284722) so older GPUs can run it too
- And then I plugged it into the agent I use every day, on a single 5090

Cloud models are cheap, sure, but the vendors eventually need to IPO — Codex and Claude will still go up in price.

Meanwhile the 5090 I bought half a year ago hasn't changed. Its price has, though: from NT$100,000 to NT$120,000 to NT$170,000, and now NT$220,000. The models went from Qwen 3.6 27B to Qwen 3.8 27B to Qwen 3.8 Flash Next, and the [Artificial Analysis Intelligence Index](https://artificialanalysis.ai/models/qwen3-8-flash-next) went from 21 to 34 to 40.

Same machine. The intelligence keeps going up.

---

## Reality Check

The numbers in this post have some obvious limits.

**First, the benchmarks are Qwen's own table.** I didn't run a quality benchmark myself — only "it runs and isn't broken" tests: arithmetic, tool calls, reading an image, finding verification codes. On 9/28 I had it write a merge_intervals function; it passed 5 edge cases but left out the test examples I explicitly asked for.

**Second, IQ3_S making fewer mistakes is a usage observation, not a controlled experiment.** The two periods had different context limits, KV precision and tasks. The real-request speeds of the two versions aren't like-for-like either: the IQ3_XXS side was running at twice the context length.

**Third, 1M is a stretch.** The model was trained natively to 262,144 positions; 1M is a 4x YaRN extension. With KV at q4_0, Strata's own tests show document perplexity up 8% at 1K and 12% at 8K. Getting all three codes right only shows it can *find* an obvious sentence, not that it *understands* the whole document.

**Fourth, Strata is one person's week-old project.** Thirty versions in a week means fast iteration, and also that tomorrow's version may change today's behavior. I kept the previous engine and config as a backup at every step.

But it got one thing right: **it proves that "a 32GB GPU + 64GB RAM running a 125B MoE" isn't a demo — it's a service you can leave running every day.** Since 9/28 it has been the default model for OpenClaw on my Mac mini.

---

## Key Takeaways

**In the MoE era, the local bottleneck moves from VRAM to DRAM.** When planning a machine, first ask "is there enough RAM for every expert," then ask "how many hot experts can VRAM cache."

**For the same model, choosing a quantization is choosing how to spend RAM.** On 64GB, IQ3_XXS buys you 1M context and IQ3_S buys you precision. For day-to-day agent work, fewer mistakes are worth more than longer context.

**Pick the model by the use case.** For a shared server, pick 27B dense; a 125B MoE on Strata belongs on a single-user personal agent machine that wants a stronger model.

> Cloud intelligence is rented, and the rent goes up. A local machine is bought, and its intelligence goes up.

---

## Appendix: Test Data

All numbers are from the same machine: Core Ultra 7 265K, 64GB DDR5-4800, RTX 5090 32GB, CUDA 12.9. All speeds are single measurements.

| Date | Version and setup | Numbers |
|------|-----------|------|
| 9/28 | 0.1.14, IQ3_XXS, 32K, int8 KV | Short requests 87.7–118.8 tok/s, first token 0.31–0.45 s; 768-token long output 129.9 tok/s, same prompt rerun 146.9; VRAM at 31,693 MiB with only 416 MiB free |
| 9/28 | Same, with Bonsai on the same GPU | Same long-output task: 97.4 tok/s |
| 9/30 | Sharing the GPU with CosyVoice | Strata gave up 4GB of VRAM; expert cache slots dropped from 13,602 to 9,084; generation 61.5 tok/s |
| 9/30 evening | 0.1.29, IQ3_XXS, 262K, int8 KV | 384-token generation 94.4 tok/s (previous version, same setup: 80.6); GPU expert cache hit rate 93.4%; 11,340 cache slots |
| 10/1 early morning | 0.1.30, IQ3_XXS, 1M, q4_0 KV | 512-token generation 101.9 tok/s; official serve tests 51/51; 300K-token needle test all correct in 76 s; 999K-token test all correct in 344 s with 5.8 GiB of RAM left |
| 10/1 early morning | Same, real requests | Actual context about 185K–188K; prefill 3,278–3,642 tok/s; long-output generation 87.2–89.6 tok/s |
| 10/1 dawn onward | 0.1.30, IQ3_S, 262K, int8 KV | 32.81 GiB of experts resident in RAM, 9,494 cache slots; 41 real requests with a median generation speed of 106.8 tok/s; prefill 3,160–4,250 tok/s; expert cache hit rate 67.8%–96.5% |

---

## FAQ

**Q: What is Strata, and how is it different from llama.cpp?**

Strata is an inference engine open-sourced (MIT) by GitHub user Niko1221 on September 24, 2026. It is built to run Qwen3.8-Flash-Next, a 125B Mixture of Experts (MoE) model, on a single 12–24GB NVIDIA gaming GPU plus 64GB of RAM. It reuses parts of llama.cpp / ggml, but works differently: llama.cpp's MoE offload keeps experts in CPU RAM and moves them to the GPU when needed, while Strata caches the most-used experts in VRAM, has the CPU compute misses directly in RAM, and leaves the 29GB n-gram lookup table on SSD. On an RTX 5090, the measured GPU expert cache hit rate was 93.4%.

**Q: What's the difference between IQ3_XXS and IQ3_S, and which should I pick?**

Both are 3-bit quantizations from the llama.cpp family: IQ3_XXS is about 3.06 bits per weight and IQ3_S about 3.44. For Qwen3.8-Flash-Next, IQ3_XXS needs 47.0GB of RAM+VRAM and IQ3_S needs 54.8GB. On an RTX 5090 with 64GB of RAM, IQ3_XXS can stretch context to 1M, while IQ3_S's extra 7.8GB limits it to 262K. A third-party run of 291 MMLU-Pro questions scored Strata's IQ3_S at 65.3%, about 3 points below Unsloth's UD-Q6_K_XL (68.7%); IQ3_XXS compresses further. For day-to-day agent work, the author saw fewer errors with IQ3_S, but that is a usage observation, not a controlled experiment.

**Q: How fast does Qwen3.8-Flash-Next actually run on an RTX 5090?**

On a Core Ultra 7 265K, 64GB DDR5-4800 and RTX 5090 32GB machine with Strata 0.1.30: IQ3_XXS generated 512 tokens at 101.9 tok/s; IQ3_S (262K context, int8 KV) served 41 real agent requests at a median generation speed of 106.8 tok/s, with prefill around 3,000–4,000 tokens per second. These were real requests of about 64K tokens, not a benchmark at a full 262K. If other GPU services share the card, Strata can cache fewer experts and slows down — for example to 61.5 tok/s when sharing with CosyVoice.

**Q: Can Qwen3.8-Flash-Next really run 1M context on a 5090?**

It runs, but it's experimental. The model's native training length is 262,144 tokens; reaching 1M requires a 4x YaRN rope scaling extension and dropping the KV cache from int8 to 4-bit (q4_0). The test was a needle-in-a-haystack run: a 999,490-token document with a verification code hidden at the 10%, 50% and 90% positions. All three were found, but reading took 344 seconds and only 5.8 GiB of RAM was left. This kind of test shows the model can find a needle, not that long-document understanding is unchanged — and only IQ3_XXS fits; IQ3_S doesn't fit in 64GB of RAM at 1M.

**Q: Why can't FreeToken run it while Strata can?**

FreeToken is an MoE serving system from a UC Berkeley-led team, published in August 2026. It keeps every expert in host RAM; the paper's RTX 5090 test machine has 192GB of DDR5. It supports Qwen3.8-Flash-Next in FP8 or NVFP4, but the NVFP4 checkpoint is still 135GB, and the n-gram lookup table needs 47.7 GiB pinned in RAM. An RTX 5090's 32GB of VRAM plus 64GB of RAM is 96GB in total, which isn't enough. Strata uses roughly 3-bit IQ3 quantization and reads the 29GB n-gram table from SSD on demand, so 64GB of RAM is enough.

---

## Further Reading

- [MoE Offload, Fully Explained (Chinese)](https://ai-coding.wiselychen.com/moe-offload-deepseek-v3-v4-local-inference-optimization/)
- [The 5090 Went from NT$100K to NT$170K in Three Months: Five Paths to a Lower Local-AI Hardware Bar (Chinese)](https://ai-coding.wiselychen.com/memory-price-surge-local-ai-five-paths/)
- [Two Days with Qwen3.8-27B: RTX 5090 SGLang/vLLM/llama.cpp Benchmarks](/en/youtube-qwen38-27b-rtx5090-sglang-vllm-mtp-transcript/)
- [Qwen3.8-27B Goes Open: SWE-bench Pro 61.7 Beats Opus 4.6 Max](/en/qwen-3-8-27b-open-weights-local-security/)
- [Can One Machine Run Tier 1 Local Models? RTX Pro 6000 One-Week Experiment, Day 1–2 (Chinese)](https://ai-coding.wiselychen.com/rtx-pro-6000-tier1-local-day1-2-glm52-deepseek-v4-flash/)

## Sources

- [Strata (GitHub, Niko1221)](https://github.com/Niko1221/Strata)
- [Qwen/Qwen3.8-Flash-Next model card (Hugging Face)](https://huggingface.co/Qwen/Qwen3.8-Flash-Next)
- [FreeToken: Efficient Edge-Native MoE Serving with Bandwidth-Adaptive Execution (arXiv 2608.16157)](https://arxiv.org/html/2608.16157v1)
- [FreeToken supported models](https://github.com/FlashML-org/FreeToken/blob/main/docs/models.md)
- [RadixArk/Qwen3.8-Flash-Next-NVFP4](https://huggingface.co/RadixArk/Qwen3.8-Flash-Next-NVFP4)
- [Artificial Analysis: Qwen3.8-Flash-Next](https://artificialanalysis.ai/models/qwen3-8-flash-next), [Qwen3.8 27B](https://artificialanalysis.ai/models/qwen3-8-27b), [Qwen3.6 27B](https://artificialanalysis.ai/models/qwen3-6-27b)
- [Nigel Hungerford-Symes (@VectorCrossProd): 291-question MMLU-Pro test of Strata IQ3_S](https://x.com/VectorCrossProd/status/2105124854169227592)
- [llama-quantize quantization types (IQ3_XXS / IQ3_S bits per weight)](https://manpages.debian.org/unstable/llama.cpp-tools/llama-quantize.1.en.html)
