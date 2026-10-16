# CAES Evidence Pack Format Specification

**Applies To:** CAES v0.2.0 Level 2 and Level 3
**Pack spec version:** 0.2.0
**Status:** Normative annex
**Requirement:** CAES-PAK-01
**Category:** Normative Artifact Specification

This document is the Evidence Pack annex cited by the CAES v0.2.0 public draft. A Level 1 claim does not require a pack. A Level 2 or Level 3 claim MUST conform to this version. A pack that says it is v0.1.1, or that omits `pack_spec_version`, does not satisfy CAES-PAK-01.

---

## 1. Purpose

A CAES Evidence Pack is a self-contained artifact that lets an independent party check, without calling the issuer:

- Decision Receipt signatures and claims
- PolicyHash recomputation from the included effective policy bundles
- Governed Spine structure
- Test results mapped to requirement IDs
- The claimed compliance level

Offline verification MUST NOT contact a private issuer service. Offline success means the recorded artifact checks. It MUST NOT be reported as "this authorization is still unused." Consumption state is an online enforcement fact (CAES-REP-03).

If a pack includes a consumption snapshot, the snapshot MUST be labeled as a point-in-time copy. It MUST NOT be described as proof that the authority is unused now.

---

## 2. Design requirements

A pack MUST:

- Be portable
- Be deterministic in encoding, under the recipe in section 3
- Be verifiable from the trust bundle inside the pack
- Not require access to production systems
- Declare `caes_version` and `pack_spec_version`
- Declare the compliance level
- Fail verification if any required entry is missing, truncated, or altered

An incomplete export MUST NOT carry a success marker, a manifest signature over a partial hash index, or any other signal that a verifier could read as a sealed pack. That rule covers network partition and disk-full during export (`CAES-E-PACK-PARTITION`, `CAES-E-PACK-DISK`).

When integrity and availability conflict, integrity wins. A pack MUST NOT be completed by dropping a receipt, a policy bundle, or a failure record in order to finish the archive (CAES-CH-00).

---

## 3. Archive format

Packs MUST be ZIP archives using compression method 0 (stored) for every entry.

Identical entry bytes MUST produce byte-identical archives. The recipe is:

- Entry names use `/` separators and no leading slash.
- Names are compared as raw UTF-8 bytes and written in ascending order.
- Each local header and central-directory header uses date and time `1980-01-01 00:00:00`.
- Extra fields are empty. The archive comment is empty.
- Data descriptors are not used. Compressed size and uncompressed size are written in the local header.
- The UTF-8 language-encoding flag is clear when the name is ASCII. It is set when the name is not ASCII.
- External attributes are zero.

`pack_timestamp` inside the manifest is evidence content. It is not the ZIP timestamp, and it is not a trusted-time proof.

---

## 4. Required entries

```text
/manifest.json
/receipts/<receipt_id>.json
/spine.json
/policies/<policy_id>.json
/test-results.json
/trust-bundle.json
/hashes/sha256sum.txt
```

Level 3 packs MUST also include:

```text
/delegation-chains.json
/chaos-attestation.json
```

`receipt_id` and `policy_id` are the identifier strings from those objects, restricted to unreserved path characters. A pack with no receipts, or with a receipt whose policy bundle is absent, fails verification.

The v0.1.1 draft listed `/receipts/.json` and `/policies/.json`. Those paths are not the 0.2.0 layout.

---

## 5. manifest.json

The manifest payload is a JSON object. Its JCS encoding is what gets signed. Required members:

- `caes_version` — `v0.2.0` for this annex
- `pack_spec_version` — `0.2.0`
- `compliance_level` — `2` or `3`
- `implementation_id`
- `test_suite_version`
- `pack_timestamp` — RFC 3339 with `Z` or a numeric offset, signer-asserted
- `pack_id` — lowercase-hex SHA-256 of the UTF-8 bytes of `/hashes/sha256sum.txt`
- `hash_index_path` — `hashes/sha256sum.txt`

The manifest file is a flattened JWS JSON document (RFC 7515) with one signature. The protected header MUST contain `alg`, `kid`, and `typ` with value `caes-pack+jws`. The payload is the JCS encoding of the manifest object. Interchange algorithms are `EdDSA` (Ed25519) and `ES256`. `none` and shared-secret algorithms are non-compliant.

