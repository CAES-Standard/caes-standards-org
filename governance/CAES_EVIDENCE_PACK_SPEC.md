# CAES Evidence Pack Format Specification

**Applies To:** CAES v0.1.1  
**Status:** Draft  
**Category:** Normative Artifact Specification  

---

## 1. Purpose

A CAES Evidence Pack is a self-contained, cryptographically verifiable artifact that allows an independent party to validate:

- Decision Receipts
- PolicyHash determinism
- Governed Spine integrity
- Test Suite results
- Claimed compliance level

Verification must be possible offline.

---

## 2. Design Requirements

A CAES Evidence Pack MUST:

- Be portable
- Be deterministic in encoding
- Be cryptographically verifiable
- Not require access to production systems
- Declare CAES version
- Declare compliance level

---

## 3. Archive Format

Evidence Packs MUST be distributed as:

ZIP archive with deterministic encoding.

Identical contents MUST produce byte-identical archives.

---

## 4. Required Files

```text
/manifest.json
/receipts/.json
/spine.json
/policies/.json
/test-results.json
/trust-bundle.json
/hashes/sha256sum.txt
```


---

## 5. manifest.json (Required)

Must include:

- caes_version
- compliance_level
- implementation_id
- test_suite_version
- pack_timestamp
- pack_id (deterministic hash)
- signature

The manifest MUST be signed.

---

## 6. Decision Receipts

Each receipt MUST:

- Be individually signed
- Include PolicyHash
- Include timestamp
- Include action identity
- Be verifiable via trust bundle

---

## 7. Governed Spine

spine.json MUST:

- Be append-only ordered
- Include event IDs
- Include parent references
- Include tenant scope
- Include hash commitments

Tampering must invalidate verification.

---

## 8. Policy Artifacts

Policies included MUST:

- Be canonical representations
- Match PolicyHash values
- Enable independent recomputation

---

## 9. Test Results

test-results.json MUST:

- Map to CAES requirement IDs
- Include pass/fail per test
- Include run identifier
- Include timestamp
- Be signed

---

## 10. trust-bundle.json

Must contain:

- Public keys used for signature verification
- Key identifiers
- Validity bounds

---

## 11. Hash Index

sha256sum.txt MUST:

- Include hash of every file in archive
- Exclude itself
- Be referenced in manifest

---

## 12. Verification Requirements

A verifier MUST be able to:

1. Verify archive integrity.
2. Verify manifest signature.
3. Verify each receipt signature.
4. Recompute PolicyHash values.
5. Validate spine invariants.
6. Validate test result mapping.

Failure in any step invalidates pack.

---

CAES-WG  
March 2026