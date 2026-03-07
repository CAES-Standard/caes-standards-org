# CAES Public Conformance Registry Model

**Status:** Draft  
**Applies To:** CAES v0.1.1  

---

## 1. Purpose

The CAES Public Conformance Registry provides a public listing of implementations that have published complete Conformance Statements and Evidence Packs.

Registry inclusion does not imply certification.
It provides discoverability.

---

## 2. Registry Requirements

An implementation may be listed if it publishes:

- Complete Conformance Statement
- Evidence Pack
- Test Suite results
- CAES version cited
- Compliance level declared

---

## 3. Registry Fields

Each entry MUST include:

- Implementation name
- Implementation identifier
- CAES version
- Compliance level
- Evidence pack URL
- Test suite version
- Last verification date

---

## 4. Registry Verification

Registry MAY:

- Run CAES CLI validation
- Publish verification status
- Indicate verification date

Registry MUST NOT:

- Modify evidence artifacts
- Imply legal certification

---

## 5. Revocation

Registry entry may be marked inactive if:

- Evidence pack fails verification
- Version superseded without update
- Conformance statement withdrawn

---

## 6. Transparency

Registry entries MUST:

- Link to public evidence artifacts
- Identify version tested
- Declare if self-attested or independently validated

---

## 7. Future Evolution

Future CAES versions may introduce:

- Automated registry verification pipeline
- Public conformance API
- Badge validation endpoints

---

CAES-WG  
March 2026