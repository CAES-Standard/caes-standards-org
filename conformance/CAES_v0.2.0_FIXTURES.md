# CAES v0.2.0 conformance fixtures

**Applies to:** CAES v0.2.0 public draft and CPP v1.0
**Status:** Acceptance evidence for the interoperability release gate
**Not:** A certification, and not evidence that two implementations interoperate

These fixtures are the vectors and expected outcomes for the requirement IDs in the drafts. A test passes only when its own pass condition holds. A shared error code is not a pass.

The signing key in section 3 is a published fixture seed. It MUST NOT be used as a production trust anchor.

Hash input for the ASCII objects below is RFC 8785 (JCS): UTF-8, object members ordered by UTF-16 code unit, no insignificant whitespace, and no Unicode normalization. Python `json.dumps` with sorted keys and tight separators matches JCS on these particular objects because every string is ASCII and every number is a small integer. That coincidence is not a general JCS implementation.

---

## 1. PolicyHash vectors

Requirement IDs: CAES-HASH-01, CAES-HASH-02, CPP-HASH-01, CPP-HASH-02, CPP-HASH-03, CPP-HASH-04.

Effective bundle, already resolved:

```json
{
  "conflict_resolution": "priority",
  "evaluation": "deny_by_default",
  "id": "pol-1",
  "rules": [
    {"action": "allow", "effect_type": "Execution", "id": "r1", "priority": 0}
  ],
  "version": "1"
}
```

JCS bytes, and the only accepted hash input:

```text
{"conflict_resolution":"priority","evaluation":"deny_by_default","id":"pol-1","rules":[{"action":"allow","effect_type":"Execution","id":"r1","priority":0}],"version":"1"}
```

| Case | Expected |
| :--- | :--- |
| SHA-256 of the JCS bytes, lowercase hex | `3a764d803e05908e335042db60b4fb03472b7fe33d6b45023995687e8a2190ef` |
| Same members inserted in a different order | The same JCS bytes and the same hash |
| Insignificant whitespace around those members | Not hashed. The hash input is the JCS bytes |
| `rules[0].action` changed from `allow` to `deny` | `4571e07d25ee65326381460410414b9dca042972df0816bba2a9470c4686ef08` |
| SHA-256 of the version label `1` | `6b86b273ff34fce19d6b804eff5a3f5747ada4eaa22f1d49c01e52ddb7875b4b` and this MUST NOT be accepted as the PolicyHash |

A payload with duplicate property names is rejected before hashing. It has no PolicyHash.

Level 1 Decision Records are not required to carry this hash. A Level 2 receipt that omits it, or that carries `6b86b273ff34fce19d6b804eff5a3f5747ada4eaa22f1d49c01e52ddb7875b4b` for this bundle, grants no execution authority.

---

## 2. Level 1 records

Requirement IDs: CAES-PRE-01, CAES-PRE-02, CAES-PRE-03, CAES-DSP-01.

### Positive

A Level 1 implementation MAY persist this record with no signature, no PolicyHash, and no spine append. The effect may start only after an immediate readback matches these bytes and the disposition is `Approved` or `Modified` with the transformed payload named.

```json
{
  "action_descriptor": "noop",
  "action_id": "act-1",
  "caes_version": "v0.2.0",
  "disposition": "Approved",
  "iat": "2026-03-01T00:00:00Z",
  "payload_hash": "8d720a027dd3ca100d7c6e92ca9a67e832f1ad67983f265aa02a64ade5c80f8d",
  "tenant_id": "tenant-a"
}
```

`payload_hash` here is SHA-256 of the JCS action payload `{"op":"noop"}`, which is `8d720a027dd3ca100d7c6e92ca9a67e832f1ad67983f265aa02a64ade5c80f8d`. Level 1 does not require this field to be a JWS claim. It is present so the record can be distinguished from a different action.

### Negative

| Input | Expected |
| :--- | :--- |
| The record above, missing `disposition` | No effect. This is a structural failure, not a signature failure |
| Readback differs by one byte | No effect |
| The record is written after the effect has started | Non-compliant, even if the bytes later match |
| Storage acknowledgment without readback | Not sufficient |

---

## 3. Level 2 receipt

Requirement IDs: CAES-SIG-01 through CAES-SIG-07.

Protected header, JCS:

```text
{"alg":"EdDSA","kid":"fixture-ed25519-1","typ":"caes-receipt+jws"}
```

