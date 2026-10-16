# CAES Conformance Statement Template

Implementations claiming CAES compliance MUST publish a complete Conformance Statement.

---

## Required Fields

### CAES Version
(e.g., v0.2.0)

### Claimed Compliance Level
Level 1 / Level 2 / Level 3

### Effect Boundary Categories Governed
(List every normative Effect Boundary category governed. v0.2.0 defines seven categories. An implementation must not narrow that set.)

### Receipt Verification Method
For Level 2 and Level 3, name `EdDSA` or `ES256`, the `kid` trust anchors, and how unknown or revoked keys are rejected. Level 1 states that signature verification is not the Level 1 check.

### PolicyHash Profile
For Level 2 and Level 3: effective policy bundle, RFC 8785, SHA-256, lowercase hex. Level 1 states that PolicyHash is not required.

### Maximum Execution Lifetime
`exp - iat`. At most 15 minutes for an interchange claim.

### Maximum Clock Skew
At most 120 seconds for an interchange claim. State whether trusted time (RFC 3161) is claimed. A signature is not a claim that the clock was accurate.

### Stream Mode
`buffered` or `segmented`.

### Evidence Pack Version
`0.2.0` when Level 2 or Level 3 is claimed.

### Fail-Closed Enforcement Mechanism
Describe enforcement model, including how an Approved disposition can still be blocked.

### Spine Storage Model
Level 2 and Level 3: describe append-only guarantees and immutability controls. Level 1: state that a spine append is not required.

---

## Optional

- Public evidence pack
- Independent audit
- Test harness results

---

Incomplete statements MUST NOT reference CAES compliance.