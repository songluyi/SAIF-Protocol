# SAIF Reference Node Security Profile v0.3

## Status and Scope

This document defines the minimum security properties for the optional SAIF v0.3 Reference Node profile. It is transport, vendor, AI model, Provider, credential, and cryptographic-algorithm neutral.

Conformance to this profile concerns observable protocol behavior. It does not require a particular authentication service, network stack, key manager, database, or deployment topology.

The profile identifier is `saif-reference-node-security/0.3`.

## Trust Zones

1. **Client:** AI clients and business systems submit untrusted protocol input.
2. **Transport:** a binding authenticates peers and protects messages but does not grant business authority.
3. **Reference Node:** validation, authorization, lifecycle, error, and audit rules are enforced.
4. **Provider:** a Provider is trusted only for declared capabilities and attributable results.
5. **Storage and Audit:** stored protocol evidence is access-controlled and integrity-protected.

## Required Security Properties

A conforming Reference Node profile implementation MUST:

1. validate schema, semantic, version, and required-extension constraints before applying state changes;
2. map each authenticated transport peer to an explicit protocol actor before accepting a state-changing operation;
3. keep peer authentication distinct from Owner-to-Agent Authorization;
4. verify Authorization ownership, Agent binding, scope, freshness, revocation status, and any decision expiry before conversion or dispatch;
5. require an expected object revision and an idempotency key as defined by [Action Execution Semantics](action-execution-semantics.md);
6. reject stale revisions and return the prior result for a valid idempotent replay;
7. bind Provider results to the selected Provider, Order, Execution, correlation ID, and expected revision;
8. prevent one Owner or tenant from reading or mutating another Owner's objects without explicit authorization;
9. verify required Extension Declarations, including namespace, compatibility, and declared schema digest, before processing extension-owned data;
10. fail closed for unknown required extensions and resist silent protocol or security-profile downgrade;
11. authorize access to Audit Events and Standard Errors, minimize returned data, and preserve attribution and integrity;
12. exclude credentials, private keys, bearer tokens, private prompts, and unnecessary personal data from portable SAIF objects, errors, and audit records; and
13. apply binding-level limits for message size, request rate, execution time, and resource consumption, returning a safe Standard Error when a limit is reached.

## Binding Requirements

A non-local transport binding MUST provide:

- peer authentication;
- message confidentiality and integrity;
- replay resistance;
- credential rotation and revocation behavior;
- an unambiguous mapping from the authenticated peer to the operation `actor`; and
- explicit negotiation of the SAIF version and security profile.

The binding MAY use mutual transport credentials, signatures, delegated tokens, hardware-backed credentials, or another mechanism. The mechanism is outside the core protocol.

Local bindings MUST document an equivalent trust assumption and actor-mapping rule.

## Authorization Freshness

An `ALLOW` Authorization Decision is usable only when:

- its Request and Authorization revisions match the current stored revisions;
- its `expires_at` value has not passed;
- the referenced Authorization remains active and has not been revoked; and
- the requested operation remains within `evaluated_scope`.

A failed freshness check MUST prevent the action. The node MUST emit a denied or conflict outcome and an attributable Audit Event. It MUST NOT silently re-use a stale decision.

## Provider Result Integrity

A Provider result MUST be accepted only through the Provider Boundary. The node MUST verify that the submitting peer is permitted to act for the referenced Provider and that the result matches the expected Order, Execution, correlation ID, and revision.

Provider evidence MAY be stored by reference or digest. Provider evidence does not authorize a lifecycle transition by itself; the Core Module Boundary remains responsible for transition validation.

## Extension Supply-Chain Controls

Before enabling a required extension, a node MUST:

- validate its manifest against `schemas/extension-manifest.schema.json`;
- confirm namespace and version compatibility;
- resolve the declared schema through an implementation-approved trust policy;
- verify the declared SHA-256 digest over the exact retrieved schema bytes; and
- record the enabled manifest and digest in audit evidence.

The protocol does not mandate a central registry or certificate authority. Implementations MUST document how extension sources become trusted.

## Audit, Error, and Privacy Controls

- Audit records MUST be append-oriented or protected by an equivalent tamper-evident integrity control.
- Audit queries MUST enforce caller authorization and bounded pagination.
- Portable errors MUST expose stable protocol information without stack traces, secrets, credentials, or internal network details.
- Correlation identifiers MUST not embed confidential data.
- Retention and deletion policies MUST preserve legally required minimization while retaining sufficient non-sensitive evidence for protocol accountability.

## Cryptographic Agility

This profile requires protected and attributable exchanges but does not select a cryptographic suite. A binding profile MUST identify its algorithms and downgrade behavior. Algorithms MUST be replaceable without changing SAIF business-object semantics.

## Conformance Requirements

Conformance evidence for this profile MUST include valid and invalid vectors for:

- Authorization Decision freshness and revision binding;
- unauthorized actor and Owner-boundary access;
- duplicate replay and idempotency-key conflict;
- Provider identity or correlation mismatch;
- unknown required extension rejection;
- extension schema-digest mismatch;
- version or profile downgrade rejection; and
- redaction of sensitive error and audit data.

Conformance vectors demonstrate protocol outcomes; they do not require a public-network deployment or a reference runtime.

## Explicit Non-Goals

This profile does not define an authentication provider, identity proofing method, payment-security standard, wallet, commerce system, regulatory regime, cryptographic product, or runtime implementation.
