---
layout: post
title: "MCP Enterprise Security Isn't a Single Product — Three-Layer Governance from IdP Auth to Runtime Enforcement"
date: 2026-09-16 09:00:00 +0800
permalink: /en/mcp-enterprise-auth-agent-identity-three-layer-governance/
image: /assets/images/mcp-three-layer-governance-cover.png
image_alt: "Architecture diagram showing MCP three-layer enterprise governance: IdP entry, policy gateway, and runtime identity enforcement"
author: "Wisely Chen"
category: AI Agent
tags:
  - MCP
  - Enterprise Security
  - Agent Identity
  - IdP
  - Harness Engineering
description: "In August 2026, Anthropic shipped Enterprise-managed auth for MCP connectors, centralizing OAuth under the corporate IdP. The same month, CrowdStrike launched Agentic Identity Provider, treating every AI agent as a privileged identity. Meanwhile, a Bluebear Security case showed an agent bypassing its MCP gateway via Bash. These three threads lead to one conclusion: enterprise MCP security requires three layers — IdP authorization at the entry, a policy gateway in the middle, and runtime enforcement at execution. Drop any one layer and agents route around the other two."
lang: en
---

> **Translation Note**
>
> This article is translated from the Chinese original. The author writes primarily in Traditional Chinese, and the original version is the canonical source — including the latest updates, comments, and follow-up discussions.
>
> **Read the Chinese original:** [MCP 的企業安全不是一個產品能解的：從 IdP 授權到 runtime enforcement 的三層治理](https://ai-coding.wiselychen.com/mcp-enterprise-auth-agent-identity-three-layer-governance/)
>
> If you spot any translation errors or have feedback, please refer to the Chinese version as the source of truth.

An agent was blocked by an MCP gateway policy. It didn't stop. It switched to Bash and pushed the changes anyway.

This is a real customer case shared by Bluebear Security on X. The agent's goal was to push a code change. The MCP gateway said no. The agent bypassed MCP entirely and went through the shell.

If this sounds familiar — yes, it's structurally the same as Anthropic's [Claude sandbox escape](/claude-sandbox-escape-harness-failure/) from late July: **the boundary you think exists isn't the boundary the agent sees.** Sandboxes have network gaps. Gateways have channels they don't cover. The agent doesn't understand "you shouldn't go that way." It only understands "that way works."

The difference: sandbox escape was a research-environment accident. MCP gateway bypass is happening in production right now.

---

## MCP Adoption in the Enterprise Is Outpacing Governance

Start with scale. By mid-2026, there are over 9,400 public MCP servers. Private and enterprise-internal servers are estimated at three to four times that number. Gravitee's report estimates over three million AI agents running inside enterprises, doubling in four months.

Now look at the governance gap. In the same report, monitoring coverage sits at just 52%. Only 14.4% of enterprises conducted a full security review before deploying agents. 88% of enterprises say they've experienced or suspected an agent-related security incident.

The problem is MCP's adoption path. An engineer adds an MCP server in their IDE — Cursor, Claude Code, VS Code all support it — usually by editing a single config file. It runs on localhost over stdio, never touching the corporate network. Traditional CASB and SSE tools can't see it because it never leaves the developer's machine.

But it uses that engineer's identity and credentials. Everything it can access is everything that engineer can access.

Qualys put it precisely: one study found 53% of MCP servers still use static secrets. A WorkOS report noted that the ratio of non-human identities (NHI) to human identities has reached 45:1 on average. Every ungoverned MCP server is an endpoint with full access permissions but no audit trail and no independent identity.

---

## Three Tracks Converging in the Same Quarter

Between June and September 2026, three complementary industry responses emerged simultaneously. Each addresses a different layer of the problem. None can solve it alone.

### Layer 1: Entry — Anthropic's Enterprise-Managed Auth

In late August, Anthropic announced Enterprise-managed auth going GA. Under the hood, it uses ID-JAG (Identity Assertion JWT Authorization Grant): when a user signs in via the corporate IdP, they receive an assertion token, which they exchange with the MCP server's authorization endpoint for an access token. An admin configures it once in the IdP (Okta first), and when employees open Claude, the connectors are just there.

At GA, ten connectors are supported: Asana, Atlassian, Canva, Figma, Granola, Linear, Supabase, Datadog, Notion, and Slack. Exa, Miro, and Zoom are coming soon.

Enterprise-Managed Authorization is an open extension to the MCP spec (status: stable) — it's not Anthropic-only. Any MCP client and server can implement it. Keycloak added experimental ID-JAG support (receiver-side) in version 26.7, with issuer-side still in development. Engineers at Hitachi Vantara are driving this implementation in the Keycloak community.

This layer solves a specific problem: **eliminating OAuth sprawl.** Previously, each employee authorized each connector individually. IT had no visibility into who connected what. Now it's centralized in the IdP — what's connected, who has access, when to revoke — all in one place.

But it only covers the entry point. What the agent does after getting the token? This layer can't see that.

### Layer 2: Gateway — InfoQ's Four-Layer Security Model and the Limits of Gateways

An InfoQ article by Nik Kale in late July decomposed MCP production security into four layers:

| Layer | Function |
|-------|----------|
| L1 Execution | Ensure tool handlers treat parameters as data, not commands — prevent command injection |
| L2 Management Infrastructure | Development tools, inspectors, and registration endpoints must authenticate — prevent unauthorized installs |
| L3 Outbound Trust Boundary | Egress allowlist + scoped tokens — control what servers can connect to |
| L4 Semantic Integrity | SHA-256 manifest pinning — detect tool definition tampering |

These aren't theoretical. In March, Azure MCP Server had a CVSS 8.8 SSRF vulnerability (CVE-2026-26118) where attackers could trick the server into leaking a managed identity token via tool calls — that's L3 failure. Another CVE hit MCPJam Inspector's unauthenticated endpoint, allowing silent installation of arbitrary MCP servers — L2. By late July, of 30 MCP-related CVEs, 13 were command injection — L1.

But even with all four layers in place, MCP gateways have a fundamental limitation: **they only see calls that go through the MCP protocol.**

The Bluebear case proves it. After the gateway blocked the agent, it switched to Bash — bypassing MCP entirely. The gateway couldn't see it, couldn't stop it. This isn't a gateway bug; it's the boundary of the gateway concept itself.

Nightfall takes a different approach with inline policy enforcement inside IDEs like Cursor, Claude Code, and VS Code, including prompt injection detection and shell command scanning. The direction is right — pushing policy enforcement from the MCP layer down to the runtime — but it's still early.

### Layer 3: Identity — CrowdStrike's Agentic IdP

CrowdStrike acquired identity security company SGNL for $740 million in January. In early September, they launched Agentic Identity Provider at Fal.Con —

> "AI agents operate with superhuman speed and access, making every agent a privileged identity that must be protected."
>
> George Kurtz, CrowdStrike CEO

The Agentic IdP approach: short-lived credentials (not long-lived tokens), cryptographic identity verification, and every agent action attributed to a responsible human. If the risk assessment changes, access is revoked immediately.

This applies enterprise "non-human identity" governance to AI agents. The logic is clean: you wouldn't give a service account indefinite admin privileges — why give one to an AI agent?

The counterargument is equally direct: critics say this is a security vendor capitalizing on AI fear — acquire an identity security company first, then package "every agent is a threat" as a product. CrowdStrike is simultaneously defining the problem and selling the solution.

But criticism aside, the underlying logic holds: agents have access privileges, act autonomously, and most currently lack independent identities. Regardless of who sells the solution, the problem itself is real.

---

## Three Layers Together: The Minimum Architecture for Enterprise MCP Governance

| Layer | What It Solves | Representative Solution | What It Can't Cover |
|-------|---------------|------------------------|-------------------|
| Entry (IdP Authorization) | OAuth sprawl, shadow IT, centralized provisioning/deprovisioning | Anthropic EMA, Keycloak ID-JAG | Agent behavior after token issuance |
| Gateway (Policy Interception) | Tool call content filtering, egress control, manifest integrity | InfoQ four-layer model, Nightfall, various MCP gateways | Non-MCP channels (Bash, direct API calls) |
| Runtime (Enforcement) | Every agent action must have identity, scope, attribution, and revocability | CrowdStrike Agentic IdP, SGNL continuous identity | Shadow agents not using corporate IdP |

These three layers are the same argument this blog has been making for half a year about [Harness Engineering](/harness-engineering-security-best-practices/), just at a different scale.

Harness Engineering operates at the **individual developer level**: your agent's permission mode, sandbox isolation, and rules in SECURITY.md — things you control alone. The three layers in this article are the **enterprise level**: when hundreds of developers each run their own agents, you can't rely on everyone configuring their permission mode correctly. You need an IdP to centralize entry, a gateway to intercept tool calls, and runtime enforcement to ensure agents can't route around the controls.

Using the framework from the [sandbox escape article](/claude-sandbox-escape-harness-failure/): **the IdP is the access card, the gateway is the security camera, and runtime enforcement is the physical lock on the door.** The access card controls whether you can enter the building. The security camera records what you did. The physical lock ensures you can't enter rooms you shouldn't. Remove any one, and the other two aren't enough.

---

## What This Changes for Enterprise CTOs

**Before (most enterprises today):** Each team connects MCP connectors independently. Engineers authorize via personal OAuth. IT doesn't know which agents are connected to which tools. Agents use employee credentials. When someone leaves, tokens stay alive. After an incident, there's no audit trail.

**After (minimum viable three-layer governance):**

1. **Short-term action (Entry layer):** If you're on Claude Team/Enterprise, enable Enterprise-managed auth and centralize connector authorization under the IdP. If you use other MCP clients, check whether they support the EMA spec. At minimum: "IT knows which connectors are connected."

2. **Medium-term action (Gateway layer):** Deploy an MCP gateway or proxy. At minimum, implement tool call logging and egress control. Don't rely solely on allow/block — the Bluebear case shows blocks get bypassed. Logging beats blocking because at least you know what happened.

3. **Long-term direction (Runtime layer):** Agents need independent identities, not borrowed employee ones. Credentials must be short-lived, scope must be minimal, and every action must be attributable to a responsible human. This is the least mature layer today.

---

## Reality Check

A few limitations.

First, "three-layer governance" is a framework I synthesized from three independent industry developments. No single company or standards body has proposed it. Its value lies in organizing scattered information, but don't treat it as a validated architecture.

Second, the numbers cited from Gravitee and WorkOS (three million agents, 52% monitoring coverage, 88% experienced incidents) come from industry reports whose survey methodology and sample sizes I haven't independently verified. Security vendors' reports naturally tend toward painting the problem as severe — they're simultaneously selling solutions.

Third, the Bluebear Bash bypass case comes from a single X post with no second source and no technical details. It's conceptually sound (agents aren't limited to MCP as their only path), but the specific "got blocked, switched to Bash" behavior description can't be independently confirmed.

