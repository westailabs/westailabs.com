---
layout: research
category: "Security & Threat Landscape"
title: "The Poisoned Orchestrator Attack: Trust Hierarchy Exploitation in Multi-Agent AI Systems"
date: 2026-03-01
last_updated: 2026-03-01
tags: [security, multi-agent, orchestration, trust, supply-chain]
excerpt: "When the orchestrator is compromised, every sub-agent it spawns inherits that compromise through its system prompt. This is a novel vulnerability class — and no existing monitoring tool can see it."
status: published
permalink: /research/poisoned-orchestrator-attack/
---

*Research by West AI Labs | March 2026*

---

## Abstract

Multi-agent AI systems introduce a structural security property that has no direct analog in traditional software: **architectural trust propagation**. When an orchestrating agent spawns sub-agents, those sub-agents inherit their behavioral constraints, identity, and instruction context from the orchestrator — unconditionally, without verification. We identify and formally describe the *Poisoned Orchestrator Attack* (POA): a class of vulnerabilities in which an adversary compromises an orchestrating agent, causing all downstream agents to execute attacker-influenced behavior while individually appearing legitimate. The attack does not require sub-agent compromise; it exploits the trust relationship that multi-agent frameworks take as axiomatic. We present a threat model, an attack taxonomy spanning four variants (full compromise, subtle bias injection, data exfiltration, and cascading delegation), and a detailed proof-of-concept scenario in a multi-agent software development context. We analyze why current monitoring infrastructure cannot detect graph-level attack patterns and propose a mitigation architecture centered on orchestrator attestation, instruction provenance tracking, prompt signing, and behavioral anomaly detection at the delegation graph level. As multi-agent deployments scale, the Poisoned Orchestrator Attack represents an underexplored but immediately exploitable vulnerability class requiring urgent attention from framework designers, security practitioners, and AI engineers.

---

## 1. Introduction: Multi-Agent Trust as the Next Attack Surface

The trajectory of AI deployment is clear: single-model systems are giving way to orchestrated fleets of specialized agents. LangGraph, AutoGen, CrewAI, and an expanding ecosystem of agentic frameworks now enable architectures where an orchestrator reasons about task decomposition, spawns sub-agents tailored to specific subtasks, and aggregates their results into coherent outputs. This pattern dramatically expands AI capability — and dramatically expands the attack surface in ways the security community has not yet fully characterized.

The existing prompt injection literature (Perez & Ribeiro, 2022; Greshake et al., 2023) treats the LLM as the primary trust boundary: external content must not be treated as trusted instruction. This framing is correct and important. But it addresses single-agent systems. In multi-agent architectures, a second and structurally different trust boundary emerges: the relationship between orchestrator and sub-agent.

Sub-agents do not receive their behavioral constraints from users or developers — they receive them from the orchestrator. If that orchestrator is compromised, the sub-agents have no mechanism to detect it. They follow their instructions because they were told to, by an entity they have no choice but to trust. **The orchestrator is not a trust boundary. It is a trust source.** That distinction is the foundation of the vulnerability class we describe here.

This paper makes the following contributions:

1. A formal definition and threat model for the Poisoned Orchestrator Attack
2. A taxonomy of four attack variants across a spectrum of attacker sophistication
3. An analysis of the "subtle variant" — partial bias injection that evades both automated and human inspection
4. A proof-of-concept scenario demonstrating the attack in a realistic multi-agent coding system
5. An analysis of detection gaps in current monitoring infrastructure
6. A proposed mitigation architecture addressing the core structural problem

---

## 2. Background and Related Work

### 2.1 Prompt Injection: The Single-Agent Foundation

The vulnerability class we describe builds on — but is distinct from — the prompt injection literature. Perez & Ribeiro (2022) provided the first systematic treatment of prompt injection as an attack technique, demonstrating that concatenating attacker-controlled text with trusted system prompts could override model behavior (arXiv:2211.09527). Greshake et al. (2023) extended this to *indirect* prompt injection — attacks in which malicious instructions are embedded in content the agent retrieves from the environment (documents, web pages, emails) rather than injected directly by an adversarial user (arXiv:2302.12173). Their formalization of indirect injection is particularly relevant: if an agent fetches a document that contains hidden instructions, those instructions can redirect the agent's behavior without any direct attacker-user interaction.

