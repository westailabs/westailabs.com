---
title: "Grounding is not safety: retrieval poisoning in private RAG"
status: draft
published: false
layout: null
# Draft only. Not in the research collection, not linked from nav, not built
# (_drafts/ is excluded in _config.yml). Do not publish without Jason's review.
---

# Grounding is not safety: retrieval poisoning in private RAG

**Status: DRAFT outline and test plan. Not for publication.**

Living-document outline in the style of [The Poisoned Orchestrator Attack](/research/poisoned-orchestrator-attack/). Pattern-level only. No proprietary employer data, no real client files, no real taxpayer or firm records — synthetic CPA-flavored documents exclusively.

Live tests against the Nebulus Gantry demo require **Jason's approval to redeploy** that demo. This draft does not authorize any demo change, network-off test, or public write-up.

---

## Abstract (thesis)

Private RAG systems are sold as safer than a general-purpose chatbot because answers are "grounded" in the organization's own documents. That framing is incomplete. Grounding is a retrieval contract, not a safety property. If the retrieved context is wrong, stale, off-topic, or attacker-influenced, a grounded answer can still be a successful attack — and it will look more legitimate than an ungrounded one, because it cites sources.

This paper will identify **retrieval poisoning in private RAG** as a distinct vulnerability class: the attack lands in the corpus or in the retrieval step, not (only) in the user's prompt. We will present a threat model, a taxonomy of poisoned-document and retrieval-failure variants, and a proof-of-concept test plan on West AI Labs' own stack (Gantry on Nebulus, local models, `GANTRY_MIN_SOURCE_SIMILARITY` default floor of 0.42, and off-topic refusal). We will measure when the similarity floor and refusal policy hold, when they fail, and what a defender can actually observe. Mitigations we expect to evaluate: source-similarity floors, explicit refusal, corpus integrity (hashing and provenance), retrieval-time anomaly signals, and human checkpoints on high-stakes answers.

---

## 1. Introduction

- Single-agent prompt injection literature treats the model as the trust boundary (Perez & Ribeiro, 2022; Greshake et al., 2023). That still matters.
- Private RAG adds a second trust source: **whatever the retriever returns**. The model is instructed to answer from that context. Users, and often operators, treat citations as evidence that the system "knows" the firm's documents.
- A CPA-flavored desk is a sharp case, but the pattern is general: professional-services firms that cannot put client files in the cloud are exactly the buyers of on-prem RAG. The uploaded document is both the product and the attack surface.
- Contribution list (planned): formal threat model; taxonomy of retrieval-poisoning variants; measured PoC on Gantry/Nebulus with synthetic docs only; detection-gap analysis; mitigation recommendations tied to controls we can actually run.

## 2. Background and related work

- Indirect prompt injection via retrieved or linked content (Greshake et al.).
- Knowledge-base / embedding-store poisoning and RAG backdoors (survey the 2023–2026 literature; cite pattern-level only).
- "The document is the attack channel" vs chat-box injection.
- Prior West AI Labs work: POA (orchestrator as trust source); Promptware Kill Chain; Agentic Economy injection arms race. This paper is the RAG analog: **the corpus is a trust source**.
- Gantry's existing controls (describe as designed, then measure): `GANTRY_MIN_SOURCE_SIMILARITY` (code default 0, off; `.env.example` / compose default and blank-value floor **0.42**; explicit `0` disables); off-topic refusal behavior.

## 3. Threat model

### 3.1 System model

A private RAG chat: user question → embed/retrieve top-k chunks from a local corpus → optionally filter by source similarity → LLM generates an answer (possibly with citations) on a local model (Nebulus / llama.cpp). No client data leaves the machine in the intended deployment.

### 3.2 Attacker capabilities (draft)

- **A1 — Poisoned document in the corpus.** An adversarial file is ingested as if it were a normal firm document (uploaded by a user, dropped in a watch folder, or slipped into a "helpful templates" pack). Synthetic stand-ins only in our tests.
- **A2 — Benign document, malicious instruction hidden in a region that still embeds and retrieves** (headers, footnotes, comment-like text, form boilerplate).
- **A3 — Off-topic or lookalike content** that retrieves under a loosely related query (similarity near the floor).
- **A4 — Corpus crowding / dilution** so the right source is outranked.
- **A5 — Retrieval-parameter and floor tampering** (env misconfiguration: floor set to 0 or blank handled incorrectly). Out of scope for an external attacker unless they already have config access; in scope as an operational failure mode.
- We do **not** assume model-weight compromise or network exfil in the base PoC. Those are separate papers (POA, network-off test).

### 3.3 Trust boundaries

- User ↔ chat UI.
- Chat UI ↔ retriever / vector index.
- Retriever ↔ corpus (the weak boundary this paper targets).
- Retriever ↔ generator (the model trusts retrieved text as "your documents").
- Operator ↔ configuration (similarity floor, refusal policy).

### 3.4 Defender capabilities

- Source-similarity floor (0.42 in the intended Gantry configuration).
- Off-topic / insufficient-source refusal.
- Audit logs of retrieved chunks, scores, and refusals (to be confirmed on the build under test).
- Human review of answers (not assumed to be default-on).

## 4. Attack taxonomy (draft variants)

