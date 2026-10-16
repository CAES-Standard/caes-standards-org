# CAES CLI Validator Specification

**Applies To:** CAES v0.1.1
**Revision note:** Not yet revised for CAES v0.2.0.
**Category:** Reference Verification Interface  

---

## 1. Purpose

The CAES CLI Validator is a deterministic verification tool that validates CAES Evidence Packs.

It does not certify implementations.
It verifies structural compliance.

---

## 2. Command Interface

Required command:

```bash
caes verify <evidence-pack.zip>
```

Optional:

```bash
caes inspect <evidence-pack.zip>
caes summary <evidence-pack.zip>
```

---

## 3. Verification Steps

The CLI MUST:

1. Validate ZIP determinism.
2. Verify sha256sum.txt.
3. Verify manifest signature.
4. Verify receipt signatures.
5. Recompute PolicyHash values.
6. Validate spine causal invariants.
7. Validate test suite mapping.
8. Confirm compliance level requirements satisfied.

---

## 4. Output Requirements

CLI MUST produce:

- Machine-readable JSON output
- Human-readable summary
- Exit code 0 on success
- Non-zero exit on failure

Output MUST include:

- caes_version
- compliance_level
- verification_result
- failed_checks (if any)
- timestamp

---

## 5. Determinism

Given identical input archive,
CLI output MUST be identical.

No network calls permitted during verification.

---

## 6. Trust Root Handling

CLI MUST:

- Accept trust-bundle.json from pack
- Allow override of trust root via CLI flag
- Reject unknown keys

---

## 7. Future Extensions

Future versions may support:

- Remote registry cross-check
- Revocation lists
- Conformance badge validation

---

CAES-WG  
March 2026