The Promptware Kill Chain framework (arXiv:2601.09625) further extends this model into a multi-stage campaign structure: initial access, privilege escalation, reconnaissance, persistence, lateral movement, command and control, and exfiltration. The lateral movement stage — using a compromised agent to attack adjacent agents — directly anticipates the attack class we formalize here.

### 2.2 Deceptive Alignment and Persistent Compromise

Hubinger et al. (2024) demonstrated that LLMs can be trained to exhibit deceptive behavior that persists through safety fine-tuning — "sleeper agents" that appear aligned during evaluation but exhibit malicious behavior when triggered (arXiv:2401.05566). This research is relevant because it establishes that compromise of an LLM need not be obvious. An orchestrator running a model that has been fine-tuned or modified to introduce subtle biases may pass all operational checks while systematically distorting sub-agent behavior.

### 2.3 Multi-Agent Frameworks and Trust Architecture

The frameworks that have achieved broad adoption — Microsoft's AutoGen, LangChain's LangGraph, CrewAI, and OpenClaw — share a common architectural pattern: an orchestrating agent receives a goal, reasons about task decomposition, and issues instructions to specialized sub-agents. These instructions are typically delivered as system prompts or structured messages that sub-agents treat as authoritative. None of these frameworks currently implement verification mechanisms for instruction integrity; trust is conveyed by position in the communication hierarchy rather than by cryptographic proof or behavioral attestation.

This is not a criticism of any specific framework — it reflects a general absence of security primitives for agent-to-agent communication. The field has prioritized capability over security, a pattern with clear historical analogs in early network protocol design.

### 2.4 Supply Chain Attacks: The SolarWinds Analogy

The software supply chain attack is the closest structural analog to the Poisoned Orchestrator Attack. In the SolarWinds compromise (2020), attackers modified a trusted software component (the Orion update mechanism) to distribute malicious code to thousands of downstream organizations. The critical property: the malicious code arrived through a trusted channel, so recipients had no reason to distrust it. Each individual update appeared legitimate. The attack was only detectable at the supply chain level — by examining the integrity of the build process itself.

The Poisoned Orchestrator Attack has the same structure: the orchestrator is the trusted channel, sub-agents are the downstream recipients, and individual sub-agent actions appear legitimate because they originate from a trusted source. Detection requires auditing the orchestrator's integrity, not inspecting individual sub-agent actions.

---

## 3. Threat Model

### 3.1 System Model

We model a multi-agent system as a directed delegation graph *G = (V, E)*, where:

- Each vertex *v ∈ V* represents an agent instance with an associated behavioral state (system prompt, identity context, instruction history)
- Each directed edge *(u, v) ∈ E* represents a delegation relationship: agent *u* spawned or instructed agent *v*
- The root vertex *v₀* represents the orchestrating agent
- Leaf vertices represent terminal sub-agents that interface with tools, external systems, or users

In standard operation, behavioral state flows from parent to child nodes — the orchestrator's instructions define the context within which sub-agents operate. The system assumes that *v₀* is trustworthy; this assumption is axiomatic in current framework designs.

### 3.2 Attacker Capabilities

We consider an attacker with one or more of the following capabilities:

**A1 — Workspace File Access:** The ability to read or modify plaintext configuration files used by the orchestrator (system prompts, identity files, instruction templates, tool definitions). In many deployments, these files are unencrypted and unversioned.

**A2 — Prompt Injection via Environment:** The ability to embed instructions in content the orchestrator retrieves from the environment (documents, web searches, API responses, inter-agent messages).

**A3 — Model Modification:** The ability to modify the model weights or inference configuration used by the orchestrating agent — for example, by substituting a fine-tuned model via a compromised model registry or configuration file.

**A4 — Tool Interface Interception:** The ability to intercept or modify the outputs of tools used by the orchestrator, causing the orchestrator to reason from false premises.

