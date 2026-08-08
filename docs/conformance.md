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
3. the reported error code matches `expected_error.code`; and
4. the implementation does not create a downstream commitment or successful lifecycle result from the rejected instance.

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

The mandatory v0.3 vector profile covers:

| Area | Required behavior |
| --- | --- |
| Authorization | Decision values, freshness, revocation, revision binding, identity binding, replay isolation, conversion preconditions. |
| Action | revision success and conflict, unchanged and changed-input replay, concurrency, atomic multi-object outcomes, asynchronous completion, Provider authority, audit de-duplication. |
| Extension | valid optional and required manifests, unknown behavior, namespace, version compatibility, digest integrity, duplicate conflicts. |
| Security | Owner isolation, Provider attribution, downgrade rejection, extension integrity, idempotency conflict, safe error and audit output. |
| Governance dependencies | object schemas, lifecycle transitions, Standard Errors, Audit Events, and cross-object references. |

A required behavior MAY be demonstrated by a vector assigned to another area
when the vector description and target identify the shared invariant. Coverage
reports MUST count the vector in every satisfied requirement without executing it
more than once.

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