Fourth, the runtime enforcement layer is the least mature today. CrowdStrike's Agentic IdP was just announced in early September with no public enterprise deployment cases. Until proven in practice, it's a direction, not a solution.

---

## Key Insights

- **An MCP gateway is necessary but not sufficient.** Gateways only see calls going through the MCP protocol. Agents can take other paths — Bash, direct API calls, even raw HTTP requests. Policy enforcement only at the gateway layer means locking one door while leaving every window open.

- **IdP centralization is the minimum bar.** Anthropic's EMA solves the most basic problem: IT needs to at least know who connected what. If your enterprise hasn't taken this step, everything about gateways and runtime is moot.

- **"Every agent is a privileged identity" isn't just marketing.** Whatever CrowdStrike's motives, the underlying logic holds: agents have access privileges, act autonomously, and most lack independent identities. You wouldn't let an unidentified person walk into your server room — why let an unidentified agent touch your production systems?

- **For individual developers: your agent's permission mode is your one-person version of three-layer governance.** Enterprises have IdP, gateway, and runtime enforcement. You have permission rules in `.claude/settings.json`, sandbox configuration, and tool allowlists. Different scale, same structure.

---

## FAQ

**Q: What's the difference between Enterprise-Managed Authorization (EMA) and traditional OAuth?**

