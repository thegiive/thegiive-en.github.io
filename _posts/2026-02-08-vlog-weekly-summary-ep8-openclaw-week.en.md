---
layout: post
title: "OpenClaw Week: From the Claude Code 1.5 Era to a Digital Jarvis | Weekly Vlog EP8"
date: 2026-02-08 12:00:00 +0800
categories: [vlog, openclaw]
tags: [weekly-vlog, openclaw, ai-agent, memory, token-optimization, enterprise]
permalink: /en/vlog-weekly-summary-ep8-openclaw-week/
image: /assets/images/vlog-ep8-openclaw-week-cover.png
description: "A week-long deep dive into OpenClaw — from creator Peter's builder philosophy to the Memory architecture (AGENTS.md, SOUL.md), three token-saving tricks (cut 50%+ easily), and the 'new employee' enterprise security strategy. The AI agent that comes closest to a real digital Jarvis. Worth your time to understand it properly."
lang: en
---

{% include youtube.html id="y6mxqAuSkeI" %}

**Published:** 2026-02-08
**Topic:** OpenClaw Deep Research Week — founder philosophy, architecture breakdown, real usage, cost optimization, and enterprise deployment
**Articles covered (2026-02-02 to 02-07):**

1. 02/02 - From $116M Exit to 100K GitHub Stars in 10 Days: Peter Steinberger's Agentic Engineering Philosophy
2. 02/03 - OpenClaw Memory System Deep Dive: SOUL.md, AGENTS.md, and the High Token Costs
3. 02/04 - I Used OpenClaw Intensively These Days: Why I Decided to Put It Into My Workflow
4. 02/05 - OpenClaw Architecture Fully Unpacked: The Six-Layer Design Every Agent Engineer Needs to Know
5. 02/06 - OpenClaw Token Optimization Guide: How to Cut AI Agent Operating Costs by 97%
6. 02/07 - How to Deploy OpenClaw in an Enterprise: Treating an AI Agent Like a Brand-New Employee

---

## Transcript

### Opening: The Claude Code 1.5 Era and OpenClaw's Next Level

This week we're introducing OpenClaw.

Last week I covered Claude Code — and because OpenClaw blew up at the same time, I touched on it there too. I started comparing the two. My take: Claude Code is a perfect artifact of the AI Agent 1.5 era.

It took a previously passive agent setup — running commands, calling lots of tools, tracking state — and wired up the hands, feet, and eyes of an AI agent. It showed us what a semi-automated AI agent can actually accomplish.

But OpenClaw is the next level. It adds something called **proactivity** — it can take actions on your behalf without being explicitly commanded, and it actively looks out for you.

This comes remarkably close to what people mean when they say **digital Jarvis**.

So this week I'll cover OpenClaw from every angle: the creator's interview, the architecture, the memory system, how to cut the API bill, and finally — given a tool this powerful, how do you integrate it into a company's workflow?

---

### Monday: Peter — Not Just an Engineer, But a Builder

On Monday I started with an interview of OpenClaw's creator, Peter.

What struck me most about Peter is that he's not just an engineer — he's a **Builder**. He understands what the product is, he writes great code, but he also handles customer needs, his own needs, and he knows how to ship. He sits at the intersection of engineer, product manager, and business person. That hybrid role is what "Builder" means now — someone who can take an idea all the way from concept to shipped product.

In the interviews he talks about why he built OpenClaw. The original goal was just to run Claude Code on a Mac while away from his desk, but as he built it he realized it was growing into something much bigger — a genuinely great digital assistant. And then, earlier this year, it suddenly exploded.

The biggest insight I took from it: Peter thinks the current AI agent obsession with MCP is largely pointless. He used simpler storage approaches that work better, and achieved something rare — **persistent, long-lived AI agent memory**.

I think this is genuinely significant. If you want the details, check the blog post or the YouTube interview — I found it excellent.

---

### Tuesday: Memory System — Basically a Pile of Markdown

