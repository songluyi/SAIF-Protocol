# SAIF Protocol Conformance Test Vector Specification v0.3

## Purpose

SAIF conformance test vectors provide shared, implementation-independent evidence that separate systems interpret the same protocol objects, states, errors, and audit events consistently.

The specification defines test data and expected outcomes. It does not prescribe a programming language, test framework, runtime, API, transport, or deployment model.

## Conformance Principles

- **Vendor neutral:** vectors must not require a particular commercial product or platform.
- **AI model neutral:** vectors must not depend on a specific model or model provider.
- **Provider neutral:** vectors validate observable SAIF semantics rather than Provider internals.
- **Implementation independent:** any implementation may consume the JSON vectors.
- **Deterministic:** the same vector and protocol version must produce the same conformance result.
- **Public:** normative vectors are stored in this repository and reviewed with the specification.

## Directory Structure

```text
conformance/
├── valid/
│   ├── request-submitted.json
│   └── order-fulfilled.json
└── invalid/
    ├── request-missing-agent-id.json
    └── order-invalid-state.json
```

Valid vectors contain instances that must be accepted. Invalid vectors contain instances that must be rejected with the expected standard error category and code.

## Test Vector Envelope

Every vector contains:

| Field | Type | Description |
| --- | --- | --- |
| `vector_id` | string | Stable identifier for the test vector. |
| `saif_version` | string | SAIF version against which the vector is evaluated. |
| `profile` | string | Conformance profile; v0.3 uses `saif-reference-node/0.3`. |
| `area` | string | Normative test area. |
| `target` | string | Schema or normative model under test. |
| `expected` | string | `VALID` or `INVALID`. |
| `description` | string | Human-readable purpose of the vector. |
| `instance` | JSON value | Protocol data or semantic scenario to validate. |
| `expected_error` | object | Required only when `expected` is `INVALID`. |

Example envelope:

```json
{
  "vector_id": "request-valid-001",
  "saif_version": "0.3.0",
  "profile": "saif-reference-node/0.3",
  "area": "OBJECT",
  "target": "schemas/request.schema.json",
  "expected": "VALID",
  "description": "A submitted document request with all required fields.",
  "instance": {}
}
```

The normative envelope schema is
[`schemas/conformance-vector.schema.json`](../schemas/conformance-vector.schema.json).

## Vector Taxonomy

The v0.3 areas are:

- `OBJECT`
- `AUTHORIZATION`
- `LIFECYCLE`
- `ACTION`
- `EXTENSION`
- `ERROR`
- `AUDIT`
- `SECURITY`

Vector IDs use lowercase kebab-case followed by a three-digit sequence. The
directory communicates expected validity; the `expected` field MUST agree with
that directory.

Every invalid vector MUST identify one primary failure. Supporting context MUST
otherwise be conforming. When more than one rule could reject the instance, the
applicable specification MUST define error precedence before the vector can be
normative.

## Standard Error Taxonomy

`expected_error.category` MUST use exactly one Standard Error Model category:

- `VALIDATION`
- `VERSION`
- `AUTHORIZATION`
- `STATE_TRANSITION`
- `PROVIDER`
- `SETTLEMENT`
- `EXTENSION`
- `PROTOCOL`

The shortened token `STATE` may appear inside a code such as
`SAIF-STATE-0002`; it is not a valid category value.

## Canonical Error Path