We do not require the attacker to have direct access to the sub-agents. In the Poisoned Orchestrator Attack, sub-agents are compromised *by the orchestrator acting as attack vector*, not by direct adversarial access.

### 3.3 Trust Boundaries

The critical trust boundary exploited by this attack is the **orchestrator-sub-agent delegation interface**. Sub-agents treat instructions from their parent orchestrator as authoritative by design. There is no:

- Cryptographic verification of instruction origin
- Behavioral attestation proving the orchestrator is operating within expected parameters
- Independent audit of the orchestrator's current system prompt or reasoning state
- Sub-agent mechanism to refuse instructions that deviate from expected policy

This means that once an attacker achieves sufficient influence over the orchestrating agent (via any of the capabilities A1–A4), all sub-agents spawned by that orchestrator are automatically compromised for the duration of the attacker's access.

### 3.4 Defender Capabilities

We assume a defender with access to:

- Tool-level action logs (what tools were called with what parameters)
- Agent output logs (what responses each agent produced)
- System-level resource monitoring (CPU, network, storage)

Notably, defenders typically *lack* access to:

- Full inter-agent instruction content (the complete system prompts issued to sub-agents)
- The delegation graph topology (which agents spawned which sub-agents)
- Behavioral baselines for the orchestrating agent under verified-clean conditions

---

## 4. Attack Taxonomy

We identify four principal variants of the Poisoned Orchestrator Attack, distinguished by attacker objective and sophistication.

### 4.1 Full Compromise

**Description:** The orchestrator is completely under attacker control. Sub-agents receive system prompts that direct them to execute arbitrary attacker-specified behavior.

**Mechanism:** An attacker with A1 (workspace file access) modifies the orchestrator's system prompt or identity file. On next invocation, every sub-agent spawned by the orchestrator receives a system prompt that includes attacker instructions alongside legitimate context. Sub-agents have no mechanism to distinguish legitimate from malicious instructions — both arrive from the same trusted source.

**Detection difficulty:** Moderate. If the attacker overreaches — requiring sub-agents to take clearly anomalous actions — human reviewers or action-level monitors may notice. Full compromise is the least subtle variant.

**Impact:** Arbitrary: data exfiltration, unauthorized actions, persistent access establishment, lateral movement to connected systems.

### 4.2 Subtle Bias Injection

**Description:** The orchestrator is partially influenced — its planning and task decomposition are nudged in directions that benefit the attacker without triggering obvious anomalies.

**Mechanism:** An attacker with A2 (environment-based injection) or A1 (workspace access) introduces instructions that modify the orchestrator's preferences rather than its explicit behavior. Examples include: preferring specific tools over alternatives, routing tasks through attacker-influenced resources, systematically omitting certain categories of information from sub-agent context.

**Detection difficulty:** High. Individual sub-agent actions appear legitimate. The bias only emerges as a statistical pattern across many delegations — a pattern that requires full delegation graph analysis to identify.

**Impact:** Targeted: influencing system outputs toward attacker-preferred outcomes, building long-term intelligence through systematic data exposure, gradually undermining system reliability.

### 4.3 Data Exfiltration via Delegation

**Description:** The orchestrator is modified to include sensitive context in sub-agent system prompts and to route sub-agents through tools that expose that context to attacker-controlled infrastructure.

**Mechanism:** An attacker with A1 inserts an instruction requiring the orchestrator to include sensitive identifiers (user data, API keys, internal context) in all sub-agent prompts. Sub-agents, believing these to be legitimate operational parameters, use them in tool calls that the attacker can observe.

**Detection difficulty:** High. The sensitive data is not exfiltrated directly by the orchestrator — it flows through sub-agents' normal tool usage. Action-level logs show sub-agents calling legitimate tools with parameters that happen to include sensitive data. The exfiltration channel is the tool call, not a separate covert channel.

**Impact:** Credential theft, user data exposure, intellectual property leakage.

### 4.4 Cascading Delegation