On Tuesday I dug into OpenClaw's memory system. "Memory system" sounds fancy, but it's really just a well-organized collection of Markdown files.

On startup, there's **AGENTS.md** — it tells the agent what kind of agent it is and what it should do. Most of the system-level configuration lives here.

There's also an interesting file called **SOUL.md** — the agent's soul. You can write in it whether you want it to be proactive or passive, and how you want it to feel. More personality controls.

There are plenty of other smaller Markdown files too. One particularly clever design: every day's conversation logs go into a separate daily Markdown file. Every time the agent starts up, it reads today's and yesterday's discussion history.

This is why people say OpenClaw seems to "remember everything" you've talked about — uncannily good memory. That **alive-feeling** comes entirely from the agent dynamically assembling those Markdown files into context.

The downside, of course, is that this makes OpenClaw what people jokingly call an **API token shredder**. Use it casually and your bill will be enormous.

---

### My Experience Using OpenClaw as a Personal Assistant

By the time I'd done all this research, I was already running OpenClaw as my personal assistant. After a few weeks with it, my impression: it genuinely **feels alive**, and it's remarkably proactive.

The biggest difference from every other AI agent: **proactivity vs passivity**.

Everything before was reactive. You needed it, you asked it. Claude Code stepped it up — better terminal interface, more tools, more capability. But it was still fundamentally semi-automated.

OpenClaw is different. It has a **Heartbeat** built in — it checks periodically to see what it can do for you without waiting to be asked.

And combined with those deep conversation records it keeps, it really functions like a personal assistant who knows you and can get things done.

A few examples:

**Scheduling meetings:** I told it to remember a few people's email addresses. When I needed a meeting, I just said "schedule it" — and it sent a Google Calendar invite.

**Reading and summarizing articles:** Drop a Google Drive link, it handles the rest.

**Editing my videos:** This one blew my mind. Last week I had a YouTube video already up, and while eating lunch I thought — can you cut a Short from this? I dropped the link.

It immediately downloaded the YouTube video, extracted the transcript with Whisper, analyzed it for the three best segments to turn into Shorts, and came back to ask me which ones to proceed with.

The best part: it added, "**While I'm asking you, I've already started editing the first segment.**"

That's the proactivity. It doesn't always wait for your command — when it thinks it can do something useful, it starts.

You don't see that from other AI agents. And I think that's a big reason this thing went viral.

---

### Thursday: The IT Architecture — Peter Is a Remarkably Careful Developer

On Thursday, the inevitable IT architecture segment. I analyzed OpenClaw's design in detail.

My conclusion: this architecture reflects someone who is an **exceptionally detail-oriented developer**. It's genuinely elegant.

It also points to something broader: in the age of AI coding, the code itself matters less. What's rare is the ability to design a **clean architecture** that clearly reflects its intent and remains understandable. That skill is still scarce in the AI era.

OpenClaw is built for one very specific purpose: **a long-running, always-on digital assistant**.

That purpose drives several key design decisions:

**Memory architecture:** It never forgets.

**Proactive design:** It needs to maintain awareness of you and independently discover things to act on.

**Multi-agent skepticism:** When multiple agents are interleaving logs, debugging becomes extremely hard. So OpenClaw made a deliberate architectural call: **serial over parallel**.

The layered design:
- **Gateway** at the front
- Channel **adapters** on top of the Gateway
- The Gateway fans out to **Workers**
- Each task is a single serial lane, not a parallel branch

Because the agent runs with significant system permissions, the architecture also includes appropriate isolation controls.

---

### Thursday (continued): OpenClaw Architecture — A Study in Taste

Let me say it plainly: OpenClaw's architecture is **exceptionally tasteful**. This probably comes from Peter being an extremely senior developer.

From the start it was designed as a **long-running, multi-channel digital assistant**. It can even accept cron jobs.

Every layer reflects a deliberate design reason:

**Multi-channel native.** Discord, Slack, Telegram, WhatsApp — multi-channel is a first-class concept. Each channel has an adapter.

