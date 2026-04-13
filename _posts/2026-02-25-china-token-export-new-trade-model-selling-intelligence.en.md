---
layout: post
title: "Token Export: China's AI Is No Longer Selling Products—It's Selling Tokens"
date: 2026-02-25 09:00:00 +0800
permalink: /en/china-token-export-new-trade-model-selling-intelligence/
image: /assets/images/china-token-export-electricity-cover.jpg
description: "China's AI export is undergoing a qualitative shift—from selling products to selling Tokens. In February 2026, Chinese models (MiniMax, Kimi, GLM) overtook the US in production-grade Token call volume for the first time. GLM-5 walked away clean from distillation accusations, beat GPT-5.2 on SWE-bench, was trained entirely on Huawei chips, and its API is 5-8x cheaper. Stack together China's electricity price ($0.08/kWh vs US $0.18/kWh), open-source talent density, and hardware autonomy, and you see a new trade paradigm forming: exporting SOTA-90% reasoning capability to the world in a metered, priceable way. This isn't a story about the tech race. It's a story about cost structure."
lang: en
---

> 📝 Originally written in Chinese. [Read the original →](https://ai-coding.wiselychen.com/china-token-export-new-trade-model-selling-intelligence/)

Last week I wrote two articles—one was [a deep dive on the GLM-5 technical report](/glm-5-vibe-coding-to-agentic-engineering-technical-report/), and the other was [the Anthropic distillation war](/anthropic-distillation-war-everyone-steals-from-everyone/).

After writing them, one thing kept nagging at me: **if GLM-5 really wasn't distilled, and if it was trained entirely on Huawei chips, and if its API is priced at one-fifth of Opus—what does that actually mean?**

It's not a story about a tech breakthrough. I already wrote that story.

This is a story about a **business model**.

---

## From "Selling Products" to "Selling Tokens"

For the past thirty years, China's export playbook has been products—from T-shirts and electronic components to TikTok, SHEIN, Temu. Physical goods, software, user experience, traffic.

In 2026, what's being exported has changed. It's no longer products. It's **Tokens**.

What does that mean?

A Cnyes report in February gave a stunning number: **February 2026 Token call volume is 13x what it was in February 2025. The top three in call volume are all Chinese models—MiniMax, Kimi, GLM. For the first time, China has surpassed the US in production-grade Token call volume.**

This isn't lab data. It's real money API calls. People are paying for these tokens, and the volume has exceeded US models.

I think the significance of this is being underestimated.

In the past, when we talked about the AI race, the focus was always on Benchmarks: whose SWE-bench is higher, whose reasoning is stronger. But Benchmarks are technical indicators, not commercial ones. The commercial indicator is: **whose tokens more people are buying.**

And right now, the answer is China.

---

## Cost Structure: Why China Can Be This Much Cheaper

Let's look at pricing first. API pricing comparison as of February 2026:

| Model | Tier | Input $/M tokens | Output $/M tokens |
|-------|------|------------------|-------------------|
| Claude Opus 4.5 | Frontier | $5.00 | $25.00 |
| GPT-5 | Frontier | $1.25 | $10.00 |
| DeepSeek R1 | Reasoning | $0.55 | $2.19 |
| GLM-5 | Frontier Open-source | ~$1.00 | ~$3.20 |
| DeepSeek V3.2 | General | $0.27 | $1.10 |

Compared to Opus 4.5, GLM-5 is **5x cheaper on input, 8x cheaper on output**. It's also significantly cheaper than GPT-5.

But the point isn't the word "cheap." The point is: **why can it be cheap?**

Token production cost can be broken down into this formula:

> **Token Price ≈ Hardware Depreciation + (Energy × Electricity Price × PUE) + Engineering Ops + Compliance Cost**

China has a structural advantage in **every single one** of these.

---

### Card #1: Electricity

An analysis from The Diplomat last year pointed out that China's industrial electricity price is around **$0.08/kWh**, versus around **$0.18/kWh** in the US. China is **56% cheaper**.

This isn't a one-off exchange rate advantage—it's structural. The Chinese government subsidizes both fossil fuels and green energy at a scale three times that of the US. A Fortune headline from last August was even more blunt: "AI experts came back from China shocked: the US power grid is too weak, the race may already be over."

For AI inference, electricity cost is ongoing. Training burns electricity once, but inference burns it on every single token. What Token Export is selling is reasoning capability, and the electricity price advantage will **compound over time**.

Rough estimate: if electricity accounts for 30-40% of token production cost, the price differential alone yields a **15-20% cost advantage**.

---

### Card #2: Open-Source Talent + Engineering Efficiency

DeepSeek trained R1 on the restricted H800 (not H100), and achieved **10-40x** the energy efficiency of US models. This isn't a joke—for the same reasoning capability, the Chinese team did it with less compute.

Why? I think it has to do with the talent structure.

The density and activity level of China's open-source community has exploded over the past two years. Zhipu's Slime RL framework, DeepSeek's MoE architecture optimizations, Kimi's Agent Swarm—these weren't copied (at least Zhipu isn't on the distillation accusation list), they were written line by line by engineers.