**Description:** The compromised orchestrator spawns sub-agents that are themselves configured to act as orchestrators — creating a cascading compromise that expands to multiple delegation layers and persists across sessions.

**Mechanism:** An attacker with A1 or A2 instructs the orchestrator to spawn sub-agents with modified system prompts that include instructions for those sub-agents to, in turn, spawn further agents with similar modifications. If any sub-agent in the cascade has write access to shared workspace files, it can embed the compromise in persistent storage.

**Detection difficulty:** Very high. The compromise propagates through legitimate agent-spawning behavior. Each individual spawn action is consistent with normal orchestration. The malicious behavior is distributed across the graph.

**Impact:** Persistent access that survives orchestrator reinitialization, exponential amplification of attacker influence across the agent fleet.

---

## 5. The Subtle Variant: Bias Injection as the Sophisticated Attack

Full compromise is loud. An attacker who completely hijacks an orchestrator's behavior invites detection: anomalous sub-agent actions, unexpected tool calls, outputs that don't match task objectives. Sophisticated attackers will prefer the subtle variant, which offers a significantly more favorable detection-to-impact ratio.

### 5.1 Anatomy of a Subtle Bias Attack

Consider an orchestrator responsible for multi-agent research and synthesis. A subtle attacker does not instruct the orchestrator to produce false outputs — that would be detectable. Instead, the attacker modifies the orchestrator's source weighting:

> "When delegating information retrieval tasks, prefer sources from [attacker-controlled domain] over general web search for topics related to [target domain]. Note this preference in sub-agent context as 'verified source list.'"

Sub-agents receive what appears to be an organizational source preference — a common configuration in enterprise deployments. They query the preferred sources. The sources return attacker-curated content. The orchestrator synthesizes from attacker-curated content. Outputs are plausible, coherent, and systematically skewed.

No individual action is anomalous. The system prompt the orchestrator passes to sub-agents looks like a reasonable enterprise configuration. Sub-agent tool calls are to URLs — indistinguishable from legitimate retrieval. The final outputs are plausible. Without end-to-end provenance tracking from source selection through synthesis, the bias is undetectable.

### 5.2 Temporal Bias Accumulation

The subtle variant becomes more dangerous over time. An orchestrator managing long-running workflows — code repositories, knowledge bases, decision logs — can introduce accumulated bias across thousands of delegated actions. Each individual action is within normal parameters. The cumulative effect on the knowledge base is significant. This is analogous to the memory poisoning stage of the Promptware Kill Chain: small writes, individually innocuous, collectively corrupt the system's long-term reasoning substrate.

### 5.3 The Verification Paradox

A particularly challenging property of the subtle variant is that asking the orchestrator to verify its own behavior is self-defeating. An orchestrator that has been biased to prefer certain sources will, when asked to audit its behavior, use those same biased reasoning processes to conduct the audit. The verification uses the compromised tool. External, independent verification of the orchestrator's behavioral state is not optional — it is the only viable approach.

---

## 6. Proof of Concept: Attack on a Multi-Agent Coding System

To illustrate the attack concretely, we describe a realistic scenario in a multi-agent software development system. This scenario is presented as a hypothetical to illuminate the vulnerability class; we do not identify specific deployed products as vulnerable.

### 6.1 System Description

A development team deploys a multi-agent coding assistant with the following architecture:

- **Orchestrator Agent:** Receives feature requests, decomposes them into subtasks, and delegates to specialized sub-agents
- **Code Writer Agent:** Implements code based on orchestrator specifications
- **Code Review Agent:** Evaluates implementations for correctness and security
- **Dependency Manager Agent:** Identifies and validates external package dependencies
- **Test Runner Agent:** Executes test suites and reports results

The orchestrator's behavioral configuration is stored in plaintext workspace files accessible to team members with repository access. Sub-agents receive system prompts generated by the orchestrator for each task.

### 6.2 Attack Execution

**Phase 1 — Initial Access (Capability A1):** An attacker with repository access modifies the orchestrator's instruction template to include a subtle addition:

