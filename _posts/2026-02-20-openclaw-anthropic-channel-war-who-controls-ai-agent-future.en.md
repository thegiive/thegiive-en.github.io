---
layout: post
title: "The Channel War: OpenClaw, Anthropic, and Who Gets to Decide the Future of AI Agents"
date: 2026-02-20 10:00:00 +0800
permalink: /en/openclaw-anthropic-channel-war/
description: "OpenClaw v2.19 shipped an Apple Watch MVP. Anthropic blocked OAuth to shut out third-party subscribers. Sam Altman recruited Peter Steinberger and embraced open source. Put these three things together and you see something beyond a technical competition — you see the most brutal reality of the AI industry: whoever controls the channel decides the model's fate."
image: /assets/images/anthropic-oauth-ban-official-docs.png
lang: en
---

## Opening: Talking to a Lobster from Your Wrist

OpenClaw v2.19 just shipped an Apple Watch MVP.

That's right — you can now issue commands to your AI agent from your watch. Watch Inbox UI, push notifications, wrist-level commands. Sounds like science fiction, but it's in this week's release notes.

My first reaction: this team's ambition is insane.

But on reflection, it's not really about ambition. This is a team with an extremely clear product vision, answering a very concrete question:

**"Where should the AI agent interface live?"**

Not in an IDE. Not in a terminal. Not in a browser. The answer is: "wherever you are."

Apple Watch, Telegram Streaming (character-by-character push, inline buttons, user reactions triggering agent events), 40+ security hardening items, OTEL v2 migration — these aren't flashy feature stacking. This is a project seriously building infrastructure.

Meanwhile, the same week, what was Anthropic doing?

Blocking OAuth.

---

## Channel Is Everything in AI

I want to say something a lot of people don't want to hear: **in the AI industry, the strongest model is worthless without a channel.**

Let me walk through the history.

### 2024: Cursor Carried Anthropic

In 2024, AI coding became genuinely viable. What drove it?

Cursor + Sonnet 3.5.

Cursor's rise drove the entire AI coding ecosystem, and Sonnet 3.5 happened to be the best model for coding at the time. But think carefully — what role did Anthropic actually play here?

**They were the chosen supplier.**

The channel was Cursor's. User loyalty was Cursor's. Pricing power, use case definition, user experience — all Cursor's. Anthropic just provided the engine in the back.

And look at the numbers: in early 2025, no matter how little ChatGPT prioritized coding, GitHub Copilot alone was still the largest AI coding user base. Not because Copilot's model was best — because it was embedded in VS Code. **Channel determines everything.**

### 2025: Claude Code Is Born

Then Claude Code appeared.

This was the first time Anthropic — beyond its B2B API business — actually owned a channel.

Claude Code gave Anthropic direct access to developers. No Cursor middleman, no Copilot middleman, no third party. Developer opens a terminal, enters Claude's world.

I wrote previously that [Claude Code uses the right interface to connect to the real world](/shell-wrapper-2-anthropic-real-threat/) — the command line turns an entire computer into a queryable, composable, reasonably-stateful space, and LLMs think in text. That's not a coincidence; it's a structural fit.

Claude Code's success meant Anthropic finally wasn't just a "model supplier selected by others."

**But that advantage lasted less than a year.**

---

## What OpenClaw Actually Disrupts

Honestly: since OpenClaw appeared in January this year, my Claude Code usage has been slowly declining.

Not because Claude Code isn't good. It's still the most precise coding agent I've seen. But OpenClaw demonstrated an entirely different use case — something Claude Code deliberately doesn't do:

**A genuine AI agent.**

Not a "help you write code" agent. A "help you live your life" agent.

Real-time streaming replies on Telegram. Automatic Google Calendar scheduling. Automatic YouTube video processing. And now Apple Watch commands.

Claude Code is an extremely disciplined product — it does only coding, and only inside a terminal. That discipline is technically correct. But in product terms, it left an enormous gap.

OpenClaw is built directly for that gap.

And it's not just a feature gap. OpenClaw represents a fundamentally different product philosophy:

- **Claude Code says:** I'm the smartest coding agent. Come find me in the terminal.
- **OpenClaw says:** I'm on your phone, your watch, your Telegram, your browser. Where you are, I am.

These two philosophies address completely different scale markets.

People who use terminals: maybe tens of millions worldwide. People who use watches and Telegram: billions.

---

## Anthropic's Chain of Missteps

