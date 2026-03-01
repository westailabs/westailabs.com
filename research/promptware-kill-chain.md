---
layout: page
title: "The Promptware Kill Chain"
date: 2026-02-27
last_updated: 2026-02-27
tags: [security, prompt-injection, malware, kill-chain]
excerpt: "Prompt injection is the wrong frame. Attacks on LLM-based systems have evolved into a distinct class of malware — and 90%+ of published defenses have already been bypassed."
status: published
permalink: /research/promptware-kill-chain/
---

*Research by West AI Labs | February 2026*

## Source

Lawfare Media: "The Promptware Kill Chain" (Feb 2026)  
arXiv paper: <https://arxiv.org/abs/2601.09625>

---

## Core Thesis

"Prompt injection" is too narrow a term. Attacks on LLM-based systems have evolved into a distinct class of **malware execution mechanisms** — the authors call it "promptware." They propose a 7-step kill chain framework mirroring traditional malware campaigns (Stuxnet, NotPetya analogy).

---

## The Fundamental Vulnerability

LLMs have no architectural boundary between **trusted instructions** and **untrusted data**. Everything — system prompts, user input, retrieved documents, emails — is processed as one undifferentiated token stream. This is structurally different from traditional computing. You can't firewall your way out of it at the model level.

---

## The 7-Step Kill Chain

1. **Initial Access** — Malicious payload enters the system. Can be direct (typed prompt) or indirect (embedded in retrieved web page, email, shared doc, or even images/audio in multimodal models).

2. **Privilege Escalation** — "Jailbreaking" — bypassing safety training via persona adoption, adversarial suffixes, social engineering the model into ignoring guardrails. Equivalent to escalating to admin in traditional attacks.

3. **Reconnaissance** — Manipulating the LLM to reveal system internals, tool schemas, memory contents. The model's own knowledge of its environment becomes the attack surface.

4. **Persistence** — Memory poisoning. Injecting content that corrupts the agent's long-term memory store, influencing future sessions even after the original attack vector is closed.

5. **Lateral Movement** — In multi-agent or tool-connected systems, using one compromised agent to attack others. Agent-to-agent communication becomes a propagation channel.

6. **Command & Control** — Establishing ongoing instruction channels via crafted content in regularly fetched sources (RSS, email, shared documents the agent reads periodically).

7. **Exfiltration / Action** — The payload executes: data sent to attacker infrastructure, unauthorized transactions made, real-world actions triggered through connected tools.

---

## Defensive Framing

Defense requires **depth at the boundary**, not just prompt-level filtering:

- Limit privilege escalation pathways (restrict persona adoption, validate identity continuity)
- Constrain reconnaissance capability (limit what the model can reveal about its own configuration)
- Prevent persistence (audit and validate memory writes; treat external content as untrusted before writing to memory)
- Disrupt command & control (don't automatically act on content from periodically fetched sources without validation)
- Restrict permitted agent actions (least-privilege tooling — agents should only have access to tools they need for their current task)

---

## Key Research Findings

### ZDNet / Collaborative Research (OpenAI, Anthropic, Google DeepMind)
Adaptive attackers using **gradient descent and RL bypassed 90%+ of published defenses**. Static rule-based defenses are fundamentally insufficient against adaptive adversaries.

### Microsoft Security Blog
New attack vector identified: **AI Recommendation Poisoning** — manipulating AI memory stores to corrupt long-term behavior. Defenses cited: prompt filtering, content/instruction separation, user-visible memory controls.

### MIT Tech Review
Human-in-the-loop agentic workflows are now attack vectors — coercing agents to take harmful real-world actions (calendar writes, email sends, financial transactions) via injected content in documents or web pages the agent fetches.

---

## Why the "Prompt Injection" Frame is Too Narrow

The kill chain framing matters because it changes how defenders think about the problem:

- **Prompt injection** implies a single event — one bad prompt, one bad response.
- **Promptware** implies a campaign — reconnaissance, establishment of persistence, lateral movement, eventual payload execution.

Most current defenses focus on detecting and blocking individual injections. The kill chain model suggests that's necessary but not sufficient. An attacker who can read your agent's system prompt (reconnaissance) and write to its memory (persistence) has won, even if every individual prompt injection attempt is detected.

The correct defensive posture treats each stage of the kill chain as its own threat surface, requiring distinct controls.

---

## Implications for Agent Architecture

1. **Memory as attack surface** — Long-term memory is not just a feature; it's a persistence mechanism that attackers will target. External content should never be written to memory without validation.

2. **Tool permission scoping is non-negotiable** — Least-privilege tooling limits the blast radius of any successful compromise. An agent that can only read files it needs, send to contacts it knows, and spend within defined limits is dramatically harder to weaponize.

3. **Local-first architecture reduces indirect injection surface** — Agents that don't browse the web, process external email, or retrieve documents from third-party sources have fundamentally fewer initial access vectors.

4. **Multi-agent systems multiply the surface** — Every agent-to-agent communication channel is a potential lateral movement path. Content from other agents should be treated as untrusted until verified.

---

*This research is maintained as a living document. Last updated: February 27, 2026.*