`alg` `none`, and any shared-secret `alg`, are non-compliant and grant no execution authority.

Payload, JCS (the signed payload bytes):

```text
{"action_descriptor":"noop","action_id":"act-1","audience":"executor","caes_version":"v0.2.0","disposition":"Approved","enforcement_outcome":"authorized","exp":"2026-03-01T00:15:00Z","iat":"2026-03-01T00:00:00Z","issuer":"fixture-issuer","matched_rules":[{"action":"allow","priority":0,"rule_id":"r1"}],"payload_hash":"8d720a027dd3ca100d7c6e92ca9a67e832f1ad67983f265aa02a64ade5c80f8d","policy_hash":"3a764d803e05908e335042db60b4fb03472b7fe33d6b45023995687e8a2190ef","policy_id":"pol-1","policy_version":"1","receipt_id":"rcpt-1","tenant_id":"tenant-a"}
```

Fixture Ed25519 seed, 32 bytes, hex (not a production key):

```text
1111111111111111111111111111111111111111111111111111111111111111
```

Public key, 32 bytes, hex:

```text
d04ab232742bb4ab3a1368bd4615e4e6d0224ab71a016baf8520a332c9778737
```

Signing input, ASCII:

```text
eyJhbGciOiJFZERTQSIsImtpZCI6ImZpeHR1cmUtZWQyNTUxOS0xIiwidHlwIjoiY2Flcy1yZWNlaXB0K2p3cyJ9.eyJhY3Rpb25fZGVzY3JpcHRvciI6Im5vb3AiLCJhY3Rpb25faWQiOiJhY3QtMSIsImF1ZGllbmNlIjoiZXhlY3V0b3IiLCJjYWVzX3ZlcnNpb24iOiJ2MC4yLjAiLCJkaXNwb3NpdGlvbiI6IkFwcHJvdmVkIiwiZW5mb3JjZW1lbnRfb3V0Y29tZSI6ImF1dGhvcml6ZWQiLCJleHAiOiIyMDI2LTAzLTAxVDAwOjE1OjAwWiIsImlhdCI6IjIwMjYtMDMtMDFUMDA6MDA6MDBaIiwiaXNzdWVyIjoiZml4dHVyZS1pc3N1ZXIiLCJtYXRjaGVkX3J1bGVzIjpbeyJhY3Rpb24iOiJhbGxvdyIsInByaW9yaXR5IjowLCJydWxlX2lkIjoicjEifV0sInBheWxvYWRfaGFzaCI6IjhkNzIwYTAyN2RkM2NhMTAwZDdjNmU5MmNhOWE2N2U4MzJmMWFkNjc5ODNmMjY1YWEwMmE2NGFkZTVjODBmOGQiLCJwb2xpY3lfaGFzaCI6IjNhNzY0ZDgwM2UwNTkwOGUzMzUwNDJkYjYwYjRmYjAzNDcyYjdmZTMzZDZiNDUwMjM5OTU2ODdlOGEyMTkwZWYiLCJwb2xpY3lfaWQiOiJwb2wtMSIsInBvbGljeV92ZXJzaW9uIjoiMSIsInJlY2VpcHRfaWQiOiJyY3B0LTEiLCJ0ZW5hbnRfaWQiOiJ0ZW5hbnQtYSJ9
```

Signature, base64url, no padding:

```text
flIzmVqPk_rGSck8ZSMyx9T8ycIcak0a_L6HzpZwLBR7EB4PolZ1v8L60xPxkI_6bDWIt73udS00QU4sKmPeCQ
```

Flattened JWS:

