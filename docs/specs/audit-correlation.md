# SAIF Audit Correlation v0.3

## Status

This document defines the traceability contract connecting an Action,
Authorization, Execution, and Audit Event in the SAIF Reference Node profile. It
does not prescribe a database, tracing product, ledger, transport, or runtime.

The normative evidence schema is
[`schemas/audit-correlation.schema.json`](../../schemas/audit-correlation.schema.json).

## Correlation Chain

A complete authorized execution trace has this portable chain:

```text
Operation Context
  -> Request
  -> Authorization
  -> Authorization Decision
  -> Order
  -> Execution
  -> Audit Events
```

The chain is identified by one `correlation_id` and one originating
`operation_id`. Caused operations preserve the correlation identifier and
record their immediate cause.

## Required References

For an action that creates or advances an authorized Execution, audit evidence
MUST preserve:

- operation ID and correlation ID;
- resolved actor identity;
- Request ID and revision;
- Authorization ID and revision;
- Authorization Decision ID;
- Order ID and revision;
- Execution ID, revision, and Provider ID;
- previous and new lifecycle states;
- Action Outcome;
- Standard Error reference for a failed outcome; and
- every portable Audit Event ID emitted by the operation.

The Authorization Decision, Operation Context, business objects, and Audit
Events MUST use the same `correlation_id`. Their IDs and revisions MUST resolve
to one consistent reference chain.

## Action and Authorization Trace

An action dependent on Owner delegation MUST identify the Authorization and
Authorization Decision that permitted it. Authentication, an API role, or a
Provider account MUST NOT replace those references.

An Audit Event for authorization evaluation records
`AUTHORIZATION_EVALUATED`. An Audit Event for conversion or execution records
the applicable Authorization Decision ID directly or through a stable protected
reference.

A denied, expired, revoked, or mismatched decision MUST remain traceable to the
rejected operation and its Standard Error, while producing no successful
commitment.

## Execution Trace

Execution creation and Provider results MUST preserve the selected Provider,
Order, Execution, operation, and correlation references. A Provider MUST NOT
substitute another Request, Authorization, Order, or Execution into an existing
trace.

Atomic multi-object transitions share one correlation identifier and expose all
required Audit Event references before an `APPLIED` result is returned.

## Replay Trace

An unchanged idempotent replay MUST return the original operation and audit
references. It MUST NOT create a second business Audit Event sequence.

A changed-input reuse of the same scoped idempotency key creates a Standard
Error correlated to the attempted operation and MUST NOT alter the original
trace.

## Failure and Error Trace

When an Action Outcome is `FAILED`, `REJECTED`, or `CONFLICT`, the trace MUST
identify the Standard Error. Error and Audit Event correlation identifiers MUST
match the rejected operation.

A missing or contradictory mandatory reference fails with `PROTOCOL` /
`SAIF-PROTOCOL-0006` before a successful outcome is observable.

## Privacy

Portable correlation evidence MUST NOT include raw idempotency keys,
credentials, private prompts, private keys, bearer tokens, or unnecessary
personal data. An implementation may store a protected digest of an idempotency
key.

## Conformance

Conformance evidence MUST include:

- a complete Action-Authorization-Execution-Audit chain;
- matching correlation identifiers across every artifact;
- rejection of a mismatched or missing Authorization Decision reference;
- Provider and Execution reference integrity;
- one audit sequence for an unchanged replay; and
- an error-linked trace for a rejected action.

This specification defines trace data and observable outcomes only.