`kid` MUST resolve in `trust-bundle.json`. An unknown or revoked `kid` fails verification. A published rotation overlap recorded in the trust bundle MAY verify under either key that the overlap names.

---

## 6. Decision Receipts

Each file under `/receipts/` MUST be one flattened JWS receipt as defined by CAES-SIG-02 through CAES-SIG-07.

Verification MUST check the signature, the required claims, `payload_hash`, and, at Level 2 and Level 3, `policy_hash` against the included bundle. It MUST NOT treat a valid signature as proof that the effect has not yet run.

---

## 7. Governed Spine

`spine.json` MUST:

- List entries in append order
- Include event IDs and tenant scope
- Include a parent reference where the CAES draft requires one
- Include a hash commitment over each entry's canonical payload

Changing an entry, reordering entries, or dropping an entry MUST invalidate verification. The pack copy is evidence of the spine at export. It does not authorize a new effect.

---

## 8. Policy artifacts

Each policy file MUST be the effective policy bundle whose JCS SHA-256 matches the `policy_hash` of the receipts that cite it. A verifier recomputes that hash offline. Runtime context is not part of the bundle.

---

## 9. Test results

`test-results.json` MUST:

- Name the CAES version and the compliance level tested
- Map each result to a requirement ID from the v0.2.0 draft or from CPP v1.0
- Include pass or fail, and the pass condition for chaos rows
- Include a run identifier and an RFC 3339 timestamp

A Level 2 or Level 3 test-result file MUST be signed with the same JWS profile as the manifest, with `typ` value `caes-test-results+jws`. A shared error code is not a pass for a chaos mode. The result MUST record the mode's own pass condition from the CAES chaos table.

Level 3 `chaos-attestation.json` MUST contain one object per mode:

| Mode | Code |
| :--- | :--- |
| DB write failure | `CAES-E-DB-WRITE` |
| Key rotation conflict | `CAES-E-KEY-ROTATION` |
| Clock skew | `CAES-E-CLOCK-SKEW` |
| Network partition during pack export | `CAES-E-PACK-PARTITION` |
| Disk full during pack write | `CAES-E-PACK-DISK` |
| Duplicate receipt injection | `CAES-E-RECEIPT-REPLAY` |
| PolicyHash mismatch | `CAES-E-POLICYHASH-MISMATCH` |
| Delegation chain expiry | `CAES-E-DELEGATION-EXPIRY` |

Each object records the observed behavior and whether that mode's pass condition held. `BINDING_MISSING` remains the code when a delegation omits its target binding (CAES-L3-02). It is not a substitute for the eight rows.

---

## 10. trust-bundle.json

The trust bundle MUST contain the public keys needed to verify the pack, as JWK or as raw key bytes with an explicit `kty` and `crv`. For each key:

- `kid`
- validity bounds, as RFC 3339 timestamps
- status `active`, `overlap`, or `revoked`

A revoked key MUST NOT verify a receipt or a manifest. An `overlap` key MAY verify a signature whose `iat` falls inside the published overlap window named on that key entry.

---

## 11. Hash index

`hashes/sha256sum.txt` MUST list every other entry in the archive. Lines are sorted by path. Each line is:

```text
<lowercase-hex SHA-256><two spaces><path>
```

The hash index does not list itself. The manifest `pack_id` is the hash of this file's exact bytes, including the final newline.

---

## 12. Verification steps

A verifier MUST, with no network call:

1. Reject a ZIP that breaks the section 3 recipe, including a partial archive that still parses.
2. Recompute every hash in `hashes/sha256sum.txt` and compare.
3. Verify the manifest signature over the JCS payload and check `pack_id`.
4. Verify each receipt signature and required claims.
5. Recompute each cited PolicyHash from the included bundle.
6. Check spine order, tenant, and parent references.
7. Check that test results name requirement IDs, and that Level 3 chaos attestation covers all eight modes with their own pass conditions.
8. Reject the pack if any required entry is missing.

Failure of any step invalidates the pack. Success still does not mean the recorded authority is unused.

---

CAES-WG
Pack spec 0.2.0
