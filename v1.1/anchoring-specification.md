# Anchoring Specification — Version 1.1

> **Status:** Normative  
> **Version:** 1.1 (Current)  
> **Supersedes:** v1.0 (frozen, permanently available at `/v1.0/`)  
> **Canonical URL:** https://anchoring-spec.org/v1.1/  
> **License:** Public Domain (Unlicense)

The key words MUST, SHOULD, and MAY in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

---

## 0. Changelog (Non-Normative)

This is a **MINOR** revision of v1.0. No semantic, cryptographic, or interoperability changes have been made. Implementations conformant with v1.0 remain conformant with v1.1 without modification.

| Change | Type | Reason |
|---|---|---|
| Document title changed from "Anchoring Specification (IEC)" to "Anchoring Specification" | Editorial | The abbreviation "IEC" caused naming collision with the International Electrotechnical Commission. The semantic model retains the name *Independent External Chronology*; only the parenthetical from the document title is dropped. |
| §1 expanded to define *Independent External Chronology* explicitly as the name of the semantic model | Editorial | Clarification. |
| §12 (Governance) rewritten to accurately reflect single-maintainer status and forkability | Editorial | Honest disclosure. No change to specification behaviour. |
| Citation form updated (see §19) | Editorial | Reflects new title. |

v1.0 remains permanently citable at `https://anchoring-spec.org/v1.0/` and will not be modified.

---

## 1. Purpose

This specification defines **anchoring**: an infrastructure primitive built on cryptographic commitments.

The semantic model formalised by this specification is named **Independent External Chronology**. An anchoring proof is a witness that a specific byte sequence existed at or before a time established by a chronology that is independent of the proof issuer and external to any single party's infrastructure.

Anchoring establishes one property only:

> **The exact bytes of an artifact existed on or before time T.**

No additional claims are made.

---

## 2. Scope

Anchoring is defined as:

> The process of creating a cryptographic commitment to a byte sequence and binding that commitment to a publicly verifiable, independent, external time reference.

This specification defines the verification semantics, proof structure, and ledger requirements for anchoring.

---

## 3. Definitions

| Term | Definition |
|------|-----------|
| Artifact | An arbitrary byte sequence submitted for anchoring |
| Hash | The output of a cryptographic hash function applied to the artifact |
| Proof | A structured bundle containing the hash, timestamp, and ledger binding |
| Ledger | A publicly verifiable, append-only record used as time reference |
| Anchoring | The complete process from hash computation to ledger binding |
| Independent External Chronology | The semantic model: a time ordering established by infrastructure that is independent of the proof issuer and external to any single party |

---

## 4. Verification Function

The verification function is defined as:

```
V(B, P, L) → { valid | invalid | unverifiable }
```

Where:
- **B** = artifact byte sequence
- **P** = proof bundle
- **L** = publicly verifiable ledger infrastructure

---

## 5. Output Semantics

| Output | Condition |
|--------|-----------|
| **valid** | Hash of B matches P, and P is bound to L with verified time T |
| **invalid** | Hash of B does not match P, or P contains inconsistent data |
| **unverifiable** | P cannot be validated against L (e.g., ledger unavailable, proof incomplete) |

No other outputs are permitted for a conformant implementation.

---

## 6. Proof Structure

A conformant proof bundle MUST contain:

1. The hash of the artifact
2. The hash algorithm identifier
3. A timestamp or time reference
4. A ledger binding (e.g., OpenTimestamps proof, blockchain transaction)

A conformant proof bundle MAY contain:

- The original artifact
- Metadata (origin identifier, capture context)
- Additional attestations

---

## 7. Ledger Qualification

A qualified ledger MUST:

1. Be publicly accessible without permission
2. Be append-only (no retroactive modification)
3. Provide independently verifiable time ordering
4. Not be controlled by the proof issuer

No single controlling authority (including the proof issuer) MUST be able to rewrite historical state or timestamps.

---

## 8. Semantic Exclusions

Anchoring explicitly does NOT establish:

- **Authorship** — who created the artifact
- **Ownership** — who has rights to the artifact
- **Originality** — whether the artifact is novel
- **Identity** — who submitted the artifact
- **Intent** — why the artifact was anchored
- **Legal enforceability** — whether the proof has legal standing
- **Truthfulness** — whether the artifact content is accurate

These exclusions are non-negotiable.

---

## 9. Independence Requirement

A conformant proof MUST be verifiable without reliance on:

- the issuer of the proof
- any infrastructure operated by the issuer
- any account or credential system

Verification must be achievable using:

- the proof bundle
- publicly available cryptographic tools
- the public ledger

If a proof requires issuer-controlled infrastructure for verification, it is non-conformant.

---

## 10. Threat Model

The specification assumes the following threat model:

- The artifact submitter may be adversarial
- The proof issuer may become unavailable
- Network infrastructure may be unreliable
- Time sources may be manipulated (mitigated by ledger consensus)

The specification does NOT protect against:

- Pre-computation attacks (submitting a hash before creating the artifact)
- Collision attacks on the hash function (mitigated by algorithm requirements)

---

## 11. Cryptographic Requirements

- Hash algorithms MUST provide collision resistance appropriate to the security level
- SHA-256 is the reference algorithm for this version
- Algorithm agility: implementations SHOULD support algorithm migration without breaking existing proofs
- The hash algorithm identifier MUST be included in the proof bundle

---

## 12. Governance

The Anchoring Specification is a public, versioned technical specification, released into the public domain.

It is currently authored and maintained by the Umarise team. There is no foundation, consortium, or multi-stakeholder governance body. This is a single-maintainer specification.

The specification is open for implementation by any party without restriction. Any party MAY fork, mirror, or republish the specification under any terms permitted by the public domain. No implementation has normative authority over the specification.

External co-maintainers and multi-stakeholder governance are welcome and may be adopted if and when adoption warrants it. Until then, governance independence is not claimed. Technical independence (the property that proofs verify without the issuer) is established by §9 and is independent of governance.

---

## 13. Legal Scope

This specification defines technical semantics only. It makes no claims about:

- legal admissibility in any jurisdiction
- regulatory compliance
- contractual obligations

Legal interpretation of anchoring proofs is outside the scope of this specification.

---

## 14. Time Semantics

Time in anchoring refers to ledger consensus time, not wall-clock time.

- The timestamp represents the time at which the ledger confirmed the binding
- Clock skew between submission and confirmation is expected
- The specification establishes "at or before T," not "exactly at T"

---

## 15. Non-Retroactivity

A conformant proof demonstrates existence at or before time T. It does not demonstrate:

- existence before a specific earlier time
- continuous existence
- non-existence before T

---

## 16. Compliance Statements

An implementation claiming conformance with this specification MUST:

1. Implement the verification function as defined in Section 4
2. Produce only the outputs defined in Section 5
3. Respect all semantic exclusions defined in Section 8
4. Satisfy the independence requirement defined in Section 9
5. State the specification version implemented (e.g., "Anchoring Specification v1.1")
6. Include the disclaimer: "The specification is normative. This implementation is not."

---

## 17. Non-Conformance

An implementation is non-conformant if it:

- produces outputs not defined in Section 5
- claims to establish any property excluded in Section 8
- requires issuer-controlled infrastructure in violation of Section 9

---

## 18. Archival Considerations

Long-term verifiability depends on:

- preservation of proof bundle integrity
- continued availability of ledger validation data
- continued availability of cryptographic verification algorithms

Anchoring proofs SHOULD be stored in durable, non-proprietary formats.

---

## 19. Citation

```
Anchoring Specification, Version 1.1.
Canonical URL: https://anchoring-spec.org/v1.1/
License: Public Domain (Unlicense).
```

BibTeX:
```bibtex
@misc{anchoring-spec-v1-1,
  title  = {Anchoring Specification},
  version = {1.1},
  year   = {2026},
  url    = {https://anchoring-spec.org/v1.1/},
  note   = {Public domain. Normative specification for cryptographic anchoring. Semantic model: Independent External Chronology.}
}
```

---

*Canonical publication: [anchoring-spec.org/v1.1/](https://anchoring-spec.org/v1.1/)*  
*The specification is normative. Implementations are not.*