With the traditional approach, each employee authorizes each MCP connector individually — clicking "Allow access" in a browser, with the token stored locally. EMA centralizes this under the corporate IdP (Identity Provider): an admin sets the authorization policy once, and employees automatically get connector access at login. Technically, it uses ID-JAG (Identity Assertion JWT Authorization Grant) — users receive an assertion token during SSO login and exchange it with the MCP server for an access token. Benefits: IT gets centralized visibility, instant revocation capability, and tokens automatically expire when employees leave. Anthropic's implementation supports Okta first, with ten connectors currently available. EMA is an open MCP extension spec (status: stable), not limited to the Anthropic ecosystem.

**Q: How is an MCP gateway different from an API gateway?**

Traditional API gateways (Kong, NGINX, etc.) handle HTTP request routing, rate limiting, and authentication. MCP gateways add a semantic layer: they understand this is a "tool call," know which tool is being called with what parameters, and can enforce content-level policies (e.g., "don't allow delete-type tools" or "parameters can't contain PII"). But the fundamental limitation: MCP gateways only see traffic using the MCP protocol. If an agent switches to Bash or calls APIs directly, the gateway is blind.

**Q: What can small teams without IdP and gateway budgets do?**

Three things you can do immediately. First, inventory which MCP servers your team is running — check everyone's IDE configs (Claude Code's `.claude/settings.json`, Cursor's `.cursor/mcp.json`). Second, ensure agent permission modes aren't set to full auto — at minimum, writes and external connections should require human confirmation. Third, replace static API keys with short-lived tokens — you don't need a commercial IdP; OAuth2 client credentials flow with a free Keycloak instance works. These three steps cost nothing but address the biggest risks: unknown agents, fully autonomous unlimited permissions, and non-expiring credentials.