And the salary structure for Chinese AI engineers is different from Silicon Valley. I'm not saying cheaper equals better, but for the same labor cost investment, you get more engineering optimization output.

---

### Card #3: Hardware Autonomy

This is the most underestimated signal from GLM-5.

GLM-5's 744B parameters were trained entirely on **100,000 Huawei Ascend 910B chips**. Zero NVIDIA dependency. Using the Huawei MindSpore framework.

What does this mean?

The US export controls (NVIDIA H100/H200 banned from China) were supposed to choke China's AI compute. But GLM-5 proved: with domestic chips, you can train a frontier-tier model.

And Huawei has already published a three-year AI chip roadmap. Ascend 910C and 920 are both on the schedule. If chip performance keeps catching up, and stays unaffected by sanctions—China's hardware costs will drop further.

Stack the three cards together: **electricity 56% cheaper + higher engineering efficiency + hardware autonomy immune to export bans**.

The conclusion: China can produce **SOTA-90%-tier** tokens at roughly **80% of the cost**.

---

## Why GLM-5 Is the Bellwether Case

In my previous articles I analyzed GLM-5 from a technical angle. Here, let's switch to the commercial angle.

GLM-5 simultaneously satisfies every condition for Token Export:

**1. Technical autonomy is viable—no distillation accusations**

Anthropic's distillation accusation list includes DeepSeek, Moonshot AI (Kimi), and MiniMax—but **not Zhipu**.

In that article I wrote: "GLM isn't on the list. Maybe that means they really are doing their own thing."

This isn't just a moral issue, it's a commercial one. If your model is accused of being distilled, overseas clients will worry about legal risk. But if you can prove technical autonomy—open-source code, public tech reports, MIT license—client trust cost drops dramatically.

**2. Quality is good enough—beats GPT-5.2 on SWE-bench**

GLM-5 scored 77.8% on SWE-bench Verified, beating GPT-5.2's 76.2%, only 3 points behind Opus 4.6.

For most enterprise clients, the gap between SOTA and SOTA-3% is barely perceptible. But a 5-8x price difference is very perceptible.

**3. Fully Huawei-trained—hardware not at anyone's mercy**

This means GLM-5's inference infrastructure can run entirely on domestic hardware. No worries about NVIDIA cutting supply, no need to go through gray markets to buy chips. Supply chain is stable, cost is predictable.

**4. MIT open-source—both paths viable**

The MIT license means overseas developers can download, fine-tune, and deploy privately. This is the Stack-as-a-Standard path—not just selling tokens, but exporting technical standards.

At the same time, Zhipu also offers API services—that's the Token-as-a-Service path.

Playing both hands builds the ecosystem.

---

## Two Export Paths

The Cnyes report breaks Token Export into two paths, and I think the analysis is spot on:

### Path A: Token-as-a-Service (Inference-as-a-Service)

Overseas users call Chinese models directly via API. Tokens are "produced" either inside China or at Chinese-owned overseas nodes.

**Advantages:**
- Lowest technical barrier (just call an API)
- Marginal cost drops with scale
- China's electricity advantage directly materializes

