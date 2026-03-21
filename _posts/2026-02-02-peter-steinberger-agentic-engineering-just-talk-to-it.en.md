---
layout: post
title: "From $116M Exit to 100K GitHub Stars in 10 Days: Peter Steinberger's Agentic Engineering Philosophy"
date: 2026-02-02 10:00:00 +0800
permalink: /en/peter-steinberger-agentic-engineering-just-talk-to-it/
image: /assets/images/peter-steinberger-cover.png
description: "\"Don't waste your time on stuff like RAG, subagents, Agents 2.0 or other things that are mostly just charade. Just talk to it.\" — Peter Steinberger. The story of a retired engineer who sold his company for $116M, then hacked a 100K-star GitHub project in an hour."
lang: en
---

> "Don't waste your time on stuff like RAG, subagents, Agents 2.0 or other things that are mostly just charade. Just talk to it." — Peter Steinberger

**Author:** Wisely Chen
**Date:** February 2026
**Series:** AI Agent Field Notes
**Keywords:** OpenClaw, Moltbot, Clawdbot, Peter Steinberger, Agentic Engineering, AI Agent, Unix Philosophy

![From $116M Exit to Agentic Engineering: Peter Steinberger and the OpenClaw Development Philosophy](/assets/images/peter-steinberger-cover.png)

---

## Table of Contents