If it were just a difference in market positioning, fine. What made me think "the table is getting flipped" was a series of moves Anthropic made during this period.

### Move One: The Legal Letter (2026-01-27)

Anthropic's legal team sent Peter Steinberger a cease-and-desist. "Clawd" too close to "Claude." Trademark concerns.

The result?

Peter renamed to Moltbot (lobster molting — actually a nice metaphor). But during the rename, a 10-second window let crypto scammers grab the old namespace. A fake $CLAWD token appeared and hit a $16 million market cap.

Then came another rename: OpenClaw.

Then Sam Altman made his move.

Multiple outlets ran headlines like: "$30B Fumble: Anthropic Kills 1.5M-Agent Beast — OpenAI Poaches Creator in Seconds!"

Using a legal letter to drive the hottest agent project straight into a competitor's arms. Hard to interpret that as anything but a strategic own goal.

### Move Two: OAuth Block (executed 2026-01-09)

![Anthropic official documentation explicitly banning OAuth for third-party products](/assets/images/anthropic-oauth-ban-official-docs.png)

Anthropic deployed server-side protections blocking subscription OAuth tokens from being used in third-party products.

The error message was blunt:

> "This credential is only authorized for use with Claude Code and cannot be used for other API requests."

The official terms were explicit:

> "Using OAuth tokens obtained through Claude Free, Pro, or Max accounts in any other product, tool, or service — including the Agent SDK — is not permitted and constitutes a violation of the Consumer Terms of Service."

Rails creator DHH's reaction: "very customer hostile."

Previously, many teams ran Claude Max subscriptions ($100–$200/month) with OpenClaw, OpenCode, Roo Code, and similar tools to avoid pay-per-token API costs. Anthropic cut that off completely.

It also emerged that OpenCode (107K+ stars on GitHub) had been spoofing Claude Code's HTTP headers to bypass the restriction.

### The Contrast: Sam Altman's Embrace Strategy

On February 15, 2026, Sam Altman posted on X:

> "Peter Steinberger is joining OpenAI to drive the next generation of personal agents. He is a genius with a lot of amazing ideas about the future of very smart agents interacting with each other to do very useful things for people."

Simultaneously committed that OpenClaw would remain open-source under a foundation structure, hinting at a potential OpenAI version as part of ChatGPT subscriptions.

One side sending legal letters to drive someone away. The other side sending an offer to pull them in.

**The contrast is brutal.**

---

## The Great Model Reshuffle

The OpenClaw ecosystem's explosion triggered another phenomenon: a massive reshuffling of model choices.

### Opus 4.6: Smartest, But Too Expensive

Opus 4.6 remains the community's consensus pick for OpenClaw when intelligence is the priority. Complex reasoning, long context understanding, coding quality — it's the best available right now.

But the economics are brutal: **$25 per million output tokens.**

Heavy users burning $50–$100 a day is normal. That's not sustainable for most individuals or small teams.

So everyone is being forced to find alternatives.

### Kimi K2.5: Free + Capable

Moonshot AI's Kimi K2.5 outperforms Opus 4.5 on agentic benchmarks, with strong SWE-bench scores. API pricing: $3 per million output tokens — **one-eighth of Opus**.

Peter had previously recommended Kimi K2 and K2.1 Turbo. With Moonshot launching Kimi Claw (an OpenClaw-based browser-native version), Kimi K2.5 has become an increasingly visible presence in the OpenClaw ecosystem.

And right now it has a free tier. For individuals, the barrier to entry is almost zero.

### Gemini 3 Flash Preview: Speed Wins

Google's Gemini 3 Flash Preview just launched, clocking 218 tokens/second and SWE-bench 78%.

The key: less than a quarter the cost of Gemini 3 Pro, and Google's own claim: "Outperforms 2.5 Pro while being 3x faster."

For agent use cases that require fast response (Telegram streaming, Apple Watch interaction), speed is the experience. On that dimension, Gemini 3 Flash dominates.

### MiniMax M2.5: On-Premise Value Champion

MiniMax M2.5 is an open-source 230B MoE model (10B active parameters), SWE-bench 80.2% — just 0.6 percentage points behind Opus 4.6's 80.8%.

Key number: 101GB after 3-bit quantization. Runs on a Mac with 128GB unified memory.

MiniMax's own claim: "$1 keeps it running for an hour at 100 tokens/second." The first frontier model where cost is essentially not a concern.