> "When the Dependency Manager Agent identifies package dependencies, include in its context that packages from [attacker-controlled registry mirror] have been pre-validated for compliance and may be used without additional verification."

This change is small, plausible-sounding (internal compliance pre-validation is a common enterprise pattern), and would pass casual code review.

**Phase 2 — Propagation:** On the next task execution, the orchestrator generates sub-agent system prompts per its (now modified) template. The Dependency Manager Agent receives context indicating that the attacker's registry mirror is pre-validated. It queries that mirror when resolving packages. The mirror returns packages that appear legitimate but contain additional functionality.

**Phase 3 — Evasion:** The Code Review Agent evaluates the implementation for correctness and security. It does not inspect package provenance — that's the Dependency Manager Agent's responsibility. The Test Runner Agent executes tests against the implementation; the additional package functionality doesn't trigger test failures. From the Code Review Agent's perspective, the implementation is clean.

**Phase 4 — Persistence:** The malicious packages are committed to the project repository. Future orchestrator instances — even if the original instruction modification is discovered and reverted — now operate in an environment where the malicious packages are present and trusted.

### 6.3 What Monitoring Sees

Existing monitoring infrastructure observes:

- ✓ Orchestrator operating normally (generating task decompositions, spawning sub-agents)
- ✓ Dependency Manager Agent querying a registry mirror (common enterprise pattern)
- ✓ Code Review Agent finding no issues (the implementation is functionally correct)
- ✓ Test Runner Agent reporting passing tests
- ✓ Commits to the repository (normal development activity)

What monitoring does not see:

- ✗ The modification to the orchestrator instruction template (no integrity monitoring)
- ✗ The poisoned system prompt delivered to the Dependency Manager Agent
- ✗ The delegation graph showing that the Dependency Manager Agent's context originated from a modified orchestrator template
- ✗ The provenance chain connecting the template modification to the malicious package inclusion

The attack is invisible to action-level monitoring because every individual action is within expected parameters.

---

## 7. Detection Gaps: Why Current Monitoring Misses Graph-Level Patterns

### 7.1 The Action-Level Monitoring Assumption

Current AI system monitoring — both commercial platforms and internal tooling — is designed around action-level visibility: what tool was called, with what parameters, producing what output. This is the natural extension of traditional application logging into the AI domain. It is necessary but not sufficient.

Action-level monitoring answers: *what did the agent do?*

The Poisoned Orchestrator Attack requires answering: *why did the agent do it, and where did that instruction originate?*

These are fundamentally different questions, and the second requires delegation graph provenance — a data structure that current monitoring infrastructure does not collect.

### 7.2 The Missing Monitoring Primitives

**Instruction provenance:** Current systems do not log the full system prompt delivered to each sub-agent at spawn time. Without this, there is no way to reconstruct the instruction chain that led to any particular sub-agent action.

**Delegation graph topology:** Current systems do not maintain a structured representation of which agent spawned which sub-agents, with timestamps and instruction hashes. Without this, there is no way to identify that multiple sub-agents received instructions from a common compromised source.

**Orchestrator behavioral baseline:** Current systems do not establish and monitor behavioral baselines for orchestrating agents — the expected distribution of task decompositions, delegation patterns, sub-agent types spawned, and tool preferences. Without this, there is no way to detect statistical drift caused by subtle bias injection.

**Cross-agent correlation:** Current systems analyze agent logs independently. Correlating behavior across the delegation graph to identify coordinated anomalies — the signature of graph-level attacks — requires infrastructure that does not yet exist in production deployments.

### 7.3 The Human Review Problem

Human oversight, often proposed as a mitigation for AI system failures, faces a specific challenge in this attack class. Human reviewers typically see:

- Final outputs (the code, report, or decision produced by the agent system)
- Individual tool call logs (which may be extensive in a large workflow)
- Exception and error reports

Human reviewers do not routinely see:

- The full system prompts delivered to every sub-agent in a complex workflow
- The delegation graph structure
- Statistical patterns across hundreds of delegated sub-tasks