- [A Retired Engineer's Boredom Problem](#a-retired-engineers-boredom-problem)
- [How Clawdbot Was Born: A One-Hour Hack](#how-clawdbot-was-born-a-one-hour-hack)
- [Agentic Engineering: Just Talk To It](#agentic-engineering-just-talk-to-it)
- [CLI Over Browser: Why He Doesn't Use Web Agents](#cli-over-browser-why-he-doesnt-use-web-agents)
- [Human-in-the-Loop vs. Agent Slop](#human-in-the-loop-vs-agent-slop)
- [A Legend: The Voice Message Incident](#a-legend-the-voice-message-incident)
- [The Rename Storm: Three Names in One Week, Hijacked in 10 Seconds](#the-rename-storm-three-names-in-one-week-hijacked-in-10-seconds)
- [Honestly](#honestly)
- [Further Reading](#further-reading)

---

## A Retired Engineer's Boredom Problem

Peter Steinberger is not your typical person.

He's Austrian, graduated from TU Wien, and taught iOS and Mac development there. He was one of the first iOS developers — started writing iOS apps the moment the iPhone launched.

In 2011, he received a job offer from a San Francisco startup at a WWDC party. But before his visa was approved, his "side project" was already making money — more than the salary from that job.

That side project was PSPDFKit.

The name comes from his initials (**P**eter **S**teinberger) + **PDF** + **Kit** (as in SDK). A one-man operation in Vienna, building a PDF processing SDK.

He tried to juggle the San Francisco job and his own company at the same time — and burned out completely. In April 2012, he went all-in on PSPDFKit and moved back to Vienna.

**For the next 13 years, he worked almost every weekend.**

The company grew from one person to over 70, fully remote and globally distributed. Clients included Dropbox, DocuSign, SAP, IBM, and Volkswagen. The product ran on over a billion devices.

And for those 13 years — **not a single dollar of outside funding. Pure bootstrap.**

In 2021, Insight Partners invested $116M in PSPDFKit. Peter and co-founder Martin Schürrer sold their shares, stepped away from operations, and officially retired.

Then he discovered he had absolutely no idea what to do with himself.

> "When I sold my shares in PSPDFKit, I felt profoundly shattered... I'd poured 200% of my time into that company."

He tried traveling. Tried therapy. Tried various pleasures. None of it worked.

> "You can't find happiness by moving. You must create purpose."

After retirement, he pivoted to web development, learning React and TypeScript. He also became a Venture Partner at Calm/Storm Ventures, investing in startups.

But none of it was what he really wanted to do.

In 2024, AI suddenly went from "not quite there yet" to "genuinely interesting." Peter decided to stop watching from the sidelines. He updated his website with a single line:

> "Came back from retirement to mess with AI."

This is the story of an engineer who sold his company, could have coasted indefinitely, and picked up the keyboard again because he felt like his mojo had disappeared.

---

## How Clawdbot Was Born: A One-Hour Hack

Clawdbot — later renamed Moltbot, then finally OpenClaw — had a very simple origin:

**Peter wanted AI to write code for him when he wasn't sitting at a computer.**

Eating, commuting, traveling — all he had was his phone. But he didn't want to stop working. So he wrote a WhatsApp adapter that sent messages to Claude and let Claude operate his computer directly.

The first version took one hour.

No business plan. No pre-funding MVP. Just a retired engineer's hack.

But when that hack landed on GitHub in late 2025, it hit 9,000 stars in 24 hours. A few days later: 70,000. Two months after that: over 100,000. One of the fastest-growing repos in GitHub history.

---

## Agentic Engineering: Just Talk To It

Peter is not the type to write 20-page architecture documents. His philosophy fits in one sentence:

> "Don't waste your time on stuff like RAG, subagents, Agents 2.0 or other things that are mostly just charade. Just talk to it."

This is the exact opposite of how most "agent frameworks" think.

He noticed a pattern: people spend enormous effort designing subagents, prompt templates, and workflow orchestration — but the actual output is no better than just telling the model what you want directly.

His approach:

1. **No MCP (Model Context Protocol):** He thinks MCP is "something the marketing department uses to check boxes," burning through context tokens at a brutal rate. GitHub's MCP consumes 23,000+ tokens per call, with results that don't justify the cost.
2. **No subagents:** Multi-agent architectures just hide the control flow and make debugging harder. Open multiple terminal windows instead — way more transparent.
3. **No plan mode:** A smart enough model reads the context and plans itself. No need for humans to decompose it.
4. **No complex system prompts:** He tested persona prompts like "You are an AI engineer specializing in production-grade LLM applications." His conclusion: makes no difference whatsoever.

So what does he actually do?

**He runs 3–8 Codex CLI instances simultaneously in a 3×3 terminal grid, manages background tasks with tmux, and has each agent make atomic git commits.**

![The actual toolset: a return to Unix philosophy](/assets/images/peter-steinberger-slides-06.png)

No framework. Just hands on the keyboard.

---

## CLI Over Browser: Why He Doesn't Use Web Agents

Peter is blunt about browser-based agents. He tried Devin, Cursor web, Jules — his verdict:

> "Really annoying to set up and broken."

His alternative? **CLI.**

He invests heavily in writing small command-line tools so agents can get things done through the terminal. The reasoning is practical:

> "Models already know how to use it, and pay zero context tax."

Models already understand Unix commands. Give an agent a CLI tool and it can run `--help` to figure out how to use it. CLI natively supports chaining and reuse — exactly the kind of tool orchestration agents are good at.

Compare that to MCP, which burns through context tokens. He gives the example again: GitHub's MCP costs 23,000+ tokens per call, while `gh` CLI does the same job for nearly zero.

This aligns with what I wrote in my [Unix Philosophy article](/unix-philosophy-command-line-renaissance/) — the command line is the best interface because it turns an entire computer into a queryable, composable, reasonably-stateful space. And LLMs think in text. That's not a coincidence; it's a structural fit.

OpenClaw is designed the same way: memory in local Markdown files, preferences in local text files, all state in human-readable formats.

This lets the agent not just execute, but *remember*.

---

## Human-in-the-Loop vs. Agent Slop

Peter is deeply skeptical of fully autonomous agents.

He calls frameworks that "run for hours with mayors and overseers" — slop. They look impressive, but the output quality is terrible.

> "AI needs human taste and vision to be effective."

His view: AI is a "weird friend," a "squad" that helps you execute ideas faster — but it cannot replace the person who has the ideas in the first place.

His example: he once saw someone report a bug in his project on Twitter. He screenshotted the tweet and sent it to WhatsApp. The agent read the tweet, checked the GitHub repo, fixed the bug, committed, and replied to the Twitter user on his behalf.

**He never sat down at a computer.**

But that doesn't mean he let go entirely. He insists:

> "If you just press go and walk away, the results won't be good. You still need to babysit this thing."

Same thing Geoffrey Huntley — the author of the "let the agent keep running" tool Ralph Wiggum — has said.

**Full automation isn't the goal. Faster, smoother human-machine collaboration is.**

---

## A Legend: The Voice Message Incident

Peter once told a story that surprised even himself.

One day he sent a voice message to Clawdbot — but he hadn't written any voice-handling code yet.

The agent received a file format it didn't recognize. So it:

1. Discovered that `ffmpeg` was installed on the computer
2. Used `ffmpeg` to convert the audio to a processable format
3. Found the OpenAI API key on the computer
4. Transcribed the voice to text using Whisper API
5. Understood the content and replied

**Nobody wrote this flow.**

The agent figured it out on its own.

Peter said he was "stunned." That was the moment he knew AI capability had crossed a genuine threshold.

![A legendary moment: voice processing that was never programmed](/assets/images/peter-steinberger-slides-09.png)

---

## The Rename Storm: Three Names in One Week, Hijacked in 10 Seconds

This was the most dramatic open-source project story of January 2026.

### Act One: Anthropic Calls

On January 27, 2026, Clawdbot had just hit 60,000 stars. Anthropic's legal team politely reached out to Peter — "Clawd" was too close to "Claude," potential trademark issue.

Peter later said: "I was forced to rename the account by Anthropic. Wasn't my decision." But he acknowledged Anthropic handled it respectfully.

He decided to rename to "Moltbot" — from the image of a lobster molting (shedding its shell). A lobster has to shed its old shell to grow. The metaphor was fitting.

### Act Two: 10 Seconds of Disaster

On the day of the rename, Peter needed to swap both the GitHub organization and the Twitter/X handle simultaneously.

His plan: release the old @clawdbot handle, then immediately register the new @moltbot handle.

**Problem: crypto scammers had been monitoring this project all along.**

Peter's own description: "I messed up the rename and my old name was snatched in 10 seconds."

In the gap between releasing the old handle and claiming the new one, scammers grabbed both the GitHub organization and the Twitter account.

They immediately started promoting fake $CLAWD tokens to tens of thousands of followers.

### Act Three: A $16 Million Scam

The scammers launched a fake $CLAWD token on Solana, riding the confusion of the rename and the project's momentum.

**The fake coin hit a $16 million market cap within hours.**

FOMO-driven investors thought they'd found an "early AI investment opportunity" and piled in.

Peter posted an emergency clarification:

> "To all crypto folks: Please stop pinging me, stop harassing me. I will never do a coin. Any project that lists me as coin owner is a SCAM."

The token collapsed immediately. The scammers who sold early walked away with millions. Late buyers lost everything.

### Act Four: OpenClaw Is Born

After this disaster, Peter decided to rename again.

This time: proper trademark search, domain registered, migration code written in advance. No rushing.

The new name: "OpenClaw."

- **Open:** open-source, community-driven, self-hosted
- **Claw:** keeping the lobster lineage

Some said the name was a nod to OpenAI. Peter didn't respond directly.

**One week. Three names. 60,000 stars became 103,000. Scammers walked away with $16M.**

Probably the most chaotic rename event in GitHub history.

But the project survived. Peter said it plainly: "The software itself was always compelling."

OpenClaw's positioning also evolved — from "Claude with hands" to "model-agnostic infrastructure." The rename actually matured the project.

---

## Honestly

### Peter Steinberger Isn't Building a Product — He's Running an Experiment

He's been clear: no interest in traditional VC funding, no desire to run a typical startup. He's considering setting up a nonprofit foundation so the project can outlive him.

His motivation is "fun" and "inspiration," not profit.

This aligns with what I analyzed in my [Shell Wrapper 2.0 article](/shell-wrapper-2-anthropic-real-threat/) — the reason Moltbot/OpenClaw can push boundaries so aggressively is precisely because it doesn't need to worry about "who's liable when something goes wrong" the way Anthropic does.

Open-source communities can bear their own risk. That's their freedom, and their constraint.

### The "Death of Apps" Prediction

Peter has a bold prediction:

> "Many consumer apps will melt away into APIs managed by personal agents."

You don't need a fitness app's UI — you just tell your agent what you ate.
You don't need an airline app's UI — you just tell your agent where you need to fly.

Agents will handle the things that "used to require an app."

This maps perfectly to the "digital employee" concept — an agent isn't a tool, it's a proxy.

### The Security Risk Is Real

Peter himself admits the project is "unsafe by design." It gives AI complete access to your computer — files, API keys, camera, all of it.

> "Prompt injection is not solved."

He's now building a team to address security. But until that work is done, this thing is fundamentally a "how much risk are you willing to take" decision.

That's why I wrote the [Moltbot Security Hardening Guide](/moltbot-security-hardening/) — the more powerful Shell Wrapper 2.0 gets, the more critical security engineering becomes.

---

## What I Learned from Peter Steinberger

1. **Frameworks don't matter — shipping does.** No RAG, no subagents, no complex orchestration, and his output is as good as anyone's.
2. **CLI is AI's natural interface.** Text in, text out. Nothing cleaner.
3. **Human-in-the-loop isn't a step backwards.** Full automation sounds cool, but "a person with taste guiding an agent" is what actually produces good work.
4. **Solve your own problem, then share it with the world.** PSPDFKit was built this way. So was OpenClaw. The best products come from real pain.
5. **Retirement isn't an endpoint — creation is.** A financially free person picks up the keyboard again because he felt his "mojo disappear." That kind of motivation outlasts any business plan.

---

## Further Reading

- [From "Shell Wrapper 1.0" to "Shell Wrapper 2.0": Why Anthropic Should Be Worried](/shell-wrapper-2-anthropic-real-threat/) — analyzing OpenClaw/Moltbot's impact on the Claude Code ecosystem
- [Moltbot Security Hardening: A Complete 4-Layer Defense-in-Depth Guide](/moltbot-security-hardening/) — if you're running OpenClaw, read this first
- [When Unix Philosophy Meets AI: The Command Line Renaissance](/unix-philosophy-command-line-renaissance/) — why CLI is AI's best interface
- [500 AI Assistants Exposed to the Public Internet: The Moltbot 0.0.0.0 Disaster](/moltbot-security-disaster/) — the security cost of Shell Wrapper 2.0
- [Peter Steinberger: Just Talk To It - the no-bs Way of Agentic Engineering](https://steipete.me/posts/just-talk-to-it) — Peter's original post

---

**Sources:**
- [YouTube: Peter Steinberger Interview (1)](https://youtu.be/AcwK1Uuwc0U)
- [YouTube: Peter Steinberger Interview (2)](https://youtu.be/qyjTpzIAEkA)
- [OpenClaw GitHub](https://github.com/clawdbot/clawdbot)
- [Peter Steinberger's Website](https://steipete.me/)
- [Peter Steinberger: Just Talk To It](https://steipete.me/posts/just-talk-to-it)
- [TechFlow: Behind ClawdBot's Viral Success](https://www.techflowpost.com/en-US/article/30106)
- [DigitalOcean: What is OpenClaw?](https://www.digitalocean.com/resources/articles/what-is-openclaw)
- [TechCrunch: PSPDFKit raises $116M](https://techcrunch.com/2021/10/01/pspdfkit-raises-116m-its-first-outside-money-now-nearly-1b-people-use-apps-powered-by-its-collaboration-signing-and-markup-tools/)
- [Semaphore: Peter Steinberger Startup Journey](https://semaphore.io/blog/peter-steinberger-startup-journey-pdf-quirks-and-wwdc19)
- [DEV Community: From Clawdbot to Moltbot](https://dev.to/sivarampg/from-clawdbot-to-moltbot-how-a-cd-crypto-scammers-and-10-seconds-of-chaos-took-down-the-4eck)
- [DEV Community: From Moltbot to OpenClaw](https://dev.to/sivarampg/from-moltbot-to-openclaw-when-the-dust-settles-the-project-survived-5h6o)
- [Wikipedia: OpenClaw](https://en.wikipedia.org/wiki/OpenClaw)

---

**About the Author:**

Wisely Chen, CTO at NeuroBrain Dynamics Inc., 20+ years in IT. Former Google Cloud consultant, VP of Data & AI at Yung Lian Logistics, Chief Data Officer at iLane Energy Data. Focused on real-world AI transformation and agent deployment in traditional industries.

---

**Links:**
- Blog: https://ai-coding.wiselychen.com
- LinkedIn: https://www.linkedin.com/in/wisely-chen-38033a5b/