**Gateway unification.** The Gateway normalizes all the incoming channels and routes them into a **lane queue** design. It determines which pipeline each message belongs to and sends it there.

**Forced serial execution.** Most multi-agent architectures push parallel execution because it's faster. But Peter noticed that parallel architectures are very hard to debug. So he made a deliberate trade-off: **forced serial execution within a pipeline**. You can still do parallel, but within a pipeline it's single-threaded by default.

Very engineering-minded. The trade-off is that each task takes longer. You'll notice OpenClaw finishes processing before it replies. But that makes the logic cleaner and the whole system more reliable.

This also fits the long-running digital assistant use case. It's like delegating a task to a team member — they don't instantly respond, they go do the work and come back 30 minutes or an hour later with results or problems. That's exactly how OpenClaw positions itself. The native use case of a digital assistant allows — and even expects — this pattern: queue a task, let it run to completion overnight, get a report back.

---

### Friday: Token Optimization — Cut 50% with Minimal Effort

On Friday I tackled another entertaining topic. Everyone calls it an **API token shredder** — so how do you actually optimize the token usage?

This has been discussed a lot, and some good, practical approaches have emerged. I tried them myself. My result: **easily cut 50% or more**. Some posts claim 80–97%.

The core ideas:

**Idea 1:** The memory system includes chat history — all conversation logs get loaded into every session by default. Many people argue this isn't always necessary. Set the default to load only **AGENTS.md, SOUL.md, and TOOLS.md**. Load full history only when needed. This alone makes a significant difference.

**Idea 2:** OpenClaw runs many small tasks — like the Heartbeat. Periodic checks, proactive actions, lightweight planning. You can use configuration to **route different tasks to different models**. For most things, use **Haiku** (fast, cheap, lightweight). Only use **Opus** for heavy planning tasks.

This barely degrades quality while dramatically cutting cost — at least 50%.

**Idea 3:** The Heartbeat only needs a small model. The recommendation is to run a local **Ollama** instance with something like **Llama 3.2 3B** — more than sufficient for that task.

Personally I have mixed feelings — the Heartbeat is one of OpenClaw's most interesting features because of its proactivity. If you're worried a tiny local model will degrade the experience, you can route it to a free API model like **Gemini Flash** or **Haiku** instead. Similar cost savings, similar quality floor.

There are other options too: **prompt caching**, **rate limiting**. I think investing 5–10 minutes in this configuration is worth it — it can meaningfully lower your bill.

One more note: many people use subscription plans like ChatGPT Pro or Claude Max to run this. Worth flagging: this carries a significant **account ban risk**. It keeps your costs predictable, but that's the trade-off — use at your own discretion.

For enterprise use: strongly recommend using the API directly, with proper token optimization configured. For personal use: if you're fine with the ban risk, subscriptions are the simplest path.

---

### Saturday: Enterprise Deployment — Treat OpenClaw Like a New Employee

On Saturday we got to enterprise integration — and the thing I'd been working toward all week: how do you actually put OpenClaw into a **company's workflow**?

This matters. Most of the conversation around OpenClaw is personal assistant, personal use — not enterprise. But enterprise is where the real leverage is.

My starting position: don't just put it on the company network yet. Wait until you know how to use it. After a week of thinking this through, talking to people, here's a framework that gets you to a reasonably solid defensive posture.

**First: treat OpenClaw as a person.** A real person. Because it genuinely behaves like one. And if you treat it like a **new hire**, a lot of the thinking falls into place naturally.

Concretely: **don't let it impersonate you.** Never let OpenClaw log into your company account. Create a new company-managed account for it — like you would for a new employee. Give it that new employee account's credentials.

We use **Google Workspace**. I created a new Workspace account for it. When I need it to access a document, I share permission to that account — just like I'd share access with a real person. When I'm done, I revoke if needed.

This mirrors how you'd onboard a real new hire: you don't hand them the keys to everything. You share access to what they need. You don't know how they'll behave — same as with OpenClaw.