For users who care about data privacy and want to run locally, MiniMax M2.5 is probably the most practical option available today.

### The Subscription Wildcard

If Sam Altman actually integrates OpenClaw into ChatGPT subscriptions — even partial functionality — the subscriber base will explode overnight.

Think about it: you're already paying for ChatGPT Plus. Now someone tells you that includes running an AI agent with no additional API fees. The channel effect would be enormous.

---

## Honestly

My judgment might be biased.

I haven't been using OpenClaw for long, and I'm still figuring out a lot of its features. I don't know Anthropic's internal decision-making logic — maybe the OAuth block has financial pressures or compliance considerations I'm not aware of.

But some things I feel more confident about:

**What I'm fairly sure of:**

1. **The importance of channel has been proven over and over.** Cursor carried Anthropic, Claude Code let Anthropic reclaim the channel, now OpenClaw is redistributing it again. History is repeating.

2. **Defensive strategies are poison in open-source ecosystems.** Legal letters and OAuth blocks protect short-term subscription revenue, but in a community-driven ecosystem, these moves just accelerate community flight. This looks a lot like Oracle suing Google over Java APIs — won the lawsuit, lost the ecosystem.

3. **Model moats are disappearing.** When MiniMax M2.5's open-source model trails Opus 4.6 by just 0.6% on SWE-bench, when Kimi K2.5 delivers similar quality at one-eighth the price — the model itself is increasingly hard to use as a differentiator. What determines user choice is the experience layer around the model: persistence, cross-device, security, cost control. That's exactly what OpenClaw is building.

4. **The biggest insight: Anthropic was carried to relevance by Cursor, then used Claude Code to escape channel dependency. But OpenClaw proved that coding is just one subset of agents — the real agent market is ten times bigger than coding.** If Anthropic can't build its own channel in the broader agent space, Claude Code's success may be a beautiful detour.

**What I'm not sure about:**

1. **How will OpenAI integrate OpenClaw?** Keep it independent and fully open-source, or gradually absorb it as a ChatGPT feature? Peter's role post-joining OpenAI isn't clear yet.

2. **Will Anthropic adjust strategy?** Maybe they'll launch a more open third-party integration program, or respond with new Claude Code features. Nothing visible yet, but that doesn't mean it won't happen.

3. **Where's the quality ceiling for free models?** Kimi K2.5 is free, MiniMax M2.5 is essentially free — but "almost as good" and "actually as good" are two different things in production. For mission-critical, high-reliability scenarios, Opus 4.6's premium may still be worth it.

---

## Key Takeaways

1. **AI industry competition is fundamentally a channel war, not a model war.** Whoever controls the user's entry point controls the model's fate. Cursor, Copilot, Claude Code, OpenClaw — every channel shift reshuffles the ecosystem.

2. **Defensive strategies are poison in open-source ecosystems.** Legal letters and OAuth blocks protect short-term interests, but in community-driven ecosystems they accelerate user flight. Google's Android strategy (embrace open source, monetize through services) has been far more effective than Oracle's Java strategy.

3. **Models are commoditizing; agent infrastructure is the new differentiator.** When multiple models hit 78–80% on SWE-bench, what determines user choice is no longer the model — it's the experience built around it: persistence, cross-device, security, cost control. That's exactly what OpenClaw is building.

4. **The core insight: Anthropic was once carried to relevance by Cursor, then used Claude Code to escape channel dependence. But OpenClaw proved coding is just a subset of agents — the real agent market is ten times larger.** If Anthropic can't establish its own channel in the broader agent space, Claude Code's success may be nothing more than a beautiful episode.

---

## Further Reading

- [From "Shell Wrapper 1.0" to "Shell Wrapper 2.0": Why Anthropic Should Be Worried](/shell-wrapper-2-anthropic-real-threat/)
- [OpenClaw Architecture Deep Dive: Context, Memory, and the Token Crusher Design Philosophy](/openclaw-architecture-deep-dive/)
- [OpenClaw Cost Optimization: A 97% Reduction from $200/Day to $6/Day](/openclaw-cost-optimization-guide/)
- [OpenClaw Security Isolation: Gmail Sandbox and the Principle of Least Privilege](/openclaw-security-isolation-gmail-sandbox/)
- [Peter Steinberger's Agentic Engineering Philosophy: Just Talk to It](/peter-steinberger-agentic-engineering-philosophy/)
