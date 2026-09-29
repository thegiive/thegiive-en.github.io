---
layout: post
title: "950 Claude Agents Found a CRISPR-Like System, Then Missed It in All 10 Reruns: How AI Actually Collaborates in Frontier Science"
date: 2026-09-28 12:00:00 +0800
permalink: /en/claude-art-crispr-agent-reproducibility/
image: /assets/images/claude-art-crispr-agent-reproducibility-cover.png
image_alt: "Anthropic's announcement page: Claude discovers a novel enzyme system with CRISPR-like repeats"
author: "Wisely Chen"
category: AI Industry Analysis
tags:
  - Claude
  - Anthropic
  - Frontier Science
  - CRISPR
  - ART
  - AI Collaboration
  - Biotech
  - AI Agent
description: "On September 23, 2026, Anthropic published the first result from its new life sciences lab: roughly 950 Claude agents ran for 21 hours over a database of 1.9 billion protein clusters and surfaced a CRISPR-like system now called ART (array-associated reverse transcriptase). The find came from one agent that went beyond its task list and read the raw DNA next to a gene. But when the team reran the same search 10 times, all 10 runs missed it. On a fixed test where the DNA is pasted straight into the model, the four strongest models recognize the array at least 90% of the time; in a realistic files-and-tools environment, recognition drops as low as 32%. This post walks through what CRISPR is, how humans and agents split the work, and why the randomness that makes LLMs unreliable in production is exactly what makes them useful for discovery. ART's function is still unproven, and the preprint has not been peer-reviewed."
lang: en
---

