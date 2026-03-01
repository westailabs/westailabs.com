---
layout: research
title: "The Agentic Economy and the Prompt Injection Arms Race"
date: 2026-02-27
last_updated: 2026-02-27
tags: [security, prompt-injection, agentic-commerce, economics]
excerpt: "Visa, Mastercard, Stripe, and Google built payment-capable agent infrastructure in 6 months. The injection problem is completely unsolved. Here's why that matters."
status: published
permalink: /research/agentic-economy-injection-arms-race/
---

*Research by West AI Labs | February 2026*

## 1. The AI Agent Economy: Systemic Risk Nobody Modeled

### The Citrini Scenario

On Feb 22, 2026, Citrini Research published an analysis imagining a "report from 2028" in which widespread agent deployment triggered a self-reinforcing economic collapse:

> AI capabilities improved → companies needed fewer workers → white collar layoffs increased → displaced workers spent less → margin pressure pushed firms to invest more in AI → AI capabilities improved…

**"It was a negative feedback loop with no natural brake. The system turned out to be one long daisy chain of correlated bets on white-collar productivity growth."**

This isn't the Skynet scenario. It's the gradual unspooling of the economy's internal transaction structure. The critical pivot: when AI agents replace third-party contractors (rather than just in-house employees), any B2B optimization relationship becomes vulnerable. The "Death of SaaS" scenario extends to all outsourced work.

The scenario is explicitly labeled a *possibility*, not a prediction. But critics couldn't point to where it goes wrong — which is telling.

### BIS Working Paper (January 2026)

The Bank for International Settlements published a working paper examining how AI agents could affect **monetary policy transmission**. Key finding: if millions of agents can adjust prices in real time in response to interest rate changes, the traditional lag between Fed action and economic response collapses.

Implication: central banks have operated for decades on models that assume price stickiness. Agents eliminate stickiness by design. Monetary policy tools built on old dynamics may simply stop working.

---

## 2. The Agentic Commerce Infrastructure Buildout

### What Was Built (April–September 2025)

In a six-month window, every major payment network launched agentic commerce platforms:

| Provider | Launch |
|----------|--------|
| Visa | "Intelligent Commerce" — agent transactions with spending controls |
| Mastercard | "Agent Pay" — Agentic Tokens combining identity + spending limits + audit trail |
| PayPal | Agent Toolkit (GitHub OSS) |
| Stripe + OpenAI | Agentic Commerce Protocol — add-to-cart and checkout from ChatGPT |
| Google | Universal Commerce Protocol — backed by Target, Walmart, Shopify |

The architectural pattern is identical across all of them: **agents need a single credential combining "who," "whose money," and "can pay"** — executed atomically without requiring humans in the loop or creating merchant accounts.

McKinsey projects $3–5T in agentic commerce revenue by 2030. Morgan Stanley is more conservative ($190–385B). Bain splits the difference ($300–500B). Whatever the number, the infrastructure is clearly being built for it.

### Consumer Trust Gap

61% of consumers trust agents with purchases under $20. That drops to 39% for larger amounts. The trust gap is the security gap — consumers won't accept full autonomy without confidence that agents can't be manipulated into unauthorized purchases.

### Security Implications

This is where agentic commerce and prompt injection intersect most dangerously. If a compromised agent with a Mastercard Agentic Token can be injected via a malicious webpage to redirect a transaction, the attack vector goes from "steal data" to "steal money." The audit trail Mastercard builds in is the right instinct — but it's detective, not preventive.

The trust problem is architectural: agents holding payment credentials are **high-value targets for exactly the injection techniques being actively developed**. If Trail of Bits found 4 techniques to exfiltrate Gmail from a browser AI just by having a user visit an attacker's page — the same class of attacks against payment agents is a direct path to real financial fraud.

---

## 3. Prompt Injection: New Developments

### ChatGPT Lockdown Mode (Feb 17, 2026)

OpenAI shipped "Lockdown Mode" — the first major commercial first-party structural isolation feature. Key characteristics:

- **Restricts ChatGPT's interactions with external systems and data**
- Limits which tools and capabilities are accessible when enabled
- Adds **"Elevated Risk labels"** to warn users of risky AI tools and content
- Targets high-risk users: execs, security pros, healthcare, enterprise

This is notable not because it's technically revolutionary — it's essentially "reduce your attack surface by limiting tool access." But it's the first time a major provider has acknowledged the structural problem publicly and shipped a toggle for it. The framing is important: *"you can't make the AI safe by asking it to be safe"* is now implicit in the product.