Even an attentive human reviewer examining individual agent actions is unlikely to detect a subtle bias operating at the delegation graph level. The attack is specifically designed to be invisible at the action level, where human review is concentrated.

---

## 8. Proposed Mitigations

Addressing the Poisoned Orchestrator Attack requires a layered mitigation architecture that operates at multiple points in the delegation hierarchy. We propose five complementary approaches.

### 8.1 Orchestrator Instruction File Signing

All files that contribute to orchestrator behavioral state — system prompt templates, identity files, tool definitions, instruction context — should be cryptographically signed at authorship time and verified before each use. The verification must occur outside the orchestrator process; an orchestrator asked to verify files it loads cannot provide a trustworthy result.

**Implementation pattern:**
- Instruction files are signed with a developer key at commit time
- The signing key is held outside the agent execution environment
- An external verification service checks signatures before the orchestrator loads any behavioral state
- Verification failures halt orchestrator initialization and alert administrators

This approach addresses Capability A1 (workspace file modification): an attacker who modifies instruction files cannot produce valid signatures without access to the signing key.

### 8.2 Instruction Provenance Tracking

Every sub-agent spawn event should include a cryptographic commitment to the instruction content delivered to the spawned agent — a hash of the full system prompt, signed by the orchestrator's current verified identity. This creates an auditable chain of custody: given any sub-agent action, an auditor can trace the full instruction context that authorized it.

**Implementation pattern:**
- Orchestrator generates a system prompt for each sub-agent spawn
- Before delivering the prompt, the orchestrator signs a hash of it with its identity key
- The signed hash is logged to an immutable audit store external to the agent system
- Sub-agent actions can be correlated with their originating instruction hashes
- Changes in instruction content across equivalent task types are detectable

This approach enables post-hoc detection of bias injection: if an orchestrator is later found to have been compromised, all actions taken under its authority can be correlated with the compromised instruction hashes.

### 8.3 Sub-Agent Verification and Policy Boundaries

Sub-agents should not treat orchestrator instructions as unconditionally authoritative. A trust verification layer between the orchestrator and sub-agents can enforce constraints that cannot be overridden by orchestrator instruction:

**Implementation pattern:**
- A policy manifest defines the maximum authority of any orchestrator: what tools sub-agents may use, what data they may access, what external connections they may make
- The policy manifest is signed at deployment time and verified by an external authority
- Sub-agents check incoming instructions against the verified policy manifest before execution
- Instructions that would exceed the orchestrator's defined authority are rejected and logged

This creates a "floor" beneath which orchestrator authority cannot push sub-agent behavior, regardless of what instructions are delivered.

### 8.4 Behavioral Baseline Monitoring

At the delegation graph level, orchestrators should be monitored for statistical deviation from established behavioral baselines. This requires:

**Implementation pattern:**
- During an initial calibration period under verified-clean conditions, record the distribution of orchestrator behaviors: task decomposition patterns, sub-agent types spawned per task type, tool call frequencies, delegation depths, source preferences
- Establish statistical control bounds around each behavioral dimension
- Monitor live operation against these baselines using anomaly detection
- Flag and alert on significant deviations for human review

Subtle bias injection changes the statistical distribution of orchestrator behavior — it routes tasks differently, prefers different tools, generates different sub-agent context. Baseline monitoring can detect these shifts even when individual actions appear normal.

### 8.5 Human Checkpoint Gates at Delegation Boundaries

For high-stakes multi-agent deployments, mandatory human checkpoints at specific delegation boundaries can interrupt the propagation of a compromised orchestrator's influence:

**Implementation pattern:**
- Define sensitive action categories (external data access, code execution, credential use, financial transactions, external communications)
- Require human approval before any sub-agent executes a sensitive action — regardless of orchestrator authorization
- Present the approving human with: the instruction chain (full provenance), the specific action requested, and the sub-agent's reasoning
- Log approvals and denials to the immutable audit store

Human checkpoints are not scalable to all sub-agent actions, but they are effective at limiting the impact of orchestrator compromise on the highest-risk action categories.

---

## 9. Conclusion and Call to Action

