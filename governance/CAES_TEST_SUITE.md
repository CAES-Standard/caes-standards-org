# CAES Test Suite Specification

**Applies To:** CAES v0.2.0 and CPP v1.0
**Status:** Public draft
**Working Group:** CAES-WG
**Category:** Normative test requirements

The v0.1.1 labels `AUTH-01`, `EVID-01`, and the rest of that set are not v0.2.0 requirement IDs. Tests for this version MUST cite the `CAES-` and `CPP-` IDs in the drafts. Vectors and pass conditions are in `conformance/CAES_v0.2.0_FIXTURES.md`.

A test suite does not certify an implementation. It checks structural compliance against the published requirements for the claimed level.

---

## 1. Principles

A CAES conformance test suite MUST:

- Test the claimed level, and the levels below it.
- Treat Level 1 "unverifiable" as a missing required field or a failed readback. It MUST NOT fail a Level 1 implementation for a missing signature, PolicyHash, spine, or Evidence Pack.
- At Level 2 and Level 3, require the JWS profile, PolicyHash profile, spine, and pack spec 0.2.0.
- Map every test to a requirement ID.
- Be reproducible from the fixture inputs.
- Not depend on undocumented internal state.
- Name the CAES version and the compliance level.

Level 2 and Level 3 test-result artifacts MUST follow the Evidence Pack annex. Level 1 results MUST still name the version and the level. They are not required to be signed.

---

## 2. Requirement map

| Test | IDs | Level |
| :--- | :--- | :--- |
| PRE-T1 Pre-effect record | CAES-PRE-01, CAES-PRE-02, CAES-PRE-03 | 1+ |
| PRE-T2 One disposition | CAES-DSP-01 | 1+ |
| SIG-T1 JWS profile | CAES-SIG-01, CAES-SIG-02, CAES-SIG-03, CAES-SIG-05 | 2+ |
| SIG-T2 Key identity | CAES-SIG-04 | 2+ |
| SIG-T3 Time bounds | CAES-SIG-06, CAES-SIG-07 | 2+ |
| HASH-T1 Effective bundle | CAES-HASH-01, CAES-HASH-02, CPP-HASH-01, CPP-HASH-02, CPP-HASH-03, CPP-HASH-04 | 2+ |
| SPN-T1 Append-only spine | CAES-SPN-01 | 2+ |
| PAK-T1 Evidence Pack | CAES-PAK-01 | 2+ |
| STA-T1 Three states | CAES-STA-01, CAES-STA-02, CAES-STA-03, CPP-OUT-02, CPP-OUT-03 | 1+ |
| EFF-T1 Composition | CAES-EFF-01, CAES-EFF-02, CPP-EFF-01 | 2+ |
| TEN-T1 Tenant selectors | CPP-TEN-01 | 2+ |
| ACT-T1 Action mapping | CPP-ACT-01, CPP-ACT-02, CPP-ACT-03, CPP-ACT-04 | 2+ |
| OBL-T1 Obligation phases | CAES-OBL-01, CPP-OBL-01, CPP-OBL-02, CPP-OBL-03 | 2+ |
| STR-T1 Streaming | CAES-STR-01, CAES-STR-02, CAES-STR-03 | 1+ |
| REP-T1 Replay lifecycle | CAES-REP-01, CAES-REP-02, CAES-REP-03 | 1+ |
| CH-T1 Chaos modes | CAES-CH-00, CAES-L3-01, CAES-L3-02 | 3 |
| PRV-T1 Recording obligations | CAES-PRV-01, CAES-PRV-02 | 1+ |

CPP is not required at Level 1 (CAES-CPP-01). Level 2 and Level 3 suites MUST include the CPP rows.

---

## 3. Required checks

### PRE-T1 — Pre-effect record

The effect does not start until a Decision Record exists, the write has been read back, and the bytes match. A missing record, a failed readback, or a record written after the effect is a failure. A storage acknowledgment without readback is a failure.

### PRE-T2 — Disposition

The record names exactly one of `Approved`, `Modified`, `Denied`, or `RequiresHumanAuthorization`. A missing disposition is a Level 1 structural failure.

### SIG-T1 — JWS profile

