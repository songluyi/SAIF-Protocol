# SAIF Extension Declaration v0.3

## Status

This document defines the machine-readable Extension Manifest used by the optional SAIF Reference Node v0.3 profile.

It implements the governance rules in the [SAIF Extension Proposal Process](../proposal-process.md) without changing v0.2 core object semantics.

## Purpose

An Extension Manifest allows independent implementations to determine:

- which extension is requested or supported;
- who controls its namespace;
- which version is in use;
- whether it is required or optional;
- which SAIF versions and protocol areas it applies to;
- how its schema or validation artifact is identified and verified; and
- what to do when the extension is unknown.

The normative schema is [`schemas/extension-manifest.schema.json`](../../schemas/extension-manifest.schema.json).

## Identifier and Namespace

Extensions MUST use the lowercase reverse-domain convention defined by the
[Extension Proposal Process](../proposal-process.md).

Example:

```text
org.example.document-priority
```

The manifest separates:

- `namespace`: the authority, such as `org.example`; and
- `extension_id`: the complete extension identifier.

Both values use lowercase
[RFC 1035](https://www.rfc-editor.org/rfc/rfc1035.html)-compatible labels. Each label is 1 to 63
ASCII characters, starts with a letter, ends with a letter or digit, and uses
only lowercase letters, digits, or interior hyphens. Each value is at most 253
characters. `extension_id` MUST begin with the declared namespace followed by
`.` and an extension-local name. Private or experimental extensions MUST NOT use
the reserved `org.saif` namespace or allocate core `SAIF-` error codes.
Hyphen-only experimental identifiers are not valid v0.3 Extension IDs.

## Manifest Fields

| Field | Meaning |
| --- | --- |
| `id` | Unique manifest object identifier. |
| `type` | Always `extension_manifest`. |
| `created_at` | Manifest creation time. |
| `metadata` | Non-normative extension data. |
| `saif_version` | SAIF version used to interpret the manifest. |
| `extension_id` | Collision-resistant extension identifier. |
| `namespace` | Namespace authority. |
| `version` | Extension semantic version. |
| `required` | Whether lack of support must fail the operation. |
| `applies_to` | Core objects, operations, capabilities, or profiles affected. |
| `compatibility` | Inclusive minimum and exclusive maximum supported SAIF version. |
| `schema` | URI and integrity digest of the extension schema or validation artifact. |
| `proposal` | SEP identifier and status, or `null` for private extensions. |
| `unknown_behavior` | `REJECT` for required extensions or `IGNORE` for optional extensions. |

## Compatibility Range

`compatibility.saif_min` is inclusive. `compatibility.saif_max_exclusive` is exclusive.

An extension is compatible only when the selected SAIF version is within that range. Implementations MUST compare numeric semantic-version components rather than lexical strings.

The extension version is negotiated separately from the SAIF version. A Node MUST NOT select an extension version outside its declared SAIF compatibility range.

## Required and Optional Behavior

### Required Extension

When `required` is `true`:

- `unknown_behavior` MUST be `REJECT`;
- the receiving Node MUST support the exact or compatible extension version;
- the schema integrity check MUST succeed; and
- an unsupported, conflicting, or invalid extension MUST fail before Authorization or lifecycle mutation.

### Optional Extension

When `required` is `false`:

- `unknown_behavior` MUST be `IGNORE`;
- an unsupported extension may be ignored only if the core object and operation remain valid and semantically complete without it; and
- ignoring it MUST NOT change Authorization, lifecycle, Execution, or Settlement meaning.

If an allegedly optional extension affects a required decision, it is semantically required and MUST be declared `required: true`.

## Discovery and Operation Placement

Capability discovery returns supported Extension Manifests or stable references to them.

A state-changing operation carries an extension reference list containing:

- `extension_id`;
- requested extension version;
- required status; and
- manifest digest.

The full manifest need not be repeated when both parties resolve the same verified manifest.

A transport binding may choose field placement, but it MUST preserve these values and MUST NOT hide required extension declarations in opaque transport metadata.

## Registry and Trust

An Extension Registry is a logical set of verified manifests. It may be local configuration, persisted data, or capability discovery output.

A conforming Node:

1. MUST verify namespace and identifier consistency;
2. MUST verify the schema artifact against its declared digest before use;
3. MUST reject two manifests with the same extension ID and version but different digests;
4. MUST reject incompatible required versions;
5. MUST record the selected manifest and digest in operation audit context; and
6. MUST NOT execute code merely because a manifest contains a schema URI.

Namespace-prefix consistency and numeric compatibility-range ordering are
semantic validations because JSON Schema cannot compare two independent string
values. `compatibility.saif_min` MUST be numerically lower than
`compatibility.saif_max_exclusive`.

Remote schema retrieval is not required. A Node may rely on an approved local artifact cache. Retrieval policy remains an implementation and security-profile choice.

## Schema Integrity

The v0.3 profile uses a digest descriptor containing:

- `algorithm`: `SHA-256`; and
- `value`: a lowercase 64-character hexadecimal digest.

This digest verifies artifact identity. It does not define canonical serialization for SAIF business objects and does not require a blockchain or signing platform.

## Proposal Status

For a standardized extension, `proposal` identifies the SEP and its status.

Only an `ACCEPTED` proposal may claim standard extension status. Draft, review, rejected, withdrawn, or superseded proposals remain experimental or historical according to the proposal process.

A private extension may set `proposal` to `null` but must retain its private namespace.

## Conflict Resolution

When multiple manifests are presented:

1. group by `extension_id`;
2. reject duplicate ID/version entries with different digests;
3. select a version compatible with SAIF and all required operation constraints;
4. reject when no compatible version satisfies a required extension;
5. ignore unsupported optional versions only under optional-extension rules; and
6. expose the selected versions through the operation result and audit context.

The protocol does not select versions by vendor preference.

## Standard Errors

| Condition | Category | Code |
| --- | --- | --- |
| Required extension unsupported | `EXTENSION` | `SAIF-EXTENSION-0001` |
| Extension version incompatible | `EXTENSION` | `SAIF-EXTENSION-0002` |
| Namespace and extension ID mismatch | `EXTENSION` | `SAIF-EXTENSION-0003` |
| Manifest digest conflict or mismatch | `EXTENSION` | `SAIF-EXTENSION-0004` |
| Invalid required/unknown behavior combination | `EXTENSION` | `SAIF-EXTENSION-0005` |
| Attempted core authorization or security bypass | `EXTENSION` | `SAIF-EXTENSION-0006` |

Errors MUST follow the [SAIF Standard Error Model](../error-model.md).

## Core Security Precedence

Extension semantics are additional constraints over the portable core. They do
not create authority.

Core schema, semantic, Authorization, lifecycle, error, audit, and Security
Profile checks MUST remain effective when an extension is present. An extension
MAY narrow an allowed operation, but it MUST NOT convert a core denial or
invalid transition into an allowed mutation.

When core and extension decisions differ, the effective decision is their
intersection. A result that is less restrictive than the core decision fails
with `EXTENSION` / `SAIF-EXTENSION-0006` before mutation.

A schema URI is data identifying an artifact. Resolution MUST use an
implementation-approved trust policy. The URI MUST NOT authorize code
execution, ambient credential use, or network access outside the policy. A
digest-verified local artifact is sufficient; remote retrieval is not required.

The selected manifest ID, version, and verified digest MUST be preserved in the
Operation Context result and the correlated audit evidence.

## Security Requirements

- Manifests are untrusted input until schema, namespace, compatibility, and digest checks pass.
- A schema URI MUST NOT trigger automatic code execution or unrestricted network access.
- Extension data MUST follow the same Owner isolation and data-minimization rules as core data.
- An extension MUST NOT weaken core validation, Authorization, state, error, or audit requirements.
- Required extension negotiation MUST complete before any state mutation.

## Conformance

Conformance vectors MUST cover:

- a valid optional manifest;
- a valid required manifest;
- optional unknown extension ignore;
- required unknown extension rejection;
- namespace mismatch;
- incompatible SAIF range and a numeric-versus-lexical comparison boundary;
- digest mismatch;
- duplicate manifest conflict;
- invalid `required` and `unknown_behavior` combinations;
- selected manifest and digest audit binding;
- schema resolution with no code execution or unrestricted network access; and
- rejection of an attempted core Authorization or security bypass before
  mutation.

This specification defines data and observable decisions only. It does not define an extension runtime, plugin loader, marketplace, or commercial registry.
