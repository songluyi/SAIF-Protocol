# SAIF Operation Context v0.3

## Status

This document defines the binding-neutral context carried or resolved for every
state-changing SAIF Reference Node operation. It refines the common operation
context already required by Action Execution Semantics without defining a
transport or runtime.

The normative schema is
[`schemas/operation-context.schema.json`](../../schemas/operation-context.schema.json).

## Purpose

Operation Context makes one action independently attributable and replay-safe.
It connects the caller, Request, Authorization, Execution, extensions, and
audit evidence without embedding transport credentials in portable data.

## Core Context

Every state-changing operation MUST carry or resolve one schema-valid Operation
Context before Authorization or lifecycle mutation.

The context contains:

- `operation_id`, identifying the logical operation across retries;
- `saif_version` and `profile`, identifying the selected protocol semantics;
- `correlation_id`, connecting related protocol objects and Audit Events;
- `causation_id`, identifying the immediately preceding operation when known;
- `idempotency_key`, scoped according to Action Execution Semantics;
- `actor`, containing the resolved protocol actor;
- `target`, containing the target object and expected revision when applicable;
- `request_context`, containing Owner, Agent, Request, and Authorization
  references when applicable;
- `execution_context`, containing Order, Execution, and Provider references
  when applicable;
- `authorization_decision_id`, when an authorized commitment is created or
  advanced;
- extension references and verified manifest digests; and
- the operation-specific `payload`.

Bindings MAY encode these values separately, but they MUST reconstruct the same
schema-valid context for validation, errors, and audit.

## Request Context

Request Context binds an action to its accountable delegation chain.

When a Request already exists, `request_context` MUST contain its ID and current
revision. When the action depends on delegated authority, it MUST also contain
the Owner ID, Agent ID, Authorization ID, and Authorization Decision ID used by
the action.

The resolved Request, Authorization Decision, and Operation Context MUST agree
on Request ID, Request revision, Owner ID, Agent ID, Authorization ID, and
`correlation_id`. A mismatch fails before creation or mutation of an Order.

`CreateRequest` may use a proposed Request ID at revision `0`. It still
identifies the Owner and Agent when the Agent acts under delegation.

## Execution Context

Execution Context binds Provider activity to the commitment being executed.

An operation that creates or advances an Execution MUST identify:

- the related Order ID and expected Order revision;
- the Execution ID and expected Execution revision when it already exists; and
- the selected Provider ID.

Provider results MUST resolve to the same Order, Execution, Provider,
`operation_id`, and `correlation_id` recorded when execution was created.
Provider input cannot replace Request or Authorization context.

## Correlation Identifiers

A `correlation_id` identifies one portable business interaction. It is opaque,
case-sensitive, and at least one character long.

All objects, decisions, results, errors, and Audit Events produced by one atomic
operation MUST preserve the Operation Context `correlation_id`. A retry MUST
reuse the original `operation_id` and `correlation_id`. A newly caused operation
uses a new `operation_id`, preserves the business `correlation_id`, and sets
`causation_id` to the preceding operation ID when known.

Correlation identifiers MUST NOT contain credentials, private keys, bearer
tokens, or confidential payload data. They are identifiers, not authorization
claims.

## Validation Order

A Reference Node evaluates Operation Context in this order:

1. schema and selected-version validation;
2. actor resolution;
3. Request and Execution reference integrity;
4. extension declaration and digest validation;
5. Authorization Decision freshness and scope;
6. expected revision and idempotency validation; and
7. lifecycle mutation.

A missing required context value fails with `VALIDATION` /
`SAIF-VALIDATION-0001`. A cross-object mismatch fails with `PROTOCOL` /
`SAIF-PROTOCOL-0006`. No failed context validation may create a commitment,
Execution, Settlement, or successful Audit Event.

## Audit Projection

Every accepted state-changing operation MUST project these values into portable
audit evidence directly or through stable references:

- operation ID and correlation ID;
- actor identity;
- affected object IDs and revisions;
- Authorization Decision ID when applicable;
- Provider ID when applicable;
- selected extension IDs, versions, and manifest digests;
- Action Outcome; and
- Standard Error reference when applicable.

The complete trace rules are defined by
[Audit Correlation](audit-correlation.md).

## Conformance

Conformance evidence MUST include:

- one complete schema-valid Operation Context;
- rejection of a missing correlation identifier;
- Request and Authorization reference agreement;
- Execution and Provider reference agreement;
- preserved correlation across a caused operation;
- preserved operation and correlation identifiers on replay; and
- rejection before mutation when context validation fails.

This specification defines portable data and observable semantics only.
