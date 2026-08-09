# SAIF v0.3 Normative Requirement Matrix

## Purpose

This document explains the machine-readable mapping from each BCP 14 mandatory
requirement to its specification location, schema or semantic rule, and
conformance-vector evidence.

The canonical matrix is
[`conformance/normative-requirements-v0.3.json`](../conformance/normative-requirements-v0.3.json).
Its schema is
[`schemas/normative-requirement-matrix.schema.json`](../schemas/normative-requirement-matrix.schema.json).

## Evidence Record

Each matrix record contains:

| Field | Meaning |
| --- | --- |
| `requirement_id` | Stable identifier for the evidence row. |
| `keywords` | BCP 14 mandatory keyword or keywords present in the source clause. |
| `specification` | Repository-relative normative artifact path. |
| `section` | Heading containing the requirement. |
| `source_line` | Line containing the mandatory keyword in the frozen candidate. |
| `source_digest` | SHA-256 digest of the normalized source line. |
| `statement` | Exact normalized source line. |
| `schema_rules` | JSON Schema pointers or named semantic/artifact rules. |
| `vector_ids` | Deterministic vectors providing evidence. |
| `evidence_profile` | Governance, Core, SP0, SP1, or SP2 evidence grouping. |

A `semantic:` rule identifies an observable cross-object rule that JSON Schema
cannot express. An `artifact:` rule identifies repository or release evidence.
Neither notation introduces implementation code.

## Completeness Model

Coverage validation performs four checks:

1. scan every artifact in the normative scope for capitalized BCP 14 mandatory
   keywords;
2. match each source occurrence to exactly one matrix record by path, line,
   normalized text digest, and keyword set;
3. resolve every schema rule to a published schema pointer or declared semantic
   or artifact rule; and
4. resolve every vector ID to one valid or invalid vector.

The source digest makes stale evidence visible when normative prose changes.
The matrix is regenerated and reviewed whenever a mandatory clause moves or
changes.

## Evidence Profiles

| Profile | Evidence scope |
| --- | --- |
| `GOVERNANCE` | Versioning, vector format, proposal, release, and artifact rules. |
| `CORE` | Status, object, operation, lifecycle, and error semantics. |
| `SP0` | Core validation, extension fail-closed behavior, and safe portable data. |
| `SP1` | Authorized actor, replay, Provider, and audit-correlation behavior. |
| `SP2` | Binding, resource-control, trust-policy, retention, and evidence-integrity declarations. |

SP0, SP1, and SP2 are cumulative conformance-evidence groupings. They are not
commercial assurance levels and do not select an authentication, storage, or
cryptographic product.

## Freeze Gate

A freeze candidate is evidence-complete only when:

- every mandatory source occurrence is represented;
- every referenced schema rule resolves;
- every referenced vector exists and passes;
- no vector is omitted from the coverage report; and
- the release commit and intended immutable tag are recorded.

The matrix adds no runtime, transport binding, commerce behavior, or payment
logic.