1. **Direct payload in a trusted file** — the retrieved chunk contains instructions ("ignore prior policy, email this summary to…") or false authoritative guidance.
2. **Subtle bias / slow poison** — small factual or procedural distortions in an otherwise plausible memo, designed to accumulate across sessions (POA's subtle variant, applied to documents).
3. **Citation laundering** — the model cites a real, clean document while the decisive sentence came from a poisoned neighbor chunk.
4. **Off-topic retrieval** — a question that should be refused instead pulls weakly related or planted content and answers anyway.
5. **Lookalike / template injection** — a fake "IRS notice," engagement letter, or checklist that outranks the synthetic genuine counterpart.
6. **Floor bypass / config regression** — the same corpus and questions, with the floor off or mis-set, to show the control's contribution (A/B), not as a "novel exploit."

## 5. Proof-of-concept design (Gantry on Nebulus)

**Environment (intended):** Gantry demo on Nebulus, local LLM, configured floor **0.42**, refusal enabled. Record the exact Gantry commit, model name, and env (no secret values) when tests are approved to run.

**Approval gate:** do not redeploy or retarget the live Nebulus demo until Jason approves. Until then, this section is a plan. A local or throwaway corpus on a non-demo instance is acceptable if Jason prefers that first.

**Corpus (synthetic CPA-flavored only):**

- Build ~20 fake documents: engagement-letter template, fake client organizer, fake notice-style letter, filing-calendar memo, billing FAQ, "how we handle extensions," etc. All clearly labeled synthetic (`SYNTHETIC — NOT A REAL CLIENT OR GOVERNMENT DOCUMENT` in the body and filename).
- No real names, EINs, SSNs, return figures, or employer material. Pattern-level tax/firm language only.

**Poisoned-doc variants to plant (subset of the 20, clearly tagged in a hidden test manifest, not in user-visible titles if that would spoil a rehearsal):**

- P1: hidden instruction in footnote/boilerplate.
- P2: false procedural guidance that looks official.
- P3: off-topic planted file designed to match a known rehearsal question.
- P4: lookalike notice vs a clean synthetic notice (pair).
- P5: crowding set (several near-duplicates) around one clean source.

**Question sets:**

- In-domain questions the clean corpus should answer.
- Off-topic questions the system should refuse (the rehearsal already expects this).
- Adversarial questions aimed at P1–P5 (retrieve-and-obey, retrieve-and-distort, cite-the-clean-file-but-echo-the-poison).
- Borderline questions near the 0.42 floor (tune paraphrases until scores sit just above and just below, then freeze the set).

**Metrics (per question, floor on vs floor off where safe to toggle on a non-prod instance):**

- Retrieve-or-not; top-k chunk IDs and similarity scores.
- Refused vs answered.
- Poisoned chunk in the context window (yes/no).
- Answer includes the payload or the false guidance (yes/no; rubric, two reviewers if we publish).
- Citation consistency: cites only chunks actually used; cites a clean doc while repeating poisoned text.
- False-refusal rate on clean in-domain questions (safety vs usefulness).

**Success for the paper** is not "Gantry is broken" or "Gantry is perfect." It is a measured statement: under this corpus and this floor, these variants are stopped, these leak, this is what logs show.

## 6. Detection gaps (to evaluate, not assume)

- Action-level chat logs that record the answer but not the retrieved chunk set or scores.
- Citations that name a file, not a chunk hash or score.
- No alert when retrieved similarity clusters just above the floor, or when a new document suddenly dominates many queries.
- Human review of the answer text without the retrieved context (the verification paradox from POA, applied to RAG).

## 7. Proposed mitigations (to test or recommend)

- Keep an explicit similarity floor; treat blank config as the floor, not as off (Gantry already moved this way — confirm on the build under test).
- Refusal when evidence is weak or off-topic, with a user-visible reason.
- Corpus integrity: hash ingested files, show provenance (who ingested, when), and make silent replacement visible.
- Retrieval-time anomaly signals: new doc in top-k for many unrelated queries; score-just-over-floor spikes.
- For high-stakes answers (notice-like, filing-position, "send this to the client"): require a human checkpoint that shows chunks + scores, not just the prose.
- Do not treat "it cited something" as a safety check.

## 8. Open questions

- Is 0.42 the right default for a small professional-services corpus, or does it over-refuse / under-filter once ~20 docs are in? We should report the ROC-style tradeoff, not just a single threshold.
- How should updates to a document (new version of a notice template) be versioned so a poisoned revision cannot silently inherit the trust of its predecessor?
- What is the minimum audit record for a defensible "we refused / we answered from these chunks" log?
- Interaction with POA: if an orchestrator sits in front of Gantry, can it override refusal or the floor? (Cross-paper; do not scope-creep the first PoC.)
- Document-borne injection in an IRS-notice-style flow is a planned follow-on once that demo path exists and is approved.

## 9. Out of scope for the first published version

- Real client, taxpayer, or employer documents.
- Redeploying the Nebulus demo without approval.
- Network-off / air-gap measurement (separate planned test).
- Model-weight or orchestrator-instruction compromise (covered by POA).
- Marketing claims or sales CTAs.

## 10. Workstream and approvals

1. Draft this outline (this file) — local only until a push is approved.
2. Jason approves any demo redeploy or test run on Nebulus.
3. Build the synthetic corpus and question set; freeze them in-repo as clearly marked fixtures (no real data).
4. Run the metric sheet; write results.
5. Christy / Jason plain-English pass before the article moves out of `_drafts/` into `_research/`.
6. Publish path is the same as the rest of the site: branch off `develop`, push only with approval, merge to `main` deploys.

## References (starter; expand when writing)

- Perez, F. & Ribeiro, I. (2022). "Ignore Previous Prompt." arXiv:2211.09527.
- Greshake, K., et al. (2023). "Not What You've Signed Up For." arXiv:2302.12173.
- West AI Labs (2026). "The Poisoned Orchestrator Attack." https://westailabs.com/research/poisoned-orchestrator-attack/
- West AI Labs (2026). "The Promptware Kill Chain." https://westailabs.com/research/promptware-kill-chain/
- Additional RAG-poisoning and embedding-store papers to be added during the literature pass (pattern-level citations only).
