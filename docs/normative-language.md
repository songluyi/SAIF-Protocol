# SAIF Normative Language

## Purpose

This document defines how requirement key words are interpreted across the
normative SAIF Protocol v0.3 specification.

## BCP 14 Declaration

Within the normative scope below, the capitalized key words **MUST**,
**MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**,
**SHOULD NOT**, **RECOMMENDED**, **NOT RECOMMENDED**, **MAY**, and
**OPTIONAL** carry the meanings defined by BCP 14, comprising
[RFC 2119](https://www.rfc-editor.org/rfc/rfc2119.html) and
[RFC 8174](https://www.rfc-editor.org/rfc/rfc8174.html).

These words have BCP 14 meaning only when written in all capitals. Their
lowercase forms are ordinary explanatory language unless a specification
explicitly states otherwise.

## Normative Scope

This declaration applies to:

- `docs/business-object-model.md`;
- `docs/request-lifecycle.md`;
- `docs/provider-interface.md`;
- `docs/protocol-versioning.md`;
- `docs/state-machine.md`;
- `docs/saif-mcp-boundary.md`;
- `docs/conformance.md`;
- `docs/proposal-process.md`;
- `docs/error-model.md`;
- `docs/audit-event-model.md`;
- `docs/reference-architecture.md`;
- `docs/v0.3-roadmap.md`;
- `docs/normative-requirement-matrix.md`;
- contract specifications under `docs/specs/`;
- JSON Schemas under `schemas/`;
- conformance vectors under `conformance/`; and
- normative examples explicitly referenced by those artifacts.

Each artifact in this scope incorporates this declaration by reference. A file
that explicitly identifies itself as non-normative remains non-normative.

The README, files under `docs/review/`, whitepapers, demo descriptions,
implementation notes, and commercial examples are informative unless an
accepted specification explicitly promotes a defined part into the normative
scope.

## Requirement Use

A normative key word is used only for an interoperability, safety, compatibility,
or conformance requirement. Explanatory statements avoid capitalized BCP 14
terms.

A **MUST** or **MUST NOT** requirement requires deterministic conformance
evidence unless the applicable profile states that it cannot be tested without a
future binding. A **SHOULD** or **SHOULD NOT** requirement documents permitted
exceptions and their interoperability consequences. A **MAY** requirement
identifies truly optional behavior and cannot weaken a Core requirement.
