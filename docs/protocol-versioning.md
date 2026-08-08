# SAIF Protocol Versioning Strategy

## Purpose

SAIF Protocol uses explicit versions so Agents, connectors, providers, and validation tools can determine whether they share compatible business object and lifecycle semantics.

## Version Format

SAIF versions use:

```text
MAJOR.MINOR.PATCH
```

Example:

```text
v0.1.0
```

The optional `v` prefix is used in documentation and release tags. Protocol data carries the numeric value without the prefix.

## Major Version

A Major version contains breaking changes.

Examples:

- removing a required or optional field;
- changing an existing field type;
- changing the meaning of an object or field;
- changing lifecycle states or transition semantics; and
- making an optional behavior mandatory.

An implementation must not assume compatibility across different Major versions.

## Minor Version

A Minor version introduces backward-compatible features.

Examples:

- adding an optional field;
- adding an object type;
- adding a Provider capability;
- adding a non-breaking lifecycle extension; and
- publishing a new optional binding or profile.

An implementation that does not understand an optional Minor-version feature may ignore it only when the specification explicitly permits that behavior.

## Patch Version

A Patch version contains fixes that do not change protocol semantics.

Examples:

- correcting errors in examples;
- clarifying documentation;
- fixing non-semantic schema defects; and
- improving explanatory text.

Patch releases must not introduce new required behavior.

## Version Declaration

Protocol implementations should declare supported SAIF versions.

Example:

```json
{
  "saif_version": "0.1.0"
}
```

An implementation may declare more than one supported version through implementation-specific configuration or capability discovery. The transport used to exchange that declaration is outside the core SAIF Protocol.

## Immutable Schema Identity

Every normative schema release MUST declare an immutable, versioned `$id` using:

```text
https://raw.githubusercontent.com/songluyi/SAIF-Protocol/v{MAJOR}.{MINOR}.{PATCH}/schemas/{schema-file}
```

For v0.3.0:

```text
https://raw.githubusercontent.com/songluyi/SAIF-Protocol/v0.3.0/schemas/request.schema.json
```

The release tag referenced by a normative schema `$id` MUST NOT be moved or
reused. Corrected schemas receive a new protocol Patch version and a new `$id`.
Default-branch and unversioned repository URLs MUST NOT be used as normative
schema identities.

Every schema also declares `x-saif-version` with the exact release version. This
annotation identifies the schema artifact; it does not add a field to business
objects. Where an object includes `saif_version`, that value MUST match the
selected schema release. For objects without that field, the operation or binding
envelope selects the schema release.

Pre-release schema files may declare the intended release-tag URL before the tag
exists. The URL becomes resolvable and immutable when the reviewed release tag is
created.

## Compatibility Principles

1. Producers must emit objects that conform to the declared SAIF version.
2. Consumers must validate the declared version before relying on object semantics.
3. Unknown required fields, object types, or lifecycle semantics must not be silently treated as understood.
4. Optional metadata must not redefine normative fields.
5. Provider-specific extensions must remain isolated from the portable core model.
6. Transport bindings must state which SAIF versions they carry.
7. Validators must resolve schemas by immutable `$id` or an integrity-checked
   local copy associated with that exact `$id`.

## Governance

Changes to normative objects, schemas, states, and lifecycle rules should document:

- the affected protocol version;
- whether the change is breaking;
- migration impact;
- compatibility expectations; and
- updated conformance examples.

The repository release tag, documentation, schemas, and examples should identify the same protocol version before a public release is marked stable.
