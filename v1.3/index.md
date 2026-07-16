# Anchoring Specification

**Version** 1.3 (Current)  
**Status** Normative  
**Publication Date** 2026-07-15  
**Supersedes** v1.2 (frozen, available at /v1.2/)  
**Canonical URI** https://anchoring-spec.org/v1.3/

**v1.3 is a MINOR revision over v1.2.** One change: Section 6.1 is rewritten and the schema file it publishes is replaced. The type `AnchorProofBundle`, which modelled the internal structure of an OpenTimestamps proof in CDDL, is withdrawn; the published schema now defines only `ProofBundle`, the verifier input. No normative requirement of Sections 1 to 5 or 7 to 18 is changed, and the change does not affect the requirements of Section 6 as to what a proof bundle must contain. Implementations conformant with v1.0, v1.1 or v1.2 remain conformant with v1.3 without modification. v1.2, v1.1 and v1.0 remain permanently citable at /v1.2/, /v1.1/ and /v1.0/, including the withdrawn schema file at /v1.2/cddl/anchor-proof-bundle.cddl.

## Non-Normative Introduction

Anchoring is defined as an infrastructure primitive built on cryptographic commitments, for establishing existence-at-or-before-T of a byte sequence.

It provides a verifiable commitment to the existence of specific bytes at or prior to a consensus-derived timestamp, without asserting authorship, ownership, identity, or legal status.

This specification formally defines the semantics, verification function, and independence requirements of this primitive.

This introduction is non-normative. Normative requirements begin in Section 1.

## Contents

