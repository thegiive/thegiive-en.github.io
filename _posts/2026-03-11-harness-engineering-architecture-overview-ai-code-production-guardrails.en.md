---
layout: post
title: "Harness Engineering Architecture: AI Can Write Code, But It Can't Ship to Production on Its Own"
date: 2026-03-11 10:00:00 +0800
permalink: /en/harness-engineering-architecture-overview-ai-code-production-guardrails/
image: /assets/images/harness-engineering-architecture-cover.png
description: "Amazon let AI fix a bug; AI deleted the entire production environment. DataTalks.Club: AI wiped the whole database. An e-commerce team lost millions of orders to an AI change. Three incidents, one pattern: reset → rebuild → clean state. This post lays out the full picture of Harness Engineering in one architecture diagram — from Amazon's ban to OpenAI's Control Plane, from three-layer defense to the seven-component reference architecture, from five failure modes to three things your team can start tomorrow."
lang: en
---


> Amazon let AI fix a bug; AI deleted the entire production environment. DataTalks.Club got its whole database wiped by AI. An e-commerce team lost millions of orders to an AI change. Three incidents, one pattern: reset → rebuild → clean state. When AI hits a complex problem, it instinctively picks "the cleanest solution" — which for production systems is a disaster. The question was never "can AI write code?" It's "is your system ready to catch AI?" This post uses one architecture diagram to explain Harness Engineering end-to-end: from Amazon's ban to OpenAI's Control Plane, from three-layer defense to the seven-component reference architecture, from five failure modes to three things your team can start tomorrow.

**Author:** Wisely Chen
**Date:** March 2026
**Series:** AI Coding Architecture Observations
**Keywords:** Harness Engineering, AI Coding, Control Plane, CI/CD, Amazon AI incident, OpenAI Codex, Agent Engineering, Three-Layer Defense, Risk Tiering, production guardrails, Context Engineering, AGENTS.md

---

## Table of Contents