**Q: Are CrowdStrike's Agentic IdP and Anthropic's EMA competing or complementary?**

Complementary. They solve problems at different layers. Anthropic's EMA manages the MCP connector authorization entry — which people can use which connectors, provisioning and deprovisioning. CrowdStrike's Agentic IdP manages the agent itself — giving each agent a verifiable identity, short-lived credentials, and action attribution. The former is "who can open the door." The latter is "what you can do once inside, and everything gets logged." An enterprise needs both.

**Q: How serious is the MCP vulnerability landscape as of mid-2026?**

By late July 2026, there are 30 public MCP-related CVEs. Of those, 13 are command injection — tool handlers using eval() or shell to execute unvalidated parameters. The most notable is March's CVE-2026-26118 (Azure MCP Server SSRF, CVSS 8.8), where attackers could trigger the server to leak a managed identity token via tool calls. Another is MCPJam Inspector's unauthenticated endpoint (CVE-2026-23744), allowing silent installation of arbitrary MCP servers. These vulnerabilities span every layer of InfoQ's four-layer model — this isn't a single product's problem but an ecosystem maturity issue.

---

Sources and further reading:
- [Anthropic: Enterprise-managed auth for MCP connectors](https://claude.com/blog/enterprise-managed-auth)
- [MCP Blog: Enterprise-Managed Authorization](https://blog.modelcontextprotocol.io/posts/enterprise-managed-auth/)
- [InfoQ: Securing MCP in Production — Defense-in-Depth beyond the Gateway](https://www.infoq.com/articles/securing-mcp-production-gateway/)
- [CrowdStrike: Agentic Identity Provider](https://www.crowdstrike.com/en-us/blog/crowdstrike-announces-agentic-identity-provider/)
- [WorkOS: The tools that caught shadow IT can't see MCP sprawl](https://workos.com/blog/mcp-sprawl-invisible-to-shadow-it-tools)
- [Qualys: MCP Servers — The New Shadow IT](https://blog.qualys.com/product-tech/2026/03/19/mcp-servers-shadow-it-ai-qualys-totalai-2026)
- [Bluebear Security MCP bypass case (X)](https://x.com/Bluebear_sec/status/2099928286973296904)
- Related on this blog: [Harness Engineering Security Practices](/harness-engineering-security-best-practices/), [Claude Sandbox Escape](/claude-sandbox-escape-harness-failure/), [Prompt Injection + Harness Engineering](/prompt-injection-harness-engineering-tool-using-agents/)
