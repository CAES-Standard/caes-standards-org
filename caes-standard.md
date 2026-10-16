# CAES
## Constitutional AI Execution Standard

**Version:** v0.1.1 (Draft)
**Status:** Superseded by [v0.2.0](caes-standard-v0.2.0-draft.md)
**Working Group:** CAES-WG
**Category:** Normative Specification

> **Superseded for new conformance claims.** The current public draft is CAES v0.2.0 with CPP v1.0. Read the [v0.2.0 draft](caes-standard-v0.2.0-draft.md) or the [HTML specification](spec/caes-v0.2.0.html). The text below preserves v0.1.1.

---

## Overview

CAES (Constitutional AI Execution Standard) defines the minimum structural requirements for AI systems that execute actions with legal, financial, regulatory, or operational consequence.

A CAES-conformant system does not merely log what happened.  
It proves — before execution occurs — that the action was authorized.

This repository contains the normative specification, version history, and working group materials for CAES.

---

## Why CAES Exists

AI systems have moved beyond advisory roles. They now:

- Commit transactions
- Modify infrastructure
- Trigger automated workflows
- Interact with physical systems
- Influence consequential human decisions

Yet there is no widely adopted standard defining what it means to **authorize** an AI action and **prove** that authorization occurred.

CAES addresses this structural accountability gap.

---

## Scope

CAES applies to any AI system that crosses an **Effect Boundary**, including:

- Digital state mutation  
- External system invocation  
- Communication with consequential impact  
- Behavioral influence in regulated contexts  
- Physical actuation (robots, vehicles, automation)  
- Cross-boundary access to regulated data  

If an AI system can produce consequential effects without independent human mediation, CAES scope applies.

Purely advisory systems are out of scope.

---

## Core Principles

A conformant CAES implementation must provide:

- Deterministic policy evaluation
- Pre-execution authorization receipts
- Cryptographically verifiable evidence
- Append-only causal event recording
- Fail-closed enforcement
- Offline-verifiable artifacts
- Explicit authority delegation for high-stakes actions

CAES defines structural requirements — not implementation architecture.

---

## Core Primitives

CAES formalizes three foundational objects:

1. **Decision Receipt**  
   A cryptographically signed authorization artifact produced at policy evaluation time.

2. **PolicyHash**  
   A deterministic hash of the canonical policy state active at evaluation.

3. **Governed Spine**  
   An append-only, causally ordered, tenant-scoped record of governed events.

These primitives form the minimum constitutional substrate for AI execution.

---

## Compliance Levels

CAES defines three conformance tiers:

### Level 1 — Receipt-Bounded Execution
All effect-boundary actions require verified Decision Receipts. Fail-closed enforcement.

### Level 2 — Verifiable Evidence Chain
Cryptographic signing, PolicyHash canonicalization, append-only spine, offline verification.

### Level 3 — Full Constitutional Conformance
Human authority delegation model, complete causal invariants, full spine specification, and effect classification.

Conformance claims must cite version and publish a complete Conformance Statement.

---

## Non-Compliant Patterns (Explicitly Disallowed)

CAES prohibits:

- Prompt-only “policy” enforcement
- Post-hoc logging presented as authorization proof
- Non-deterministic LLM-only policy evaluation
- Mutable logs masquerading as evidence ledgers
- Silent degradation under failure
- Invocation-layer governance bypass

The specification includes a full Non-Compliant Patterns section to prevent dilution of the standard.

---


---

## Conformance Claims

Implementations claiming CAES compliance **must publish**:

- CAES version
- Claimed compliance level
- Receipt verification method
- PolicyHash canonicalization method
- Fail-closed enforcement description
- Effect Boundary categories governed

Unversioned or incomplete claims are non-conformant.

---

## Reference Implementation

The CAES Working Group designates one reference implementation per version.

**CAES v0.1.1 Reference Implementation:**  
Keon Systems — Governed Execution  
https://keon.systems

Reference designation does not imply commercial endorsement.  
Other implementations may independently conform.

---

## Working Group

CAES is maintained by the CAES Working Group (CAES-WG).

The Working Group publishes:
- Normative specification versions
- Amendment proposals
- Conformance guidelines
- Mark usage policy

Specification changes are versioned.  
There are no informal amendments.

---

## Amendment Process

To propose a change:

1. Identify the normative requirement affected.
2. Describe the structural issue.
3. Document backward compatibility impact.
4. Submit as a versioned proposal.

Changes are incorporated only via published version increment.

---

## Mark Usage

“CAES” and associated marks are used to designate conformance with a specific published version.

Use of the CAES mark requires a complete Conformance Statement.

Mark usage guidelines are provided in `/assets`.

---

## License

The CAES specification text is published under a permissive documentation license to encourage adoption and independent implementation.

See `LICENSE.md` for details.

---

## Status

CAES v0.1.1 is a public draft released for review and comment.

Feedback and formal amendment proposals are welcome.

---

CAES  
Constitutional AI Execution Standard  
March 2026