**Challenges:**
- Latency (cross-border networking)
- Compliance (data sovereignty)
- Cross-border payments
- Geopolitical risk (what if sanctions hit?)

### Path B: Stack-as-a-Standard (Standards-as-Ecosystem)

Overseas developers download Chinese open-source models and fine-tune/deploy on their own infrastructure.

**Advantages:**
- No latency issues
- No data sovereignty issues
- Builds ecosystem and technical standards

**Challenges:**
- Can't directly profit from electricity advantage
- Requires long-term community cultivation
- Commercializing open-source models is hard

2025 data shows that Chinese OSS models' weekly token share has already approached **30%** during some time windows.

I think the truly smart players (like Zhipu) will take both paths. API for short-term revenue, open-source to build the long-term ecosystem. Once the ecosystem is built, switching costs become too high even if others want to switch away.

---

## What Does This Look Like?—The AI Remake of Manufacturing Export

If this pattern sounds familiar, that's because it's a remake of China's manufacturing export playbook.

| Dimension | Traditional Manufacturing Export | Token Export |
|-----------|----------------------------------|--------------|
| What's sold | Physical goods | Reasoning capability (Tokens) |
| Source of cost advantage | Labor + supply chain | Electricity + hardware + engineering efficiency |
| Quality positioning | "Good enough" → gradual upgrade | SOTA 90% → gradual catch-up |
| Export path | OEM → own brand | API service → open-source ecosystem |
| Moat | Scale effects | Scale effects + open-source lock-in |

Thirty years ago, China was selling T-shirts and electronic components.
Ten years ago, it was phones and apps.
Now, it's Tokens—metered intelligence.

And unlike physical goods, Tokens have no logistics cost, no tariffs (for now), no inventory. Once the model is trained, marginal cost is just inference electricity and hardware depreciation.

This is a business with extremely high potential gross margins.

---

## Honestly: Risks and Uncertainties

As usual, after the upside, the risks.

**1. Geopolitics is the biggest variable**

If the US imposes sanctions on Chinese AI APIs (similar to the TikTok logic), the Token-as-a-Service path gets cut off directly. The Stack-as-a-Standard path is relatively safer, but could also face "security reviews."

**2. The quality gap could widen or close**

The SOTA-3% gap looks small right now. But if Anthropic or OpenAI achieves a breakthrough in their next-gen model (like Opus 5 or GPT-6), the gap could reopen. That said, open-source community iteration speed is also accelerating.

**3. Compliance cost is being underestimated**

GDPR, data localization, AI Act—compliance costs in European markets aren't low. Chinese companies have relatively thin experience here. If compliance isn't done well, no amount of cheap tokens will get you into high-value markets.

**4. Zhipu not being on the distillation list ≠ never being questioned**

Not being accused doesn't mean innocent—it just means no evidence right now. If new accusations surface in the future, trust cost spikes instantly.

---

## Key Insights

**1. Token Export is a cost-structure story, not a tech-race story**

Chinese models don't need to win Benchmarks. As long as they reach SOTA 90% and are 5-8x cheaper, the market will buy. This is exactly the same logic as Chinese manufacturing—quality good enough, price cut to the bone.

**2. Electricity is an overlooked strategic resource**

Everyone talks about compute (chips), but electricity is the ongoing cost of the inference economy. China's $0.08 vs US's $0.18—that gap isn't disappearing anytime soon.

**3. GLM-5's full-Huawei training is a milestone**

It proves Chinese AI can completely decouple from the US hardware ecosystem. This isn't just a technical achievement, it's a commercial moat—supply chain unaffected by sanctions, cost predictable, capacity plannable.

**4. Open-source + API dual-track is the smartest strategy**

Short term, make money with API. Long term, build ecosystem with open-source. Zhipu's MIT license isn't charity, it's strategy.

**5. Implications for Taiwanese companies**

If you're building AI applications, the price-performance ratio of Chinese tokens can no longer be ignored. I'm not saying switch everything over, but at minimum you should evaluate: does your use case really need Opus-tier models? Or is SOTA 90% good enough?

**The 80% cost savings can fund a lot more things.**