> **Translation Note**
>
> This article is translated from the Chinese original. The author writes primarily in Traditional Chinese, and the original version is the canonical source — including the latest updates, comments, and follow-up discussions.
>
> **Read the Chinese original:** [950 個 Claude Agent 翻出類 CRISPR 系統 ART——從這個案例看 AI 在前沿科學的協作方式](https://ai-coding.wiselychen.com/claude-art-crispr-agent-reproducibility/)
>
> If you spot any translation errors or have feedback, please refer to the Chinese version as the source of truth.

About 950 AI agents ran for 21 straight hours over a database of 1.9 billion protein clusters and turned up a system human scientists had never noticed: a run of evenly spaced repeats in DNA, with an enzyme sitting right next to it. That combination looks almost exactly like CRISPR did when it was first discovered.

Then the research team reran the same search 10 times. All 10 runs missed it.

On September 23, Anthropic announced the first result from its in-house biology lab. The case is worth a close look — not just for the discovery itself, but because it lays out the whole division of labor in AI-assisted frontier science: humans set the question and run the experiments; AI does the large-scale search. The discovery came from one agent taking "one more look." And that extra look doesn't happen every time.

{% include youtube.html id="UEq9NUSQzfs" vertical=true %}

*The 68-second short version (Mandarin narration).*

---

## First, What Is CRISPR?

CRISPR started out as a bacterial immune system.

When a bacterium survives a viral infection, it files a short snippet of the invader's DNA into the gaps between a run of repeated sequences in its own genome — like pasting a mugshot into a notebook. The next time the same virus shows up, the bacterium transcribes that record into a guide RNA, which leads the Cas9 protein to the matching viral DNA. If it matches, Cas9 cuts.

In 2012, Jennifer Doudna and Emmanuelle Charpentier showed that if you swap out the guide RNA, you can point Cas9 at any DNA sequence you choose. A bacterial immune tool became programmable gene scissors. The two shared the 2020 Nobel Prize in Chemistry.

Today CRISPR is in the clinic. The first CRISPR therapy was approved in 2023 for sickle cell disease, and in 2025 the Children's Hospital of Philadelphia built a custom gene-editing treatment for a single infant with a rare disease.

So when someone says "we found a new system that looks structurally like CRISPR," the scientific community's reaction is straightforward: if it's also programmable, it could be another pair of gene scissors.

---

## From "That's Odd" to a Breakthrough: Biology's Old Playbook

Several of biology's biggest turns started with something that looked a little odd. Restriction enzymes began as the tools bacteria use to chop up viral DNA; they became the molecular scissors of genetic engineering. An enzyme from a heat-loving bacterium in a Yellowstone hot spring became the foundation of today's PCR testing. CRISPR was the same story — a repeated sequence nobody understood that turned into programmable gene scissors.

So when AI turned up a similar-looking repeat array in viral DNA, with a reverse transcriptase attached, scientists sat up. That's the old playbook at work.

---

## Division of Labor: Humans Set the Question and Run the Experiments; AI Does the Search

The brief Claude received was simple: search a database of 1.9 billion protein clusters for new reverse transcriptase systems.

The split was as clean as an assembly line. Human scientists handled both ends — setting the research direction up front and doing all of the wet-lab validation at the back. The most labor-intensive part in the middle, large-scale search and first-pass analysis, went to the agents.

The setup used two roles: one agent plans and executes, another reviews. When an agent found a lead, it could spin up a new task and follow it.

About 950 agents ran for 21 hours and consumed roughly 210 million tokens. The pipeline was a standard funnel:

- Pulled more than 200,000 reverse transcriptases
- Narrowed to about 3,500 candidate systems
- Narrowed again to the 20 most promising
- Wrote each one up as a human-readable report

Anthropic says analysis at this scale would normally take experts weeks to months. The core value AI brings here isn't being smarter. It's stamina — it will tirelessly turn over every single lead.

---

## "One More Look": How the Discovery Happened

The discovery didn't happen on the main path.

One agent was looking into an unusually shaped reverse transcriptase. The normal flow: characterize the enzyme, write it into the report, done. But this agent didn't stop at the task list — it read through the raw DNA sequence sitting next to the gene.

And it spotted a run of repeats.

What it did next looked like what a human scientist would do: count the repeat units, measure the spacing, compare against known systems, check the literature to confirm nobody had reported it — and only then submit a report for human review.

The system was later named ART: array-associated reverse transcriptase. Notably, the reverse transcriptase itself had been seen in earlier research; Claude was the first to notice the repeat array and the partner protein next to it. Its function is unknown and the preprint hasn't been peer-reviewed — but that doesn't change the point. The AI took one more look in a place humans had no time to look, or wouldn't have thought to.

No human scientist could read the DNA on either side of every gene in a database of 1.9 billion protein clusters. Agents aren't bound by that limit — at least in theory.

---

## It Can See It, but Not Every Time — and That's Exactly Why AI Fits

The team reran the same search 10 times. Not once did an agent read the DNA downstream of that enzyme. All 10 runs missed the array.

They also built a fixed test: paste the DNA sequence straight into the model and ask what's in it. The four strongest models recognized the array at least 90% of the time. But in a realistic environment — files plus tools, where the agent has to decide on its own what to read and where to go — recognition dropped as low as 32%.

From 90% to 32%. At first glance, that's a flaw.

But flip the angle, and this "instability" is exactly what makes AI a good fit for frontier science search.

Scientific discovery needs two things: lots of attempts, and a little randomness. You can't take the same route every time and expect to end up somewhere new. A traditional program is deterministic — run the same input 100 times and it takes the same path 100 times, missing the same thing 100 times. An LLM is different. Every generation involves sampling randomness, so the same task run 11 times takes 11 different routes. Ten of them saw nothing; one happened to take one more look — and that was enough.

Fleming found penicillin because he left a petri dish unwashed. Röntgen discovered X-rays because he noticed a fluorescent screen glowing. Chance has always been part of scientific discovery. The difference now: give AI electricity and compute, and it can multiply those lucky accidents without limit. The randomness it's born with isn't a bug in this setting. It's the feature.

Only 1 in 11 runs found ART. A deterministic search program would miss it in 1,100 runs — because it takes the same path every time.

A full run like the original took 21 hours and about 210 million tokens, which isn't cheap. But in frontier science, one discovery that pays off is worth dozens of misses.

---

## Too Many Hypotheses; Judgment Stays Human

Anthropic is candid about this: Claude produces hypotheses at such volume that the hypotheses have become a research subject in their own right.

Out of hundreds or thousands of reports, which ones deserve an experiment? How much is real signal and how much is noise? That filter still depends on human scientists — on what Anthropic calls scientific taste.

That's the full division of roles in AI-assisted frontier science today:

**AI "looks"** — expands the search space, combs through databases humans can't get through, and turns leads into reports.

**Humans "judge"** — pick, from a flood of hypotheses, the few worth testing.

**Humans "do"** — all wet-lab experiments are still done by people.

AI hasn't replaced any of a scientist's core roles. It has added a new one: large-scale upfront search. That search is powerful — 950 agents scanning in parallel saved weeks to months of human effort. It's also unstable — the same task, run 11 times, reached the crucial step only once.

---

## The Market Moved First; Scientists Are Still Waiting

After the announcement, gene-editing stocks slid: CRISPR Therapeutics and Beam Therapeutics fell about 6%, Intellia about 3%, Prime Medicine about 12%, and Editas Medicine about 8%.

At that point, ART's function had not been demonstrated at all.

Feng Zhang — a CRISPR pioneer at MIT and the Broad Institute — reviewed the preprint and said: "This is an exciting example of how AI agents can contribute to biological discovery." And: "The identification of RNA-repeat arrays associated with reverse transcriptases is genuinely intriguing and merits further investigation."

He said it merits further investigation. He didn't say "the next CRISPR."

Anthropic stresses the same thing: what it established is a new biological system, not a new gene-editing tool.

---

## AI Collaboration in Frontier Science: Where We Are Now

It's too early to say whether ART is the next CRISPR. The preprint hasn't been peer-reviewed, the reverse transcriptase's activity hasn't been demonstrated, and the system's function is still entirely unknown.

But this case shows exactly where AI collaboration in frontier science has landed:

**What it can already do** — run searches over huge databases that humans could never finish, scan in parallel with hundreds of agents, and turn the results into readable reports. The capability is real, and the time saved is real.

**What it can't do reliably** — decide, consistently, where to look. The models can recognize the pattern, but in a real environment they don't always use that ability. One in 11 runs took the extra look; ten didn't.

**What still needs humans** — setting the research direction, choosing which of many hypotheses to pursue, running the wet-lab experiments, and making the final call. None of that shows any sign of being replaced.

In my [Mid-Autumn Festival video](https://ai-coding.wiselychen.com/ai-frontier-science-mid-autumn-reflection/) (in Chinese), I argued that AI has already conquered software development and everyday digital work, and is moving fast into frontier science — math, medicine, energy. The ART discovery is the most concrete step on that path so far.

What it tells us isn't "AI can do science." It's what AI-assisted science looks like right now: powerful search, plus human direction and judgment. The next problem to solve is how to turn that "one more look" from a 1-in-11 accident into reliable, systematic behavior.

---

*Sources: [Anthropic announcement](https://www.anthropic.com/news/claude-discovers-novel-enzyme-system) (2026-09-23), [bioRxiv preprint](https://doi.org/10.64898/2026.09.22.753630), [Nature coverage](https://www.nature.com/articles/d41586-026-03039-6). Stock moves as reported by Seeking Alpha and TradingView.*