```json
{
  "payload": "eyJhY3Rpb25fZGVzY3JpcHRvciI6Im5vb3AiLCJhY3Rpb25faWQiOiJhY3QtMSIsImF1ZGllbmNlIjoiZXhlY3V0b3IiLCJjYWVzX3ZlcnNpb24iOiJ2MC4yLjAiLCJkaXNwb3NpdGlvbiI6IkFwcHJvdmVkIiwiZW5mb3JjZW1lbnRfb3V0Y29tZSI6ImF1dGhvcml6ZWQiLCJleHAiOiIyMDI2LTAzLTAxVDAwOjE1OjAwWiIsImlhdCI6IjIwMjYtMDMtMDFUMDA6MDA6MDBaIiwiaXNzdWVyIjoiZml4dHVyZS1pc3N1ZXIiLCJtYXRjaGVkX3J1bGVzIjpbeyJhY3Rpb24iOiJhbGxvdyIsInByaW9yaXR5IjowLCJydWxlX2lkIjoicjEifV0sInBheWxvYWRfaGFzaCI6IjhkNzIwYTAyN2RkM2NhMTAwZDdjNmU5MmNhOWE2N2U4MzJmMWFkNjc5ODNmMjY1YWEwMmE2NGFkZTVjODBmOGQiLCJwb2xpY3lfaGFzaCI6IjNhNzY0ZDgwM2UwNTkwOGUzMzUwNDJkYjYwYjRmYjAzNDcyYjdmZTMzZDZiNDUwMjM5OTU2ODdlOGEyMTkwZWYiLCJwb2xpY3lfaWQiOiJwb2wtMSIsInBvbGljeV92ZXJzaW9uIjoiMSIsInJlY2VpcHRfaWQiOiJyY3B0LTEiLCJ0ZW5hbnRfaWQiOiJ0ZW5hbnQtYSJ9",
  "protected": "eyJhbGciOiJFZERTQSIsImtpZCI6ImZpeHR1cmUtZWQyNTUxOS0xIiwidHlwIjoiY2Flcy1yZWNlaXB0K2p3cyJ9",
  "signature": "flIzmVqPk_rGSck8ZSMyx9T8ycIcak0a_L6HzpZwLBR7EB4PolZ1v8L60xPxkI_6bDWIt73udS00QU4sKmPeCQ"
}
```

| Case | Expected |
| :--- | :--- |
| Signature verifies under the fixture public key and `kid` `fixture-ed25519-1` | Signature match |
| One bit of the signature is flipped | No execution authority |
| `kid` is unknown or revoked | No execution authority |
| Either key inside a published rotation overlap, and `iat` is inside that window | MAY be accepted. Overlap alone is not a denial |
| `exp` is `2026-03-01T00:15:00Z` and `iat` is `2026-03-01T00:00:00Z` | Lifetime is 15 minutes. Inside the interchange maximum |
| `exp` is `2026-03-01T00:15:01Z` with the same `iat` | Lifetime is 901 seconds. No interchange execution authority |
| Declared skew above 120 seconds, or clock accuracy unknown | No execution authority. Archival verification MAY report a signature match and MUST label the time untrusted |
| RFC 3161 token absent, and the Conformance Statement does not claim trusted time | Acceptable. `iat` is still required |
| Verification after `exp` | The archived signature may still match. The verifier MUST NOT report that the authority is unused |

`exp - iat` was checked when this file was generated: 900 seconds is the boundary, 901 seconds is outside it.

---

## 4. Effect composition

Requirement IDs: CAES-EFF-01, CAES-EFF-02, CPP-EFF-01.

Action `act-ext` lists `ExternalSideEffect`, `GovernanceRelevantState`, and `WorkflowTransition`. One CPP `effect_type` of `ExternalInvocation` applies to all three. Evaluation MUST include every eligible policy for any of those categories, then apply CPP conflict resolution.

| Case | Expected |
| :--- | :--- |
| Policies exist for all three categories and conflict resolution yields `allow` | Disposition `Approved`. The effect still needs live execution authority |
| A policy is dropped because the author named only `ExternalInvocation` and ignored governance state | Non-compliant |
| The action also lists `SafetyCriticalActuation` | Denied. This version has no CPP `effect_type` for that category |
| `effect_type` is not in the CPP set | Fail closed |

---

## 5. Tenants and dispositions

Requirement IDs: CPP-TEN-01, CPP-ACT-01, CPP-ACT-02, CPP-ACT-03, CPP-ACT-04.

| Input | Expected |
| :--- | :--- |
| Policy tenant `Tenant-A`, request tenant `tenant-a` | No match. No case folding |
| Policy tenant `*` with a delegation scoped `cross_tenant_policy` | Eligible wildcard |
| Policy tenant `*` without that delegation | Policy ineligible |
| Tenant identifier `*` | Rejected. Identifiers MUST NOT be `*` |
| `allow`, authority reserved and inside `exp` | Effect may proceed |
| `allow`, `exp` passed before consumption | No effect. Disposition stays `Approved` |
| `deny` | No effect |
| `modify`, and the executor runs the pre-transform payload | Non-compliant |
| `escalate` with no later execution authority | No effect. The disposition is not authority |

---

## 6. Obligations