The bonus: **IT's existing SOP doesn't need major rewrites.** The same information security controls you apply to new employees apply here.

**Second: for internal system access, write the Skill yourself first.** Don't let OpenClaw write the code that connects to internal systems. Write it yourself, get it right, then hand the Skill to OpenClaw.

More importantly: **hardcode the guardrails in the code itself, not in the prompt.**

Why? Because prompts are natural language — no matter how strict, prompt injection can bypass them. Code-level controls are fixed logic. They cannot be "talked around."

---

### Email: The Highest-Risk Component

**Email deserves its own discussion — it's possibly the biggest risk in the whole enterprise setup.**

Internal documents, internal system access — the input is controllable because it comes from inside the company.

Email is different. It's the **one external-facing channel** that accepts arbitrary input from anyone. It may be OpenClaw's only public-facing node. That demands the strictest protection.

**Rule 1:** Don't expose the new employee account's email address externally. If people outside the company know it exists, they can target it.

**Rule 2:** Don't trust the LLM to read raw email directly. Far too many security incidents involve emails with white-on-white text containing prompt injection instructions that humans can't see. The LLM reads them, gets hijacked, and becomes someone else's agent.

The approach: **write a Skill** that reads email programmatically first. The program extracts content, sanitizes it, and outputs clean structured JSON — title, content, sender, timestamp. Only then does the LLM see it.

This is an application of the **Guardrail** concept. It significantly reduces the risk of direct external attacks.

**Operationally, how does this work?**

For internal system emails (JIRA, etc.) — set up a Gmail filter to auto-forward directly to OpenClaw. Very low injection risk.

For external emails — I personally filter everything first. If I decide something is worth delegating, I read it and forward it manually.

**Two layers of protection:**

1. Code-level content sanitization before the LLM ever sees the email
2. Human filtering — only explicitly delegated emails reach the assistant

This gives you meaningful, layered risk reduction.

---

### Weekly Wrap-Up: Why I Spent a Whole Week on OpenClaw

Why go this deep? Architecture, Memory, creator mindset, cost optimization, enterprise integration — why all of this?

Because **I genuinely believe OpenClaw represents something important in the future of AI agents.**

I'm not certain this specific project survives long-term or becomes the canonical piece of software in this space. But the pattern it demonstrates — a **personal secretary, a digital assistant with genuine proactivity**, strong memory, and the full expressive power of a real computer — this is what the AI agent era demands.

So we need to understand it:
- What kind of creature is it? (Architecture and Memory dimensions)
- What's its biggest problem — the token shredder — and how do we fix it?
- How do we integrate it into enterprise workflows?

I genuinely believe this is **the closest thing I've seen in the past year or more to an AI solution that can actually change how a company operates**.

That's why it's worth the time to study, understand, and improve it.

That's my wrap for the week. Thank you all.

---

## Article Links

1. [From $116M Exit to 100K GitHub Stars in 10 Days: Peter Steinberger's Agentic Engineering Philosophy](/peter-steinberger-agentic-engineering-just-talk-to-it/)
2. [OpenClaw Memory System Deep Dive: SOUL.md, AGENTS.md, and the High Token Costs](/openclaw-architecture-deep-dive-context-memory-token-crusher/)
3. [I Used OpenClaw Intensively These Days: Why I Decided to Put It Into My Workflow](/openclaw-real-world-usage-workflow-not-chatbox/)
4. [OpenClaw Architecture Fully Unpacked: The Six-Layer Design Every Agent Engineer Needs to Know](/openclaw-architecture-deep-dive-what-claude-code-didnt-tell-you/)
5. [OpenClaw Token Optimization Guide: How to Cut AI Agent Operating Costs by 97%](/openclaw-cost-optimization-guide-97-percent-reduction/)
6. [How to Deploy OpenClaw in an Enterprise: Treating an AI Agent Like a Brand-New Employee](/openclaw-enterprise-deployment-new-employee-mindset/)