1. [Scope](#scope)
2. [Terminology](#terminology)
3. [Definition of Anchoring](#definition)
4. [Verification Function](#verification)
5. [Output Semantics](#outputs)
6. [Proof Structure Requirements](#proof-structure)
7. [Ledger Qualification Requirements](#ledger)
8. [Semantic Exclusions](#exclusions)
9. [Independence Requirement](#independence)
10. [Legal Scope](#legal)
11. [Versioning](#versioning)
12. [Governance](#governance)
13. [Threat Model](#threat-model)
14. [Cryptographic Requirements](#cryptographic)
15. [Time Semantics](#time-semantics)
16. [Non-Retroactivity](#non-retroactivity)
17. [Compliance Statement](#compliance)
18. [Archival Considerations](#archival)

## 1 Scope

This specification defines anchoring as a cryptographic method for establishing existence-at-or-before-T of a byte sequence.

This document normatively defines:

- the verification function
- output semantics
- proof structure requirements
- ledger qualification requirements
- independence requirements
- explicit semantic exclusions
- cryptographic requirements
- threat model
- time semantics
- non-retroactivity conditions
- compliance requirements
- archival considerations

**This specification is normative.**

## 2 Terminology

Artifact (B)
A finite byte sequence. B may represent any byte sequence, including derived artifacts. This specification does not interpret the semantic meaning of B.

Proof Bundle (P)
A structured set of data sufficient to verify anchoring claims.

Ledger Infrastructure (L)
A publicly verifiable consensus system capable of establishing temporal ordering.

Time (T)
A consensus-derived temporal reference associated with ledger inclusion of a commitment. T may be a structural identifier (such as a block height) or a wall-clock value derived from such an identifier. T must not be self-asserted by the issuing entity. Profile specifications define which form of T is normative for a given ledger; see §15.

## 3 Definition of Anchoring

Anchoring establishes one property only:  
A specific byte sequence existed on or before time T.

Existence refers solely to the presence of a cryptographic commitment to the byte sequence in ledger L at or prior to time T.

No additional claims are implied.

## 4 Verification Function

Verification must be defined as:

V(B, P, L) → { valid | invalid | unverifiable }

B
The exact byte sequence of the artifact under verification.

P
The proof bundle.

L
Publicly verifiable ledger infrastructure.

Output semantics are defined in Section 5.

**Note.** In the companion IETF profile (*draft-fassbender-scitt-time-anchor*), the verification function is instantiated with descriptive parameter names for clarity: V(ArtifactBytes, AnchorProof, AnchorLedger) where `ArtifactBytes` corresponds to `B`, `AnchorProof` corresponds to `P`, and `AnchorLedger` corresponds to `L`, restricted to ledgers meeting the qualification criteria of the profile. The semantics are identical. The short-form notation `V(B, P, L)` is retained in this specification as the abstract, implementation-neutral form. Implementations may use either notation provided the semantics defined in Section 5 are preserved. The IETF profile additionally specifies its normative temporal binding as the Bitcoin block height containing the OP_RETURN commitment, with `block.time` provided as a Reference Wall-Clock Projection for human readability. This is one valid instantiation of the abstract Time T defined in §15.

## 5 Output Semantics

An implementation must return exactly one of the following outputs:

| Output       | Condition                                                                                                                                      |
| ------------ | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| valid        | All required conditions hold: Hash(B) equals the hash stated in P. Ledger inclusion proof is valid under L. Ledger timestamp T is derivable.   |
| invalid      | At least one required condition fails, including: hash mismatch, invalid ledger proof, or corrupted/malformed proof structure.                 |
| unverifiable | Verification cannot be completed due to: missing artifact, missing proof data, inaccessible ledger infrastructure, or unsupported ledger type. |

Implementations must evaluate all required conditions prior to returning "valid".

Implementations must not return intermediate, partial, or qualified results outside this set.

## 6 Proof Structure Requirements

A compliant proof bundle must include:

- a cryptographic hash of B
- a timestamp claim
- a ledger inclusion proof or commitment

Optional:

- attestation metadata (non-normative)

Attestation metadata is explicitly non-normative and has no effect on verification outcome under this specification.

### 6.1 Schema Files

A machine-retrievable CDDL ([RFC 8610](https://www.rfc-editor.org/info/rfc8610)) schema for the verifier input is published at:

- **CDDL schema**: [`/v1.3/cddl/proof-bundle.cddl`](https://anchoring-spec.org/v1.3/cddl/proof-bundle.cddl)

It defines one type, `ProofBundle`: the structure passed to a verifier. The ledger inclusion proof is carried within it as an opaque byte string.

**Scope of the schema.** CDDL describes CBOR and JSON data models. The internal encoding of a ledger inclusion proof is defined by the proof format in use, and is not described by this schema or by this specification. A profile specification states which proof format applies and where its encoding is defined.

**Withdrawal note.** Versions 1.2 and earlier published a schema defining a second type, `AnchorProofBundle`, which modelled the internal structure of an OpenTimestamps proof. That type is withdrawn. A CDDL model of a binary wire format is not a schema for that format: it cannot express the format's framing, tags or branching, and presenting one as though it could invites implementations to be built against a model the format does not follow. The withdrawn schema remains available at [`/v1.2/cddl/anchor-proof-bundle.cddl`](https://anchoring-spec.org/v1.2/cddl/anchor-proof-bundle.cddl) for citation. No requirement of Section 6 depended on it, and no implementation is made non-conformant by its withdrawal.

## 7 Ledger Qualification Requirements

A ledger L qualifies if it:

- provides public verifiability
- provides consensus-based ordering
- resists unilateral timestamp rewriting
- allows reconstruction of historical state

Permissioned or centrally mutable systems do not qualify unless their historical state is independently verifiable without reliance on a single controlling authority.

This specification is ledger-agnostic.

## 8 Semantic Exclusions

Anchoring does not establish:

- authorship
- ownership
- originality
- identity
- intent
- legal enforceability
- truthfulness

Claims beyond existence-at-or-before-T are out of scope.

An implementation must not present anchoring results in a manner that implies any excluded claim.

## 9 Independence Requirement

A compliant anchoring proof must remain verifiable without reliance on the issuing entity.

Verification must not require:

- issuer-controlled databases
- issuer-controlled accounts
- issuer-controlled infrastructure

Practical convenience of verification tools does not affect compliance. Verification must be technically achievable without issuer-controlled infrastructure.

If verification depends exclusively on the issuer, the proof is non-conformant.

## 10 Legal Scope

This specification defines technical verification semantics only.

It does not create legal rights, obligations, or guarantees.

Legal interpretation of anchoring proofs depends on applicable jurisdiction and independent evaluation.

## 11 Versioning

This specification uses **MAJOR.MINOR** versioning.

- **MAJOR** increments indicate breaking semantic changes
- **MINOR** increments indicate clarifications without semantic change

All published versions must remain permanently accessible at their canonical URI.

Current version: **v1.3**  
Canonical URI: [https://anchoring-spec.org/v1.3/](https://anchoring-spec.org/v1.3/)  
Previous versions:  
 **v1.2** (frozen) [/v1.2/](https://anchoring-spec.org/v1.2/)  
 **v1.1** (frozen) [/v1.1/](https://anchoring-spec.org/v1.1/)  
 **v1.0** (frozen) [/v1.0/](https://anchoring-spec.org/v1.0/)

A published version is never edited in place. Where a published version is found to contain an error, the error remains in that version and the correction is issued as a new version, so that the change is itself dateable. Withdrawal of a non-normative artefact published by a version, such as a schema file, is recorded in the section that published it.

## 12 Governance

The Anchoring Specification is a public, versioned technical specification, released into the public domain.

It is currently authored and maintained by the [Umarise](https://umarise.com) team. There is no foundation, consortium, or multi-stakeholder governance body. This is a single-maintainer specification.

The specification is open for implementation by any party without restriction. Any party may fork, mirror, or republish it under any terms permitted by the public domain.

Implementations may reference compliance with a specific specification version.

**No implementation has normative authority over the specification.**

If conflicts arise between an implementation and the specification, the specification prevails.

External co-maintainers and multi-stakeholder governance are welcome and may be adopted if and when adoption warrants it. Until then, governance independence is not claimed. **Technical independence** (the property that proofs verify without the issuer) is established by Section 9 and is independent of governance.

## 13 Threat Model

Anchoring assumes:

- collision resistance of the selected hash function
- integrity of ledger consensus
- availability of public ledger validation data
- continued availability of cryptographic verification algorithms

Anchoring does not protect against:

- pre-image attacks
- collusion attacks against ledger consensus
- artifact substitution prior to anchoring
- social engineering
- legal misinterpretation

## 14 Cryptographic Requirements

Hash functions used in anchoring must:

- be publicly specified
- have no known practical collision attacks
- be widely reviewed by the cryptographic community

If a hash function becomes compromised, future versions of this specification may deprecate its use.

This specification permits future versions to introduce alternative cryptographic primitives. Algorithm agility is supported through versioned specification updates.

## 15 Time Semantics

Time T is defined as the earliest consensus-confirmed inclusion point of the commitment in ledger L.

Anchoring establishes existence-at-or-before-T.

It does not establish exact creation time.

The "inclusion point" referenced above is a structural fact in the ledger's data model. For ledgers structured as a chain of blocks (such as Bitcoin), the inclusion point is the block at which the commitment becomes consensus-fixed; this block is identified by its block height or equivalent canonical position. A wall-clock value associated with that block (such as Bitcoin's `block.time` or `nTime` field) may be exposed as a Reference Wall-Clock Projection for human readability, but the normative inclusion point is the block itself, not the wall-clock value.

Implementations of profile specifications (such as the IETF profile in *draft-fassbender-scitt-time-anchor*) may define their normative temporal binding in terms of the structural inclusion point (e.g., block height) and present any wall-clock projection as informative.

Finalization criteria are ledger-specific and defined by the operational consensus rules of ledger L. This specification does not prescribe a fixed confirmation threshold.

## 16 Non-Retroactivity

Anchoring does not permit retroactive modification of proof validity once ledger inclusion is finalized.

If a ledger undergoes reorganization prior to finalization, verification status may change from *valid* to *unverifiable*.

## 17 Compliance Statement

An implementation claiming compliance must state:

- the specification version
- the hash function used
- the ledger type used
- any deviations from default proof structure

Compliance claims must reference a specific specification version.

An implementation is non-conformant if it:

- extends or alters the outputs defined in Section 5
- omits or weakens the exclusions defined in Section 8
- requires issuer-controlled infrastructure in violation of Section 9

## 18 Archival Considerations

Long-term verifiability depends on:

- preservation of proof bundle integrity
- continued availability of ledger validation data
- continued availability of cryptographic verification algorithms

Anchoring proofs should be stored in durable, non-proprietary formats.

**Conformance note.** An implementation that extends or alters the outputs defined in Section 5, omits or weakens the exclusions defined in Section 8, or requires issuer-controlled infrastructure in violation of Section 9, is non-conformant with this specification.

Canonical publication of the Anchoring Specification v1.3 (Current).  
The specification is normative. Implementations are not.

[Reference verifier](https://verify-anchoring.org) · [One-page reference](https://anchoring-spec.org/one-page/) · [Why Anchoring (non-normative)](https://anchoring-spec.org/appendix/why-anchoring/) · [Source](https://github.com/AnchoringTrust/anchoring-spec)