Requirement IDs: CAES-OBL-01, CPP-OBL-01, CPP-OBL-02, CPP-OBL-03.

| Phase | Failure | Disposition | Authority | Outcome evidence |
| :--- | :--- | :--- | :--- | :--- |
| `pre_effect` | Obligation fails | Stays `Approved` if that was the policy result | Not granted | Records the block |
| `continuous` | Obligation fails after some bytes | Unchanged | Further covered bytes stop | Released bytes remain |
| `post_effect` | Obligation fails after completion | Unchanged | Not retroactively removed | Remediation duty. The effect is not described as absent |

---

## 7. Streaming

Requirement IDs: CAES-STR-01, CAES-STR-02, CAES-STR-03.

Emission is the first governed-content byte, including caller-visible metadata that carries governed content. A heartbeat that cannot carry governed content is not emission.

| Mode | Pass | Fail |
| :--- | :--- | :--- |
| `buffered` | No governed byte leaves until one receipt covers the complete output | Any governed byte leaves first |
| `segmented` | Each segment is defined, hashed, and covered before its first byte | A later completion record is treated as authorization for bytes already sent |
| Either | The Conformance Statement names the mode | The mode is unnamed |

---

## 8. Replay lifecycle

Requirement IDs: CAES-REP-01, CAES-REP-02, CAES-REP-03.

Reservation tuple: `(issuer, tenant_id, audience, action_id, payload_hash, receipt_id)`.

Worked tuple for the section 3 receipt: `("fixture-issuer", "tenant-a", "executor", "act-1", "8d720a027dd3ca100d7c6e92ca9a67e832f1ad67983f265aa02a64ade5c80f8d", "rcpt-1")`.

| Step | Expected state | Effect |
| :--- | :--- | :--- |
| Atomic durable reserve, inside `exp`, before any effect | Reserved | Not started |
| Crash before any effect, reservation released | Abandoned | None. Same receipt MAY retry inside `exp` |
| Effect completes and completion is recorded | Consumed | One effect |
| Second reserve of the same tuple | No new execution | Original evidence still verifies |
| Crash after the effect and before completion | Uncertain | Do not repeat with a fresh receipt until reconciliation shows the effect did not occur. Further execution of that action is blocked |
| `exp` passes before consumption | Expired | No effect |

Offline verification of the section 3 receipt MUST NOT report that this tuple is unused.

---

## 9. Chaos modes

Requirement IDs: CAES-CH-00, CAES-L3-01, CAES-L3-02.

Each row is a separate test. Passing one row does not pass another. Two rows at once still require every applicable prevention and every applicable evidence record. Integrity outranks availability.

| Code | Pass condition |
| :--- | :--- |
| `CAES-E-DB-WRITE` | The effect is absent, including when the evidence write itself failed |
| `CAES-E-KEY-ROTATION` | A signature in the published overlap is not denied. A revoked or unknown `kid` grants no authority |
| `CAES-E-CLOCK-SKEW` | No effect outside the declared skew bound. A time-untrusted archival result may still be recorded |
| `CAES-E-PACK-PARTITION` | The partial export fails verification and does not look sealed |
| `CAES-E-PACK-DISK` | The incomplete pack fails verification and has no success marker |
| `CAES-E-RECEIPT-REPLAY` | At most one effect. The original receipt still verifies |
| `CAES-E-POLICYHASH-MISMATCH` | No effect. The mismatch is in the outcome evidence |
| `CAES-E-DELEGATION-EXPIRY` | No effect. The code is recorded |
| `BINDING_MISSING` | A delegation with no target binding is rejected with this code. It does not stand in for the eight rows |

---

## 10. Evidence Pack

Requirement ID: CAES-PAK-01.

A Level 2 or Level 3 pack MUST conform to `governance/CAES_EVIDENCE_PACK_SPEC.md` version `0.2.0`.

| Case | Expected |
| :--- | :--- |
| Required entry missing | Verification fails |
| Verifier calls a private issuer service | Non-compliant verification |
| Archive verifies offline | MUST NOT be reported as "this authorization is still unused" |
| Partial ZIP that still parses | Not a sealed pack |
| v0.1.1 layout `/receipts/.json` | Does not satisfy pack spec 0.2.0 |

---

## 11. What these fixtures do not show

Passing every row in one implementation does not show that a second implementation accepts the same receipt. An interoperability claim still needs that independent evidence. These fixtures are the gate such a claim has to meet. They are not the claim.