The limitation: it's opt-in, enterprise-only, and reduces functionality. Most users will leave Lockdown Mode off, creating a two-tier security model where the most targeted users (executives) have to sacrifice capabilities to get protection. That's not a long-term solution.

### Comet Browser Audit (Trail of Bits, Published Feb 2026)

Trail of Bits audited Perplexity's Comet browser before launch. They developed **four prompt injection techniques** — all successfully exfiltrating user Gmail data when the user simply visited an attacker-controlled page.

**Threat model (TRAIL framework):**
- Two trust zones: user's local machine (browser data, cookies, authenticated sessions) vs. Perplexity's servers
- Agent tools (fetch URL, browse history, control browser) create **data paths between zones**
- These data paths are the attack surface

**The four techniques** were not individually named in the public excerpt, but the key finding was:
1. They were *combinable* — layering two or more techniques dramatically increased success rate
2. The root vulnerability was consistent: **external content was not treated as untrusted input**
3. All four techniques shared the same exfiltration goal: read Gmail, send to attacker's server

This is the most concrete technical demonstration to date of the threat model. A real browser, real users, real Gmail access, real data exfiltration — not theoretical. Perplexity engaged Trail of Bits *proactively*, which is the right move. The question is whether most browser AI developers do the same before launch.

### Lasso Security Prompt Injection Taxonomy

Lasso Security published a standardization guide distinguishing **technique** from **intent** in prompt injection attacks.

**The core framework:**

| Concept | Definition |
|---------|-----------|
| System Prompt | Core behavioral instructions |
| Refusal Space | Semantic space where model refuses |
| Intent | What the attacker wants to accomplish |
| Technique | How the attack is delivered (intent-agnostic) |

**Two primary intents:**
1. **System Prompt Leakage** — extracting hidden instructions, business logic, proprietary rules
2. **Jailbreak** — bypassing safety controls for responses the model would normally refuse

**Key insight:** Neither intent is a technique. You can pursue either objective with any of the following techniques:
- **Encoding/obfuscation** — hiding malicious instructions in encoded or reformatted text
- **Role-playing** — asking the model to act as a persona without the original constraints
- **Context manipulation** — framing the attack as an extension of a legitimate context
- **Instruction smuggling** — hiding instructions within data that the model treats as commands

**Evolution:** Direct attacks ("ignore all previous instructions") now mostly fail against modern LLMs. Attackers have shifted to subtler indirect approaches — misdirection, abstraction, contextual framing. Techniques can be layered, compounding effectiveness.

---

## Synthesis: The Economic Stakes of the Injection Problem

The three threads connect in a way that hasn't been widely recognized:

1. **Agentic commerce infrastructure is being built at massive scale** — Visa, Mastercard, PayPal, Stripe, Google, OpenAI all in 6 months.

2. **These agents will hold credentials with real financial authority** — payment tokens, spending limits, transaction capability.

3. **The injection problem is completely unsolved** — Trail of Bits found 4 techniques against a well-funded product (Perplexity) with proactive security investment.

The logical conclusion: within 12-18 months, there will be large-scale financial losses attributed to prompt injection attacks against payment-capable agents. Not theoretical. Real losses.

The market for agent security isn't just "protect your data." It's "protect your wallet." The dollar figure creates regulatory attention that data breaches alone don't. GDPR emerged from data loss. SOX emerged from financial fraud. The regulatory framework for agentic financial security doesn't exist yet — **and it will be written in the wake of a major incident**.

---

## Defensive Takeaways

1. **For agent builders now:** Adopt TRAIL-style threat modeling. Map every tool to its trust zones. Treat any tool that reads external content as a potential injection vector.

2. **For payment-capable agents:** The Mastercard Agentic Tokens audit-trail approach is table stakes. But the real defense is structural isolation between the agent's instruction space and any content it fetches.

3. **For Lockdown Mode skeptics:** The feature is right but the UX is wrong. Security shouldn't require giving up functionality. The answer is context-aware isolation — restrict tool access when operating on untrusted content, not globally.

4. **For the taxonomy:** When evaluating any injection defense, test against both intents (leakage AND jailbreak) and at least encoding, role-playing, and context manipulation techniques. Most defenses are built against direct attacks that attackers no longer use.

---

*This research is maintained as a living document. Last updated: February 27, 2026.*