Level 2 and Level 3 only. The receipt is flattened JWS JSON. `alg` is `EdDSA` or `ES256`. `typ` is `caes-receipt+jws`. `none` and shared-secret algorithms fail. Required claims from CAES-SIG-05 are present. The fixture receipt verifies. A flipped signature bit fails.

### SIG-T2 — Key identity

An unknown or revoked `kid` grants no execution authority. A signature under either key inside a published rotation overlap is accepted. Overlap is not by itself a denial.

### SIG-T3 — Time bounds

`iat` and `exp` are RFC 3339. For an interchange claim, `exp - iat` greater than 15 minutes grants no execution authority, and skew beyond 120 seconds grants none. Archival verification may still report that the signature matches and MUST label the time untrusted. `exp` does not erase an archived receipt. Verification MUST NOT report that the authority is unused.

### HASH-T1 — Effective bundle

The hash input is the JCS bytes of the effective bundle. The fixture positive vector matches. Key order and insignificant whitespace do not change the hash. A different rule action does. A version label or a pointer as the hash input fails.

### SPN-T1 — Spine

Level 2 and Level 3. An append failure means the effect does not proceed. Existing entries cannot be modified or deleted in place. Level 1 does not fail this test for the absence of a spine.

### PAK-T1 — Evidence Pack

Level 2 and Level 3. The pack meets pack spec 0.2.0. Verification uses no issuer network call. A missing entry fails. A partial export does not verify as sealed. Offline success is not reported as unused authority.

### STA-T1 — Three states

An `Approved` disposition remains `Approved` when a later obligation blocks the effect. `enforcement_outcome` records the block. Outcome evidence does not rewrite the disposition.

### EFF-T1 — Composition

An `ExternalInvocation` that also changes governance state and workflow state is evaluated against every applicable category. Dropping a category fails the test. An action that lists `SafetyCriticalActuation` is Denied under this version.

### TEN-T1 — Tenant selectors

Comparison is exact code points. `*` is the only wildcard. A tenant identifier equal to `*` is rejected. A `*` policy without a `cross_tenant_policy` delegation is ineligible.

### ACT-T1 — Action mapping

`allow` does not execute after authority expires. `deny` has no effect. `modify` that executes the pre-transform payload fails. `escalate` does not execute on the disposition alone.

### OBL-T1 — Obligation phases

A failed `pre_effect` obligation grants no authority. A failed `continuous` obligation stops further covered bytes and keeps the bytes already released in the outcome evidence. A failed `post_effect` obligation is recorded as remediation and is not described as undoing the effect.

### STR-T1 — Streaming

The implementation's Conformance Statement names `buffered` or `segmented`. In `buffered` mode, no governed byte leaves before one receipt covers the complete output. In `segmented` mode, each segment is hashed and covered before its first byte. A completion record produced after unknown bytes have already left fails the test.

### REP-T1 — Replay lifecycle

Reservation of `(issuer, tenant_id, audience, action_id, payload_hash, receipt_id)` is atomic and durable before the effect. A second reservation does not execute. A crash before any effect can return to `Abandoned` and retry inside `exp`. A crash after the effect and before completion becomes `Uncertain`. A fresh receipt does not repeat that effect while the state is `Uncertain`. Offline verification does not report the authority unused.

### CH-T1 — Chaos modes

Level 3. Each of the eight modes meets the pass condition in the CAES chaos table and in the fixture document. A test that only checks for a shared error code fails. Two modes at once still require every applicable prevention and every applicable evidence record.

### PRV-T1 — Recording

Observation or recording without the decision required at the claimed level fails. The legal-actor sentence is not a test. CAES-PRV-02 is: an attributable decision, an identified operator, authorization before capture or use, and an auditable effect.

---

## 4. Results

A result file MUST include the run identifier, CAES version, implementation identifier, compliance level, pass or fail per test ID, and an RFC 3339 timestamp.

Unsigned Level 2 or Level 3 reports are informational only. They do not satisfy the Evidence Pack annex.

Failure of a required test, omission of a required test, or a missing requirement-ID mapping invalidates the conformance claim for that level. Passing this suite is not certification by CAES-WG, and it is not evidence that a second implementation interoperates.

---

CAES-WG
v0.2.0 public draft