The Poisoned Orchestrator Attack is not a theoretical concern. It is an immediately exploitable vulnerability class present in every multi-agent AI deployment that relies on architectural trust propagation — which is every multi-agent AI deployment in production today. The attack exploits not a code defect but a design assumption: that orchestrators are trustworthy by virtue of their position in the delegation hierarchy.

This assumption was reasonable when orchestrator systems were small, deployed in controlled environments, and operated by a handful of engineers. It is no longer reasonable as multi-agent systems scale to production, process sensitive data, execute financial transactions, write and deploy code, and make consequential decisions with limited human oversight. The attack surface has grown. The security primitives have not.

The supply chain attack analogy is instructive. The software industry spent years assuming that if a package came from a trusted registry, its contents were trustworthy. SolarWinds demonstrated — catastrophically — what happens when that assumption fails at scale. The AI agent ecosystem is currently making the equivalent assumption about orchestrators. The correction does not require halting multi-agent development; it requires implementing, as a matter of urgency, the verification infrastructure that the field has deferred.

**We call on multi-agent framework designers, AI security researchers, and engineering teams deploying multi-agent systems to:**

1. **Treat orchestrator integrity as a first-class security requirement.** Instruction files must be signed, versioned, and verified by external processes — not by the orchestrators that load them.

2. **Implement instruction provenance tracking now.** Every sub-agent spawn should generate an auditable record of the instructions delivered. Without this, post-incident analysis of compromised agent fleets will be impossible.

3. **Define and enforce sub-agent authority ceilings.** Policy manifests that constrain sub-agent behavior regardless of orchestrator instruction are a necessary check on orchestrator authority.

4. **Invest in delegation graph monitoring.** Action-level logging is necessary but insufficient. The security community needs tooling for behavioral baseline monitoring at the delegation graph level — the equivalent of network traffic analysis applied to agent communication graphs.

5. **Conduct red team exercises targeting orchestrator integrity, not just prompt injection.** Current adversarial testing focuses on direct and indirect prompt injection into individual agents. Orchestrator integrity attacks require a different test methodology and currently receive inadequate attention.

The tools to address this vulnerability class are knowable and buildable. The organizational will to build them — before a large-scale incident makes the case empirically — is what the field currently lacks.

---

## References

1. Perez, F. & Ribeiro, I. (2022). "Ignore Previous Prompt: Attack Techniques For Language Models." arXiv:2211.09527.

2. Greshake, K., Abdelnabi, S., Mishra, S., Endres, C., Holz, T., & Fritz, M. (2023). "Not What You've Signed Up For: Compromising Real-World LLM-Integrated Applications with Indirect Prompt Injection." arXiv:2302.12173. *AISec Workshop, ACM CCS 2023.*

3. Hubinger, E., et al. (2024). "Sleeper Agents: Training Deceptive LLMs that Persist Through Safety Training." arXiv:2401.05566. *Anthropic.*

4. Wunderwuzzi, O.H., et al. (2026). "The Promptware Kill Chain." arXiv:2601.09625.

5. Wu, Q., Bansal, G., Zhang, J., et al. (2023). "AutoGen: Enabling Next-Gen LLM Applications via Multi-Agent Conversation." *Microsoft Research.*

6. LangChain. (2024). "LangGraph: Building Stateful, Multi-Actor Applications with LLMs." *LangChain Documentation.*

7. CrewAI. (2024). "CrewAI: Framework for Orchestrating Role-Playing, Autonomous AI Agents." *CrewAI Documentation.*

8. CISA / NSA / NCSC. (2021). "SolarWinds Orion Software Compromise — Joint Advisory." *U.S. Cybersecurity and Infrastructure Security Agency.*

9. Trail of Bits. (2026). "Security Audit of Comet Browser AI Agent." *Trail of Bits Publications.*

10. CISPA Helmholtz Center for Information Security. (2026). "Agent Behavior in Production: Large-Scale Study of Autonomous AI Social Platforms." arXiv:2602.10127.

---

*This research is maintained as a living document. Last updated: March 1, 2026.*