`expected_error.path` is an
[RFC 6901](https://www.rfc-editor.org/rfc/rfc6901.html) JSON Pointer relative
to the vector's
complete `instance`. The empty string is the canonical pointer for a root
instance error. `/` identifies a member whose name is the empty string.

A missing-member error uses the pointer at which the member would have appeared.
A semantic vector points to the smallest input value responsible for the primary
failure. Runners MUST compare the reported pointer exactly after JSON Pointer
escaping; they MUST NOT infer an implementation-specific object path.

## Valid Vector Rules

A valid vector passes when:

1. its envelope contains all required vector fields;
2. its `saif_version` is supported by the implementation;
3. its `target` resolves to the identified normative SAIF artifact;
4. its `instance` conforms to the target schema or model; and
5. no additional semantic rule makes the instance invalid.

An implementation must report a failure if it rejects a normative valid vector.

## Invalid Vector Rules

An invalid vector passes when:

1. the implementation rejects the instance;
2. the reported error category matches `expected_error.category`;
3. the reported error code matches `expected_error.code`;
4. the reported RFC 6901 pointer exactly matches `expected_error.path`; and
5. the implementation does not create a downstream commitment or successful
   lifecycle result from the rejected instance.

An implementation must report a failure if it accepts a normative invalid vector.

## Conformance Areas

### Object Conformance

Validates required fields, types, enumerations, and additional-property rules defined by SAIF JSON Schemas.

### Lifecycle Conformance

Validates allowed and forbidden state transitions defined in the [Protocol State Model](state-machine.md).

### Error Conformance

Validates error categories, codes, required fields, and retry semantics defined in the [Standard Error Model](error-model.md).

### Audit Conformance

Validates the minimum attribution and correlation fields defined in the [Audit Event Model](audit-event-model.md).

### Extension Conformance

Validates that extensions use an approved namespace, declare compatibility, and do not redefine core SAIF semantics.

## v0.3 Reference Node Coverage

The mandatory v0.3 profile uses the following requirement-to-vector matrix.
Vector IDs are normative identifiers; one vector may satisfy multiple rows when
its target and description identify every shared invariant.

| Requirement | Required evidence | Vector IDs |
| --- | --- | --- |
| `STATUS-01` | Protocol `SUCCESS` with every Action Outcome | `status-success-outcomes-010`, `status-layer-separation-001` |
| `STATUS-02` | Protocol `FAILURE` with error and no outcome | `status-protocol-failure-002` |
| `STATUS-03` | Three distinct `PENDING` layers | `status-pending-layers-011` |
| `STATUS-04` | Complete Audit projection | `status-audit-projection-012` |
| `STATUS-05` | Reject cross-layer enum placement | `status-layer-confusion-001`, `status-layer-confusion-002` |
| `API-01` | All required state-changing operations | `api-operation-matrix-010` |
| `API-02` | Allowed actor matrix | `api-actor-allowed-011` |
| `API-03` | Denied actor behavior | `api-actor-denied-010` |
| `API-04` | Query authorization and ordering | `api-query-behavior-012`, `api-query-owner-isolation-011` |
| `API-05` | Pagination and invalid cursor handling | `api-query-behavior-012`, `api-query-invalid-cursor-012` |
| `API-06` | Discovery surface and neutrality | `api-discovery-behavior-013` |
| `API-07` | Required discovery response fields | `api-discovery-incomplete-013` |
| `AUTH-01` | Decision values and required fields | `authorization-decision-valid-001`, `authorization-decision-deny-002`, `authorization-requires-action-003`, `authorization-decision-invalid-001` |
| `AUTH-02` | Freshness, revision, revocation, replay, conversion | `authorization-decision-expired-002`, `authorization-revision-mismatch-003`, `authorization-revoked-after-evaluation-004`, `authorization-cross-request-replay-005`, `authorization-non-allow-conversion-006` |
| `ACTION-01` | Replay, revision, and concurrency | `action-semantics-valid-001`, `action-idempotency-conflict-003`, `action-semantics-invalid-001`, `action-concurrent-terminal-conflict-004` |
| `ACTION-02` | Atomic and asynchronous outcomes | `action-semantics-valid-002`, `action-execution-completion-003`, `action-asynchronous-completion-004`, `action-incomplete-atomic-outcome-005` |
| `EXT-01` | Manifest, unknown, and compatibility behavior | `extension-manifest-valid-001`, `extension-manifest-required-002`, `extension-manifest-invalid-001`, `extension-optional-unknown-ignore-003`, `extension-semantics-invalid-002`, `extension-incompatible-version-004` |
| `EXT-02` | RFC 1035 namespace grammar and authority | `extension-namespace-rfc1035-010`, `extension-namespace-trailing-hyphen-010`, `extension-namespace-leading-hyphen-011`, `extension-namespace-empty-label-012`, `extension-namespace-leading-digit-013`, `extension-namespace-reserved-014`, `extension-namespace-mismatch-003` |
| `EXT-03` | Digest and duplicate conflict | `extension-digest-mismatch-005`, `extension-duplicate-conflict-006` |
| `SEC-01` | Owner, Provider, negotiation, downgrade, redaction | `security-owner-isolation-001`, `security-owner-scoped-access-001`, `action-semantics-invalid-002`, `security-version-negotiation-002`, `security-version-downgrade-002`, `security-error-secret-disclosure-003`, `security-audit-secret-disclosure-004` |
| `GOV-01` | Objects, lifecycle, errors, audit, references | `request-valid-001`, `request-invalid-001`, `order-valid-001`, `order-invalid-001`, `lifecycle-request-submit-001`, `lifecycle-terminal-order-reopen-001`, `error-standard-shape-001`, `error-nonstandard-category-001`, `audit-owner-attribution-001`, `audit-missing-authorization-001`, `cross-object-reference-chain-001`, `cross-object-reference-mismatch-001` |

A coverage report MUST list each matrix row, the vectors executed for that row,
and its pass, fail, or skipped result. A required behavior MAY be demonstrated by
a vector assigned to another area only when the vector description and target
identify the shared invariant. The report counts that vector in every satisfied
row without executing it more than once.

## Conformance Claim

An implementation claiming SAIF v0.3 Reference Node conformance should publish a report containing:

- implementation name and version;
- supported SAIF version;
- supported profile;
- date of evaluation;
- vector commit or release identifier;
- number of passed, failed, and skipped vectors;
- reason for every skipped vector; and
- supported optional extensions.

A claim must not state full conformance when any required vector fails. Partial profiles may be declared only when the profile and omitted areas are explicit. Legacy v0.2 claims remain governed by the v0.2 release artifact and vectors.

## Test Vector Governance

Normative vectors follow the same review and versioning rules as schemas and lifecycle documents.

- A vector correction that does not change expected semantics may be a Patch change.
- A new backward-compatible vector may be a Minor change.
- Changing a previously valid instance to invalid, or the reverse, is a breaking semantic change and requires a Major version unless correcting an acknowledged specification defect before a stable release.

Every accepted protocol change should add or update vectors that demonstrate the intended interoperable behavior.

## Security and Privacy

Conformance vectors must use synthetic identities and data. They must not contain credentials, personal information, private keys, real payment information, or confidential Provider details.
