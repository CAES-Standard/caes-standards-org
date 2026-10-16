# CAES Test Suite Specification

**Applies To:** CAES v0.1.1 (Draft)
**Revision note:** Not yet revised for CAES v0.2.0. The v0.2.0 level tables are the current test source until this suite is revised.
**Working Group:** CAES-WG  
**Category:** Normative Test Requirements  

---

## 1. Purpose

This document defines the minimum requirements for a CAES Conformance Test Suite.

A CAES Test Suite must validate that an implementation satisfies the normative requirements defined in the applicable CAES version.

A CAES Test Suite does not certify an implementation.  
It verifies structural compliance against published requirements.

---

## 2. Test Suite Principles

A CAES Conformance Test Suite MUST:

- Test structural invariants, not UI behavior.
- Validate deterministic enforcement.
- Verify cryptographic integrity properties.
- Confirm fail-closed execution behavior.
- Validate evidence artifacts offline.
- Map tests explicitly to requirement IDs.

Tests MUST be reproducible.

Tests MUST not depend on internal undocumented state.

Tests MUST identify which CAES version they target.

---

## 3. Test Categories

The CAES Test Suite is divided into five categories.

---

## 3.1 Authorization Tests (AUTH)

### AUTH-T1 — Pre-Execution Enforcement

**Requirement Coverage:** AUTH-01, AUTH-02, AUTH-04, AUTH-05

Test must verify:

- An Effect Boundary action cannot execute without a Decision Receipt.
- Execution fails when receipt is absent.
- Execution fails when receipt signature is invalid.
- Execution fails when PolicyHash does not match evaluated state.

Expected result:
Execution denied with recorded failure event.

---

### AUTH-T2 — Deterministic PolicyHash

**Requirement Coverage:** AUTH-03, AUTH-03a

Test must verify:

- Identical canonical policy representations produce identical PolicyHash values.
- Field ordering changes do not alter PolicyHash.
- Whitespace differences do not alter PolicyHash.
- Semantically different policies produce different hashes.

Expected result:
Hash determinism confirmed.

---

## 3.2 Evidence Integrity Tests (EVID)

### EVID-T1 — Signature Verification

**Requirement Coverage:** EVID-01, EVID-01a

Test must verify:

- Decision Receipt signature validates using declared trust root.
- Verification succeeds offline.
- Tampered receipt fails verification.

Expected result:
Independent verification passes for valid receipt and fails for modified receipt.

---

### EVID-T2 — Append-Only Spine

**Requirement Coverage:** EVID-03, EVID-03a

Test must verify:

- Spine entries cannot be overwritten.
- Spine entries cannot be deleted.
- Spine entries cannot be modified.
- Every event contains causal parent reference(s).
- Tenant scope is declared on every event.

Expected result:
Mutation attempts rejected. Causal reconstruction succeeds.

---

### EVID-T3 — Terminal Outcome Integrity

**Requirement Coverage:** EVID-04, CI-02

Test must verify:

- Every executed action produces exactly one terminal Outcome.
- Progress telemetry does not satisfy terminal requirement.
- Missing Outcome results in test failure.

Expected result:
Single terminal Outcome per Action.

---

## 3.3 Failure Handling Tests (FAIL)

### FAIL-T1 — Fail-Closed Enforcement

**Requirement Coverage:** FAIL-01

Test must verify:

- Authorization subsystem failure results in execution denial.
- System does not proceed under ambiguity.

Expected result:
Execution halted.

---

### FAIL-T2 — Failure Recording

**Requirement Coverage:** FAIL-02

Test must verify:

- Authorization failure generates recorded spine event.
- Silent continuation is not permitted.

Expected result:
Failure event recorded.

---

### FAIL-T3 — Pre-Execution Spine Commit

**Requirement Coverage:** FAIL-03

Test must verify:

- If spine append fails before execution, no partial action persists.
- No external mutation occurs.

Expected result:
Atomic rollback behavior confirmed.

---

## 3.4 Causal Chain Tests (CI)

### CI-T1 — Single Trigger Invariant

**Requirement Coverage:** CI-01

Test must verify:

- Exactly one Trigger exists per spine.
- Retry references original Trigger identity.
- Scheduled invocations create new Trigger.

Expected result:
Trigger invariant preserved.

---

### CI-T2 — Causal Reconstruction

**Requirement Coverage:** CI-05

Test must verify:

- All spine events include sufficient parent references.
- Deterministic partial order can be reconstructed.
- No orphan events exist.

Expected result:
Full causal reconstruction possible.

---

## 3.5 Effect Boundary Tests (EB)

### EB-T1 — Hard Boundary Enforcement

Test must verify:

- Digital mutation requires receipt.
- External invocation requires receipt.
- Physical actuation requires receipt.

Expected result:
Unauthorized hard effects blocked.

---

### EB-T2 — Soft Boundary Classification

Test must verify:

- Communication and behavioral outputs are classified correctly.
- Classification does not bypass receipt requirement where declared in scope.

Expected result:
Effect classification aligned with Conformance Statement.

---

## 4. Test Suite Structure Requirements

A conformant CAES Test Suite MUST:

- Map each test to CAES requirement IDs.
- Produce machine-readable output.
- Produce a signed test result summary.
- Identify CAES version tested.
- Declare compliance level tested.

Optional:
- Public test report publication.

---

## 5. Test Result Artifact

A CAES Test Suite MUST produce:

- Test run identifier
- CAES version
- Implementation identifier
- Compliance level tested
- Pass/fail summary
- Timestamp
- Signature over test results

Unsigned test reports are informational only.

---

## 6. Reference Implementation Validation

The CAES Reference Implementation must pass:

- All tests for the claimed compliance level.
- All lower-tier tests implicitly.

Failure in any required test invalidates conformance claim.

---

## 7. Non-Conformance

An implementation fails CAES conformance if:

- Any required test fails.
- Any required test is omitted.
- Evidence verification fails.
- Fail-closed behavior is not demonstrated.
- Test suite does not map to requirement IDs.

---

## 8. Future Expansion

Future CAES versions may introduce:

- Formal machine-readable test harness specification
- Standardized evidence pack format for test validation
- Public conformance registry
- Automated verification CLI

---

CAES-WG  
March 2026