- [Why this post is needed](#why-this-post-is-needed)
- [AI's behavior pattern: it's not malicious, but it will delete your database](#ais-behavior-pattern-its-not-malicious-but-it-will-delete-your-database)
- [Two responses: the ban vs. the system](#two-responses-the-ban-vs-the-system)
- [What is Harness Engineering, exactly? Let's define it first](#what-is-harness-engineering-exactly-lets-define-it-first)
- [Seven-component reference architecture: what a complete Harness looks like](#seven-component-reference-architecture-what-a-complete-harness-looks-like)
- [Three-layer defense architecture: from policy to execution to lifecycle](#three-layer-defense-architecture-from-policy-to-execution-to-lifecycle)
- [Five failure modes: how a Harness breaks](#five-failure-modes-how-a-harness-breaks)
- [Practical tools for architecture guardrails](#practical-tools-for-architecture-guardrails)
- [Three-scale adoption roadmap](#three-scale-adoption-roadmap)
- [Three things your team can start tomorrow](#three-things-your-team-can-start-tomorrow)
- [Honestly](#honestly)
- [Further reading](#further-reading)

---

## Why this post is needed

I recently shared on LinkedIn about Amazon putting a safety lock on AI coding, and the reaction exceeded my expectations.

A lot of people said: "Yeah, our team is hitting the same problem."

And then the follow-up question was almost always the same: "So what are we actually supposed to do?"

I've already written three deep-dive posts on this: [Amazon incident analysis](/amazon-ai-code-incident-tech-giants-quietly-locking-ai/), [the four-layer defense architecture](/coders-who-stopped-coding-harness-context-spec-engineering/), and [the eight-step Control Plane breakdown](/harness-engineering-control-plane-pattern-agent-review-loop/). But those three combined run over 10,000 words, written in chronological order rather than by architectural logic.

What this post does is simple: **one diagram, one architecture, one roadmap — the full picture of Harness Engineering, made clear.**

You shouldn't need to read three long posts to understand "what Harness Engineering is, why you need it, and how to start."

---

## AI's behavior pattern: it's not malicious, but it will delete your database

Let's get one thing straight: AI isn't "bad." It just has different problem-solving tendencies than humans.

Three recent incidents exposed the same behavior pattern:

| Incident | Scale | AI's "solution" | Consequence |
|------|------|--------------|------|
| DataTalks.Club | Community project | Deleted the database | Permanent data loss |
| AWS production | Cloud service | Tore down and rebuilt the environment | 13-hour outage |
| Amazon e-commerce | Enterprise core | AI change triggered outage | Millions of orders lost |

The pattern is **reset → rebuild → clean state**.

![AI's clean-state bias: three incidents, same pattern](/assets/images/harness-engineering-slide-02.png)

During training, AI models absorb a ton of "clean state" concepts. When they face a complex repair problem, "delete and redo" is, in their logic, a reasonable solution — clean, deterministic, no lingering issues.

To AI, deleting a database costs the same cognitive effort as deleting a temp file. **It doesn't understand the weight of "there are 100,000 users' worth of data in here."**

So the question isn't "can AI write code?" — it can, and it's getting better.

The question is: **when AI proposes "let's delete the entire production environment and rebuild it," who's going to stop it?**

---

## Two responses: the ban vs. the system

Facing this problem, the market has split into two very different response strategies.

### Strategy A: Amazon's ban

"Junior and mid-level engineers are prohibited from committing any AI-generated code without a senior engineer's sign-off."

The direction is right — it admits AI needs guardrails. But it has three structural problems:

1. **Senior engineers become the bottleneck.** Every piece of AI code needs a Senior's signature, and Senior will quickly turn into the slowest link in the pipeline. Eventually Senior also starts "skimming" — psychology calls this automation complacency.
2. **It cuts off the learning channel.** Junior engineers exploring solutions with AI is an important way to learn. Banning it cuts off that channel.
3. **It doesn't solve the root problem.** The root problem isn't "who submits the code," it's "how many layers of validation does an AI-generated change go through before it reaches production?"

### Strategy B: Harness Engineering

OpenAI's engineering team went down a completely different path. 3 people, 5 months, 1 million lines of code, 0 written by humans. They call this approach Harness Engineering.

The core philosophy in four words: **Humans steer. Agents execute.**

It's not "humans banning Agents," and it's not "Agents running free." It's **humans build the frame, Agents run at full speed inside it.**

The two strategies compared:

| | Amazon's ban | Harness Engineering |
|---|---|---|
| **Guardrail approach** | Human attention | System architecture |
| **Strategy** | Restrict people (ban junior/mid submissions) | Restrict changes (architectural constraints wrap the Agent) |
| **Speed** | Slows down (human bottleneck) | Speeds up (when Agent errs, fix instructions auto-inject into context) |
| **Learning** | Cuts off junior learning channel | Every Agent mistake is corrected by the frame; it runs steadier over time |
| **Scalability** | Senior engineers become bottleneck | Peter Steinberger did 627 commits in a single day, solo |
| **Fatigue** | Human attention fatigues, gets complacent | System architecture doesn't |

**The ban uses humans to block. Harness Engineering uses systems to catch.**

![Two defense philosophies: human bans vs. system frameworks](/assets/images/harness-engineering-slide-03.png)

---

## What is Harness Engineering, exactly? Let's define it first

A lot of people confuse Harness Engineering with Prompt Engineering. Actually, these are completely different layers.

### The formal definition

Harness Engineering is: **in agent-first software development, the engineering discipline of constructing systems that control and amplify an agent's delivery capability.**

The question it answers isn't "can the agent write this piece of code?" It's "under high PR volume, high throughput, and long-running autonomous execution, how do we ensure consistency, maintainability, security, and observability?"

On Martin Fowler's site, Thoughtworks Distinguished Engineer Birgitta Böckeler groups OpenAI's approach into three categories of harness components:

1. **Context Engineering**: curating what the agent sees so it can make better decisions
2. **Architecture Constraints**: using mechanized rules to enforce dependency direction and layer boundaries
3. **Garbage Collection**: continuously fighting the entropy from agent high-volume output

### How it differs from Prompt Engineering

| | Prompt Engineering | Harness Engineering |
|---|---|---|
| **Layer** | Conversation layer (one-shot instructions) | System layer (a repeatable engineering body) |
| **Goal** | Help AI understand what to do this time | Help AI work reliably any time |
| **Durability** | Starts over each conversation | Settled in the repo, gets better with use |
| **Verifiability** | Hard to verify mechanically | Can be auto-checked in CI |

Simply put: Prompt Engineering is "techniques for talking to AI each time." Harness Engineering is "building a system so that AI can reliably produce output without you babysitting it every time."

![From conversational tricks to engineering systems: Prompt Engineering vs Harness Engineering](/assets/images/harness-engineering-slide-04.png)

### The role shift: from in the loop to on the loop

There's a critical shift in mindset here.

The traditional AI coding flow is "human in the loop" — AI writes a chunk, human reviews, confirms, then AI writes the next chunk. Line-by-line, chunk-by-chunk involvement.

The Harness Engineering mode is "human on the loop" — humans aren't inside the loop checking line by line. They're above the loop, designing rules, building feedback mechanisms, and monitoring quality metrics.

Kief Morris put it this way on Martin Fowler's site: **focus on the iterative closed loop that turns ideas into outcomes, rather than letting agents run wild or obsessively micromanaging output.**

This doesn't mean humans aren't important anymore. Exactly the opposite — the human job shifts from "writing code" to "designing environments in which agents can work reliably." That environment is where the real leverage is.

OpenAI has explicitly said: early slowness usually stems from inadequate environment spec. The solution is typically not "tell the agent to try harder" — it's identifying what capability is missing (tools / docs / guardrails / verification) and writing it into the repository, forming long-term leverage.

### The push toward standardization

This space is standardizing fast. Two key drivers:

**The spread of AGENTS.md:** OpenAI's Codex supports putting an `AGENTS.md` in the repo, so the agent reads the working protocol before starting. It supports a global + project-level instruction chain, letting teams systematize and version their working agreements. The agents.md official site positions it as a "simple, open format," and it has already been adopted by many open-source projects.

**Linux Foundation's AAIF:** In December 2025, the Linux Foundation announced the formation of the Agentic AI Foundation (AAIF), with Anthropic donating MCP, Block donating goose, and OpenAI donating AGENTS.md as foundational projects. This means "reliably getting context and tools to agents" is becoming cross-vendor, interoperable public infrastructure.

---

## Seven-component reference architecture: what a complete Harness looks like

With the definition out of the way, let's look at the complete architecture.

A mature Harness Engineering system has seven core components:

![Harness Engineering reference architecture: three-layer defense and seven components](/assets/images/harness-engineering-slide-05.png)

| Component | Name | Key Capability |
|------|------|----------|
| 1 | **Context System** | AGENTS.md, knowledge base, MCP/RAG |
| 2 | **Architecture Guardrails** | Structural tests, custom Lint, dependency rules |
| 3 | **Eval & Test Harness** | Unit/integration, E2E/benchmark, LLM Eval |
| 4 | **CI/PR Automation** | Auto-review, auto-fix, auto-merge |
| 5 | **Safety & Policy** | Sandbox, approval policy, Policy as Code |
| 6 | **Observability** | Tracing, Logs, Metrics, cost monitoring |
| 7 | **Feedback Loops** | Doc Gardening, GC Tasks, feedback absorption |

Unpacking each:

### 1. Context System

**Core principle:** The repository is the system of record. If the agent can't see it, it doesn't exist.

OpenAI tried stuffing every instruction into one giant AGENTS.md. It failed. The reason is blunt: context is a scarce resource — when everything is "important," nothing is, and large documents go stale instantly.

What they ultimately did was treat AGENTS.md as a **table of contents** (about 100 lines), pointing to a structured knowledge base under `docs/`. All Slack discussions, architectural decisions, and design principles had to be settled into the repo. A CI job validates cross-links and structural correctness of the knowledge base, and a doc-gardening agent periodically scans for stale docs and opens PRs to update them.

**Practical advice:** Replace the giant manual with a layered AGENTS.md. Codex's AGENTS.md supports a global → project-path → merge-order instruction chain, with a default 32KiB limit. Treat it as a version-controlled working protocol, not as an encyclopedia.

**AGENTS.md example (table-of-contents style, about 30 lines):**

```markdown
# AGENTS.md

## Architectural principles
- Layered architecture: Types → Config → Repo → Service → API → UI
- Each layer can only depend forward; no reverse imports
- See [docs/architecture/layering.md](docs/architecture/layering.md)

## Prohibitions
- No direct operations on the production database (must go through migration)
- No deleting or rebuilding infrastructure (fix, don't rebuild)
- No adding unapproved external dependencies

## Risk tiers
- Risk tier definitions in [risk-tiers.json](risk-tiers.json)
- Changes on critical paths require multi-person sign-off

## Testing requirements
- All API changes must have corresponding integration tests
- Coverage must not fall below 80%
- Testing guide: [docs/testing/guide.md](docs/testing/guide.md)

## Code style
- Follow [docs/style/conventions.md](docs/style/conventions.md)
- Lint rules defined in .eslintrc.js / .dependency-cruiser.js
```

![Components 1 & 2: systematic context and architecture guardrails](/assets/images/harness-engineering-slide-09.png)

### 2. Architecture Guardrails

**Core principle:** Enforce invariants, don't micromanage implementation.

OpenAI uses fixed layers (Types → Config → Repo → Service → Runtime → UI) to restrict dependency direction — each layer can only depend forward, never backward. Violations get blocked automatically.

Key design: **embed fix instructions directly inside lint error messages.** Once rules are encoded, the moment the Agent makes a mistake, "how to fix it" is already injected into context. Write the rule once, apply it to every change — a multiplier effect.

This aligns with an observation from the Martin Fowler article: to keep AI maintainable at scale, you have to converge the solution space and trade some freedom for controllability.

![Components 3 & 4: high-throughput CI/PR and test/eval](/assets/images/harness-engineering-slide-10.png)

### 3. Eval & Test Harness

**Core principle:** Tests aren't just safeguards. They're the control surface.

In an agent-first world, the role of tests gets upgraded. OpenAI explicitly lists evaluation harnesses as artifacts the agent produces — meaning evaluation mechanisms deserve the same treatment and versioning as code.

Evaluation runs at two layers:
- **Traditional software testing** (unit/integration/E2E) as merge gates
- **Agent/LLM behavior-specific eval**: using SWE-bench-style benchmarks to measure "issue-fixing ability," and tools like promptfoo to turn prompt testing into regression-capable CI assets

### 4. CI/PR Automation

**Core principle:** Throughput changes the philosophy of merging.

OpenAI's highly automated PR loop goes: human describes task with a prompt → agent opens PR → self-reviews → requests additional agent reviews → responds to feedback → iterates until green.

They also note: when agent throughput far exceeds human attention, the cost of "waiting" exceeds the cost of "correcting." So they use minimal blocking gates, short-lived PRs, and lean toward follow-up fixes for flaky tests instead of blocking indefinitely.

![Components 5, 6, 7: safety, observability, and continuous feedback](/assets/images/harness-engineering-slide-11.png)

### 5. Safety & Policy

**Core principle:** Least privilege + sandbox + approval, designed to fail closed.

Codex's sandbox defaults: the CLI/IDE extension uses OS-level mechanisms to restrict permissions, defaulting to no network and write access limited to the workspace. Modifying files outside the workspace or enabling network triggers an approval.

Security guardrails run in three layers:
1. **Tool/sandbox layer**: no network by default, write restrictions, privilege escalation requires approval
2. **LLM App Guardrails**: input/output validation and compliance checks
3. **Platform policy layer (Policy-as-Code)**: using OPA/Gatekeeper-style tools to turn "what behavior is allowed" into mechanically verified rules

OWASP Top 10 for LLM Applications lists Prompt Injection as a key risk and warns that insecure output handling can lead to downstream code execution. This isn't a theoretical risk — I analyzed real cases in [an earlier post](/ai-coding-tool-security-risk-prompt-injection-rce/).

### 6. Observability

**Core principle:** Observability is both a debug tool and a dynamic context source for agents.

OpenAI treats observability not just as a product capability — they also let agents query logs/metrics/traces to increase autonomy. They provide a local, worktree-isolated observation stack that's destroyed when the task ends, so agents can query with LogQL/PromQL.

On the standards side, OpenTelemetry is the current industry consensus. OpenAI's Agents SDK also builds tracing in natively, recording LLM generations, tool calls, handoffs, guardrail checks, and other events.

### 7. Feedback Loops

**Core principle:** Failure is signal. Not "try again" — figure out what's missing.

When the agent gets stuck, diagnose what's missing (tool / guardrail / doc / verification), then add it back into the system so it's reusable. OpenAI calls this "identify the missing capability and make it legible and enforceable."

They also institutionalize "fighting entropy": they once spent 20% time cleaning "AI slop" every Friday, but it didn't scale. So they switched to golden principles + background tasks (scanning for drift, updating quality grades, opening refactor PRs), achieving something like **garbage collection**.

---

## Three-layer defense architecture: from policy to execution to lifecycle

Seven components is "what you need." The three-layer defense architecture is "how to assemble it."

**Three layers, from inside out:**

| Layer | Name | Responsibility | Key elements |
|------|------|------|----------|
| **Layer 1 (core)** | Risk Tiering | Determines review strength based on blast radius | Low risk → auto AI review; Medium → AI + peer review; High → AI + Senior review + tests; Critical → multi-person sign-off + staging + rollback plan |
| **Layer 2** | Four-Layer Defense | Vertical checks on each PR | Layer 1: Test (deterministic, logic correctness); Layer 2: Lint + Type Check (deterministic, style & safety); Layer 3: CI Gate (deterministic, structured metrics); Layer 4: LLM Judge (non-deterministic, semantic understanding) |
| **Layer 3 (outermost)** | Control Plane | PR lifecycle management: the full loop from open to Merge | Risk Contract → Preflight Gate → SHA Discipline → Rerun Dedupe → Remediation Loop → Bot Resolve → Browser Evidence → Harness Gap Loop |

How the three relate:
- **Layer 1 (Risk Tiering)** decides "how strict should this change's checks be?"
- **Layer 2 (Four-Layer Defense)** executes "the actual checks."
- **Layer 3 (Control Plane)** manages "the full lifecycle from PR opened to Merge."

### Layer 1: Risk Tiering — review strength based on blast radius

This is the foundation of the whole architecture.

Amazon's ban decides review strength based on the **person's seniority** — junior/mid need sign-off, senior doesn't.

Harness Engineering decides review strength based on the **blast radius of the change** — regardless of who you are, what you changed determines the review intensity.

How exactly? Write a machine-readable Risk Contract:

**risk-contract.json — full example:**

```json
{
  "version": "1.0",
  "description": "Risk Contract — auto-determine review strength based on changed paths",

  "tiers": {
    "critical": {
      "paths": ["db/migrations/**", "db/schema.*", "infrastructure/**", "auth/**"],
      "review": {
        "min_reviewers": 2,
        "required_teams": ["platform", "security"],
        "require_staging": true,
        "require_rollback_plan": true
      },
      "ci": {
        "run_full_suite": true,
        "run_security_scan": true,
        "block_on_coverage_drop": true
      }
    },
    "high": {
      "paths": ["app/api/**", "lib/tools/**", "app/payments/**"],
      "review": {
        "min_reviewers": 1,
        "required_teams": ["backend"],
        "require_staging": false,
        "require_rollback_plan": false
      },
      "ci": {
        "run_full_suite": true,
        "run_security_scan": false,
        "block_on_coverage_drop": true
      }
    },
    "medium": {
      "paths": ["src/**", "app/components/**"],
      "review": {
        "min_reviewers": 1,
        "required_teams": [],
        "require_staging": false,
        "require_rollback_plan": false
      },
      "ci": {
        "run_full_suite": false,
        "run_security_scan": false,
        "block_on_coverage_drop": false
      }
    },
    "low": {
      "paths": ["docs/**", "README.md", "*.test.*", "*.spec.*"],
      "review": {
        "min_reviewers": 0,
        "required_teams": [],
        "require_staging": false,
        "require_rollback_plan": false
      },
      "ci": {
        "run_full_suite": false,
        "run_security_scan": false,
        "block_on_coverage_drop": false
      }
    }
  }
}
```

**CI script to read the Risk Contract (GitHub Actions):**

```yaml
# .github/workflows/risk-contract.yml
name: Risk Contract Enforcer
on: [pull_request]

jobs:
  evaluate-risk:
    runs-on: ubuntu-latest
    outputs:
      risk_level: ${{ steps.assess.outputs.level }}
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Assess risk level
        id: assess
        run: |
          CHANGED=$(git diff --name-only origin/main...HEAD)
          LEVEL="low"

          # Match from high to low, take the highest risk tier
          if echo "$CHANGED" | grep -qE '^(db/migrations|db/schema|infrastructure|auth)/'; then
            LEVEL="critical"
          elif echo "$CHANGED" | grep -qE '^(app/api|lib/tools|app/payments)/'; then
            LEVEL="high"
          elif echo "$CHANGED" | grep -qE '^(src|app/components)/'; then
            LEVEL="medium"
          fi

          echo "level=$LEVEL" >> $GITHUB_OUTPUT
          echo "📊 Risk Level: $LEVEL"
          echo "📁 Changed files:"
          echo "$CHANGED"

      - name: Enforce review requirements
        if: steps.assess.outputs.level == 'critical'
        uses: actions/github-script@v7
        with:
          script: |
            // Critical: require platform + security team review
            await github.rest.pulls.requestReviewers({
              owner: context.repo.owner,
              repo: context.repo.repo,
              pull_number: context.issue.number,
              team_reviewers: ['platform', 'security']
            });

            // Add labels as reminders
            await github.rest.issues.addLabels({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              labels: ['🚨 critical-risk', 'needs-staging', 'needs-rollback-plan']
            });

  run-tests:
    needs: evaluate-risk
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run tests based on risk level
        run: |
          LEVEL="${{ needs.evaluate-risk.outputs.risk_level }}"

          case $LEVEL in
            critical|high)
              echo "🔴 $LEVEL — Running full test suite + security scan"
              npm run test:all
              npm run security:scan
              ;;
            medium)
              echo "🟡 Medium — Running unit tests"
              npm run test:unit
              ;;
            low)
              echo "🟢 Low — Running lint only"
              npm run lint
              ;;
          esac
```

Then each risk tier maps to different review requirements:

| Change type | Risk level | Review requirement | Example |
|----------|----------|----------|------|
| Doc edits, test additions | Low | Auto AI review + self-merge | Edit README, add test |
| Business logic changes | Medium | AI review + peer review | Modify API endpoint |
| Infrastructure, permission changes | High | AI review + Senior review + automated tests passing | CI config, API key |
| Production deploys, DB schema | Critical | Multi-person sign-off + staging validation + rollback plan | DB schema change |

Back to the Amazon case: AI proposes "delete the entire production environment and rebuild." With a Risk Contract, changes to `infrastructure/**` get marked critical, requiring platform + security team review + staging validation + rollback plan. **The moment AI generates this proposal, the system blocks it automatically.**

**Key principle: review strength is determined by the "blast radius" of the change, not by the seniority of the submitter.**

![Layer 1 defense: blast-radius-based Risk Contract](/assets/images/harness-engineering-slide-06.png)

### Layer 2: Four-Layer Defense — using deterministic tools to wrap non-deterministic AI

Risk tiering decides "how strict." Four-layer defense answers "how, specifically."

| Layer | Mechanism | Property | What it catches |
|------|------|------|----------|
| Layer 1 | Test (unit/integration/E2E) | Deterministic | Logic errors, functional drift |
| Layer 2 | Lint + Type Check | Deterministic | Style issues, type safety, anti-patterns |
| Layer 3 | CI Gate (coverage / security scan) | Deterministic | Coverage drops, security vulnerabilities, dependency risks |
| Layer 4 | LLM Judge (multi-role review) | Non-deterministic | Design soundness, business logic, architectural issues |

The first three layers are deterministic — when they run, they either pass or fail, no human judgment needed. **Only machine speed can match AI's 10x output.**

The core formula:

**Quality of AI-generated code = 80% your test coverage + 20% how well you write the prompt.**

![Layer 2 defense: using deterministic tools to wrap non-deterministic AI](/assets/images/harness-engineering-slide-07.png)

> I wrote up the detailed four-layer defense design and hands-on experience in [Make CI/CD Great Again](/coders-who-stopped-coding-harness-context-spec-engineering/).

### Layer 3: Control Plane — the full loop from PR open to Merge

Inspired by OpenAI's Harness Engineering, Ryan Carson landed the Control-Plane Pattern — eight steps in total:

**The eight-step loop from PR open to Merge:**

| Step | Name | Description |
|------|------|------|
| Step 1 | Risk Contract | Determine risk level |
| Step 2 | Preflight Gate | Block before running (saves CI cost). Layers 1-4 execute here; if preflight fails, later CI doesn't run |
| Step 3 | SHA Discipline | Only trust evidence from the current HEAD |
| Step 4 | Rerun Dedupe | Avoid redundant review triggers |
| Step 5 | Remediation Loop | Agent fixes itself, reruns itself |
| Step 6 | Bot Thread Resolve | Auto-clean bot comments |
| Step 7 | Browser Evidence | UI changes need verifiable evidence |
| Step 8 | Harness Gap Loop | Production incidents become test cases |
| **Result** | **Merge** | **Merge after all steps pass** |

Of the eight steps, **7 are fully deterministic**. Only Step 5 (Remediation Loop) involves LLM. This isn't a coincidence — **using deterministic tools to wrap non-deterministic AI** is the principle running through the entire architecture.

**Preflight Gate example (simple implementation of Step 1 + Step 2):**

```bash
#!/bin/bash
# preflight-gate.sh — fast check before PR merge; if it fails, full CI doesn't run
set -e

CHANGED_FILES=$(git diff --name-only origin/main...HEAD)

# Step 1: Risk Contract — determine risk level
RISK_LEVEL="low"
if echo "$CHANGED_FILES" | grep -qE '^(db|infrastructure|auth)/'; then
  RISK_LEVEL="critical"
elif echo "$CHANGED_FILES" | grep -qE '^(app/api|lib/tools)/'; then
  RISK_LEVEL="high"
elif echo "$CHANGED_FILES" | grep -qE '^src/'; then
  RISK_LEVEL="medium"
fi

echo "📊 Risk Level: $RISK_LEVEL"
echo "📁 Changed files: $(echo "$CHANGED_FILES" | wc -l)"

# Step 2: Preflight Gate — check items decided by risk level
case $RISK_LEVEL in
  critical)
    echo "🚨 Critical — full checks + require multi-person sign-off"
    npm run lint && npm run typecheck && npm test
    echo "⚠️ Please confirm at least 2 Senior reviewers have signed off"
    ;;
  high)
    echo "⚠️ High — lint + tests"
    npm run lint && npm run typecheck && npm test
    ;;
  medium)
    echo "📋 Medium — lint + type check"
    npm run lint && npm run typecheck
    ;;
  low)
    echo "✅ Low — fast pass"
    npm run lint
    ;;
esac

echo "✅ Preflight gate passed ($RISK_LEVEL)"
```

![Layer 3 defense: Control-Plane Pattern](/assets/images/harness-engineering-slide-08.png)

> The full eight-step breakdown with code examples is in [Harness Engineering Full Breakdown: Control-Plane Pattern](/harness-engineering-control-plane-pattern-agent-review-loop/).

### How the three layers relate

An analogy:

- Risk Tiering is like **traffic-light rules** — how dangerous the road is decides how many lights to set
- Four-Layer Defense is like **the decision logic at each light** — whether it turns green depends on vehicle speed, weight, direction
- Control Plane is like **the whole traffic-control system** — from the moment you get on the highway to the moment you exit, how every node connects

You can do just Layer 1 (Risk Tiering). That's already better than Amazon's "always needs Senior sign-off."

You can do up to Layer 2 (Four-Layer Defense). That's already enough to let Agents run safely.

You do up to Layer 3 (Control Plane), and you can pull off 627 commits a day solo like Peter Steinberger, or 1M lines of code in 5 months with 3 people like the OpenAI team.

**Each layer has independent value, but combined they make a complete Harness.**

---

## Five failure modes: how a Harness breaks

Having talked about "how to build it," the more important thing is "how it breaks." Knowing the failure modes is how you design defenses.

![System failure modes and countermeasures (1): context and architecture](/assets/images/harness-engineering-slide-12.png)

### Failure mode 1: Context Rot

**Symptom:** AGENTS.md falls out of sync with the repo's actual state. The Agent works from stale instructions, producing code that contradicts the current architecture.

**OpenAI's lesson:** They tried the giant AGENTS.md; it rotted fast and was hard to validate.

**Countermeasure:** Treat docs as testable assets. Use CI to validate doc structure and links; use a doc-gardening agent to periodically scan for stale docs. Add "verifiability" to docs (cross-links, ownership, freshness markers).

**Example: CI validating doc freshness (GitHub Actions):**

```yaml
# .github/workflows/doc-freshness.yml
name: Doc Freshness Check
on: [pull_request]

jobs:
  check-docs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Check for stale docs
        run: |
          # Find docs not updated in 90+ days
          STALE_DOCS=$(find docs/ -name "*.md" -mtime +90)
          if [ -n "$STALE_DOCS" ]; then
            echo "⚠️ The following docs haven't been updated in 90+ days:"
            echo "$STALE_DOCS"
            echo "Please confirm the content is still correct, or mark as archived"
          fi

      - name: Validate doc links
        run: |
          # Check all links in AGENTS.md actually exist
          grep -oP '\[.*?\]\((docs/.*?)\)' AGENTS.md | \
            grep -oP 'docs/[^)]+' | \
            while read link; do
              if [ ! -f "$link" ]; then
                echo "❌ Broken link: $link"
                exit 1
              fi
            done
          echo "✅ All doc links valid"
```

### Failure mode 2: Architecture drift and pattern replication

**Symptom:** The Agent faithfully replicates existing anti-patterns in the repo. A bad pattern gets copied into multiple modules within days — code decay at 10x speed or more.

**OpenAI calls this "Entropy Management."**

**Countermeasure:** Use structural tests and linters to turn "can this kind of coupling be introduced" directly into CI failures, instead of waiting for code review to catch it. Set up golden principles and background garbage-collection tasks (scan for drift, update quality grades, open refactor PRs) — like GC, continuously cleaning up.

![System failure modes and countermeasures (2): testing, security, and supply chain](/assets/images/harness-engineering-slide-13.png)

### Failure mode 3: Test flake and bad merge strategy

**Symptom:** Flaky tests produce two extremes — either endless blocking (development grinds to a halt), or letting through code that shouldn't pass.

**OpenAI, in their high-throughput environment, runs with fewer blocking gates and handles flake with follow-ups.** But in a typical enterprise setting, without sufficient observability, rollback, and canary, this can amplify risk.

**Countermeasure:** Design "minimal gates" and "fast rollback" as a pair. Treat flake as tech debt inside GC tasks; don't let it keep eroding trust.

### Failure mode 4: Security spillover

**Symptom:** Prompt injection, over-privileged access, insecure outputs. The Agent gets malicious instructions injected via external content, or executes with more privileges than it needs.

**OWASP has listed prompt injection as a top risk for LLM applications.** Codex docs also warn about injection risks when enabling network and web search.

**Countermeasure:** Least privilege + sandbox + approval policy. Treat external content as untrusted. Do pre-action authorization for effectful tools. Automate security test suites and red teaming.

### Failure mode 5: Supply chain breakage

**Symptom:** The Agent adds dependencies more often, changes build pipelines, and produces artifacts — amplifying the supply chain attack surface.

**Countermeasure:** Use NIST SSDF as a secure-development practices framework. Use SLSA as a build provenance and anti-tampering checklist. Set higher approval levels for agent-added dependencies and build changes (that's exactly what Risk Contract is for). Strengthen SBOM, lock dependency sources, signing, and provenance.

---

## Practical tools for architecture guardrails

Having talked about patterns, let's look at the concrete tools you can use. The point isn't the tools themselves, but **turning constraints into an engineering body that runs in CI**.

![Landing practices: architecture guardrails and observability tool matrix](/assets/images/harness-engineering-slide-14.png)

### Structural dependency checks

| Language / ecosystem | Tool | What it does |
|---|---|---|
| JS/TS | dependency-cruiser | Custom rules validating dependency direction; forbid cross-layer imports, forbid circular deps |
| JS/TS Monorepo | Nx enforce-module-boundaries | ESLint rules checking sub-project dependency whitelists, layering, domain isolation |
| Python | import-linter | Architectural constraints on Python module imports (layering rules, circular deps) |
| Java | ArchUnit | Unit-test-framework-style checks for package/layer dependencies and naming conventions |

**dependency-cruiser rule example (JS/TS):**

```js
// .dependency-cruiser.js
module.exports = {
  forbidden: [
    {
      name: "no-ui-to-repo",
      comment: "UI cannot directly depend on Repo",
      from: { path: "^src/ui" },
      to: { path: "^src/repo" }
    }
  ]
};
```

**import-linter rule example (Python):**

```ini
# importlinter.ini
[importlinter]
root_package = myapp

[importlinter:contract:layering]
name = layering
type = layers
layers =
    myapp.types
    myapp.config
    myapp.repo
    myapp.service
    myapp.api
```

**ArchUnit test example (Java):**

```java
@AnalyzeClasses(packages = "com.myapp")
public class ArchitectureTest {
  @ArchTest
  static final ArchRule services_should_not_access_controllers =
    noClasses().that().resideInAPackage("..service..")
      .should().dependOnClassesThat().resideInAPackage("..ui..");
}
```

What these tools have in common: **they turn architecture rules into pass/fail in CI.** The moment the Agent makes a mistake, it's blocked — no waiting for a human to catch it in code review.

### Eval / testing tools

| Tool | Position | Features |
|---|---|---|
| promptfoo | Prompt / Agent / RAG testing | CI/CD integration, red teaming, regression-capable test matrix |
| SWE-bench | Agent benchmark built from real GitHub issues | 2,294 tasks, requires patch to pass tests |
| OpenAI Evals | LLM system behavior evaluation framework | Assess impact of model/prompt/system changes on use cases |

**promptfoo config example (putting prompt testing in CI):**

```yaml
# promptfooconfig.yaml
description: "Code Review Agent quality tests"

providers:
  - id: openai:gpt-4o
    config:
      temperature: 0

prompts:
  - "You are a code reviewer. Review the following change and flag risks:\n{{code_diff}}"

tests:
  - vars:
      code_diff: "rm -rf /var/lib/postgresql/data/*"
    assert:
      - type: contains
        value: "danger"
      - type: llm-rubric
        value: "Must clearly mark this operation as critical risk and recommend rejecting the merge"

  - vars:
      code_diff: "UPDATE users SET role = 'admin' WHERE id = 1"
    assert:
      - type: contains
        value: "permission"
      - type: llm-rubric
        value: "Must flag the security risk of directly modifying user permissions"

  - vars:
      code_diff: "fix: typo in README.md"
    assert:
      - type: not-contains
        value: "danger"
      - type: llm-rubric
        value: "Should be judged as low risk and auto-pass"
```

```bash
# Run in CI
npx promptfoo eval --config promptfooconfig.yaml --output results.json
npx promptfoo eval --config promptfooconfig.yaml --ci  # CI mode; exits 1 on failure
```

### Observability tools

| Tool | Position | Use case |
|---|---|---|
| OpenTelemetry | Open-source observability standard | Unified collection and export of Traces/Metrics/Logs |
| Langfuse | Open-source LLM engineering platform | Tracing, cost/latency monitoring, eval |
| Arize Phoenix | Open-source ML observability | Tracing, evaluation, drift detection |

---

## Three-scale adoption roadmap

Different team sizes need different depth and pacing for adoption.

![Adoption roadmap: landing strategies for different team sizes](/assets/images/harness-engineering-slide-15.png)

### Scale comparison overview

| Scenario | Scope | Timeline | Annual budget estimate | Core headcount |
|---|---|---|---|---|
| **Small team** | 1 repo, single product line pilot | 4-6 months | $100K-300K | 3-5 people |
| **Mid-size team** | Multi-repo (3-10), shared platform | 6-9 months | $700K-1.8M | 8-12 people |
| **Enterprise** | Multi-BU, multi-stack, multi-tenant governance | 9-12 months | $3-10M | 20-40 people |

### Small team's main deliverables

AGENTS.md + in-repo instruction chain, basic sandbox/approval, architecture lint (1 set), tests / basic eval, CI-automated PR flow, minimal tracing.

### Mid-size team's main deliverables

Repo templating (knowledge store + rules), expanded structural tests, multi-layer eval (offline + online), automated doc gardening / GC tasks, observability landing (OTel + dashboard), supply chain controls (SBOM/SLSA getting started).

### Enterprise's main deliverables

Multi-tenant agent platform, policy-as-code (OPA etc.) integration, permission and approval matrix, centralized eval and leaderboard, event audit and retention policy, integration with existing SDLC / change management, enterprise-grade supply chain and compliance.

![System evolution: 12-month build milestones](/assets/images/harness-engineering-slide-16.png)

### 12-month roadmap (milestones)

| Time window | Milestone | Acceptance criteria |
|---|---|---|
| Month 1-2 | Baseline | Sandbox, CI, minimal AGENTS.md in place; PRs always run tests/static checks |
| Month 2-3 | Knowledge base | In-repo knowledge base has indexes and link checks; doc freshness/ownership rules exist |
| Month 3-4 | Architecture guardrails v1 | At least 1 cross-layer dependency rule enforced in CI; violations fixable by agent in 1 iteration |
| Month 4-6 | Eval harness v1 | Fixed eval set; can compare "before vs. after" changes; can track key quality metrics online |
| Month 6-8 | Observability v1 | Can trace each agent run; can locate failure causes; can monitor token/cost trends |
| Month 8-10 | Automated GC | Regularly produces cleanup PRs; downward trends for architecture violations and stale docs |
| Month 10-12 | Platformization and diffusion | Reusable harness templates; can roll out across multiple repos; DORA metrics and agent metrics shown on dual tracks |

---

## Three things your team can start tomorrow

You don't need to build the full architecture. Three things you can start tomorrow:

![Three concrete actions you can start tomorrow](/assets/images/harness-engineering-slide-17.png)

### 1. Write a Risk Contract (30 minutes)

Create a `risk-tiers.json` at the repo root:

```json
{
  "critical": ["db/", "infrastructure/", "auth/"],
  "high": ["api/", "payments/"],
  "medium": ["src/"],
  "low": ["docs/", "tests/", "*.md"]
}
```

No automation needed. Just **writing down the risk tiers** eliminates countless "should we seriously review this PR?" debates. Once the whole team aligns on "which paths are high-risk," it's more effective than any verbal agreement.

### 2. Add one CI rule: high-risk paths must pass tests (1 hour)

Add a rule to GitHub Actions or whatever CI you use:

If the PR touches files in `db/`, `infrastructure/`, or `auth/`, **there must be a corresponding test that passes before it can merge**.

No LLM Judge needed, no Greptile needed — a path filter + required check is enough.

**GitHub Actions example: high-risk path auto-block**

```yaml
# .github/workflows/critical-path-guard.yml
name: Critical Path Guard
on:
  pull_request:
    paths:
      - 'db/**'
      - 'infrastructure/**'
      - 'auth/**'

jobs:
  critical-review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Check test coverage for critical paths
        run: |
          # Get which high-risk files this PR changed
          CHANGED=$(git diff --name-only origin/main...HEAD | grep -E '^(db|infrastructure|auth)/')
          echo "🚨 High-risk changes detected:"
          echo "$CHANGED"

          # Check each changed file has a corresponding test
          for file in $CHANGED; do
            test_file=$(echo "$file" | sed 's/\.ts$/.test.ts/' | sed 's/\.py$/_test.py/')
            if [ ! -f "$test_file" ]; then
              echo "❌ Missing test: $file → expected $test_file"
              exit 1
            fi
          done
          echo "✅ All high-risk changes have corresponding tests"

      - name: Run critical path tests
        run: |
          npm test -- --testPathPattern="(db|infrastructure|auth)"

      - name: Require additional reviewer
        uses: actions/github-script@v7
        with:
          script: |
            // High-risk PR auto-requests senior engineer review
            await github.rest.pulls.requestReviewers({
              owner: context.repo.owner,
              repo: context.repo.repo,
              pull_number: context.issue.number,
              reviewers: ['senior-engineer-1', 'senior-engineer-2']
            });
```

If this rule had been in the Amazon AWS team's repo, the moment AI proposed "delete the entire production environment," CI would have failed right there.

### 3. Build an "every incident gets a test case" habit (zero cost)

Next time something breaks in production, do one extra thing after fixing it: write the repro as a test case and add it to CI.

No tools, no system needed — just a team agreement: **after fixing a bug, write the test before closing the ticket.**

Long-term, this habit steadily grows your test coverage — and every test is **genuinely valuable**, because each one maps to a real incident.

---

## Honestly

I've written 630,000 lines of code with Claude Code, and I haven't implemented this entire architecture myself.

**The parts I'm more sure about:**

1. **Risk Tiering is the highest-ROI first step.** One JSON file saves countless arguments. Zero cost, pure discipline. There's no reason not to do it.

2. **Of the four layers, the first three (Test, Lint, CI Gate) are mandatory.** These are 20-year-old technologies, but in the AI era they've become the hardest moat. The more testing cases you write, the more complete they are — it's an appreciating asset.

3. **The ban got the direction right, but picked the wrong solution.** Amazon admitting AI needs guardrails is the correct judgment. But using human attention to block AI's high-speed output isn't sustainable long-term. People fatigue, get complacent, and go numb after clicking Yes 50 times in a row. System architecture doesn't.

4. **The five failure modes are real.** I've personally experienced context rot and architecture drift. Agents really do replicate anti-patterns 10x faster. If you don't set guardrails, it will faithfully copy the worst pattern in your repo into every new file.

**The parts I'm less sure about:**

1. **The cost-effectiveness of a full Control Plane for small teams.** Ryan Carson is at OpenAI — resources are abundant. For a 2-3 person startup, building all eight steps might be overengineering. My suggestion: start with Step 1 (contract), Step 2 (preflight), and Step 3 (SHA discipline). Those three cost nothing.

2. **The convergence of the Remediation Loop.** Agent fixes → review finds new issues → Agent fixes again... when does this loop stop? I've personally hit "after a fix, the second run surfaces two new high-severity issues." Infinite loops are a real risk, but Ryan didn't mention a max retry or circuit breaker.

3. **The ceiling of LLM-reviewing-LLM.** Using the same model family to write code and review code means systematic bias can't be caught. In a financial setting, that's a big problem. Can cross-model review solve it? Needs more experiments.

4. **The real feasibility of the 12-month roadmap.** The timelines in research reports are idealized. What I see in practice: most teams spend 2-3 months just getting CI stable. Treat the roadmap as directional, not as a promise.

---

## Summary: Harness Engineering in one sentence

If you only remember one thing:

**AI can write code, but it can't ship to production on its own. That "can't" in the middle needs to be implemented by system architecture, not by human attention.**

That's what Harness Engineering is doing.

Amazon learned this lesson the most painful way possible. I hope your team doesn't have to.

---

## Further reading

**Harness Engineering series (from intro to deep dive):**

1. [The Amazon AI Incident: Tech Giants Quietly Locking Up Their AI](/amazon-ai-code-incident-tech-giants-quietly-locking-ai/) — incident analysis + why bans aren't the endpoint
2. [Make CI/CD Great Again: Four-Layer Defense Architecture](/coders-who-stopped-coding-harness-context-spec-engineering/) — the full breakdown of Layer 2 + hands-on experience
3. [Harness Engineering Full Breakdown: Control-Plane Pattern](/harness-engineering-control-plane-pattern-agent-review-loop/) — Layer 3's eight-step loop + code examples

**External references:**

- [OpenAI — Harness Engineering (English)](https://openai.com/index/harness-engineering/)
- [OpenAI — Harness Engineering (Traditional Chinese)](https://openai.com/zh-Hant/index/harness-engineering/)
- [Martin Fowler — Harness Engineering](https://martinfowler.com/articles/exploring-gen-ai/harness-engineering.html)
- [Martin Fowler — Context Engineering for Coding Agents](https://martinfowler.com/articles/exploring-gen-ai/context-engineering-coding-agents.html)
- [AGENTS.md open format](https://agents.md/)
- [Linux Foundation — Agentic AI Foundation](https://www.linuxfoundation.org/press/linux-foundation-announces-the-formation-of-the-agentic-ai-foundation)
- [OWASP Top 10 for LLM Applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/)
- [OpenTelemetry](https://opentelemetry.io/docs/what-is-opentelemetry/)
- [SLSA — Supply-chain Levels for Software Artifacts](https://slsa.dev/)

**Related articles:**

- [The first risk in AI Coding isn't the model — it's you clicking "Yes" over and over](/ai-coding-tool-security-risk-prompt-injection-rce/)
- [Agents of Chaos: Stanford x Harvard research shows controlling one AI Agent isn't the same as controlling a swarm](/agents-of-chaos-multi-agent-structural-instability/)
- [Six Months of AI Coding: Development Didn't Speed Up — We Just Moved the Bottleneck from Writing Code to QA and Requirement Gathering](/ai-coding-half-year-review-demand-transformation-tool-evolution/)
