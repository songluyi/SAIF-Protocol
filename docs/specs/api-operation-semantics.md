# SAIF API Operation Semantics v0.3

## Status

This document defines binding-neutral operations for the optional SAIF Reference Node v0.3 profile.

“API” means a logical protocol surface. It does not require HTTP, REST, MCP, RPC, messaging, a server process, or an SDK.

## Normative Dependencies

Operations use:

- the [Business Object Model](../business-object-model.md);
- the [State Model](../state-machine.md);
- the [Authorization Decision Contract](authorization-decision-contract.md);
- [Action Execution Semantics](action-execution-semantics.md);
- the [Extension Declaration](extension-declaration.md);
- the [Standard Error Model](../error-model.md); and
- the [Audit Event Model](../audit-event-model.md).

## Common Request Envelope

Every state-changing operation has a binding-neutral envelope containing:

- operation name and operation ID;
- selected SAIF version and Reference Node profile;
- correlation ID;
- idempotency key;
- actor reference;
- target reference and expected revision when applicable;
- Authorization Decision ID when applicable;
- required and optional extension references; and
- operation-specific payload.

A binding MAY encode these values in a message body, headers, attributes, or another structure. It MUST preserve their meaning and make them available to protocol validation, audit, and error handling.

## Common Result Envelope

Every state-changing operation returns:

- operation ID;
- outcome status from Action Execution Semantics;
- selected SAIF version and profile;
- correlation ID;
- affected object IDs and resulting revisions;
- created object IDs;
- Authorization Decision ID when applicable;
- Audit Event IDs;
- selected extension versions; and
- Standard Error reference when not applied.

## Actor Roles

| Role | Meaning |
| --- | --- |
| `OWNER` | Accountable Party or an explicitly delegated representative. |
| `AGENT` | AI or robotics Agent acting under Owner Authorization. |
| `AUTHORIZATION_EVALUATOR` | Trusted system role permitted to record a policy evaluation. |
| `PROVIDER` | Provider identity permitted to act on assigned Orders or Executions. |
| `REFERENCE_NODE` | Internal system role performing protocol-controlled atomic transitions. |
| `AUDITOR` | Authorized reader of Audit Events and errors. |

Role assignment is resolved through the Security Profile and binding. Role names do not define an authentication provider.

## Required State-Changing Operations

### `CreateRequest`

| Contract item | Requirement |
| --- | --- |
| Actor | `OWNER` or `AGENT` linked to the Owner. |
| Pre-state | No existing Request with the proposed ID. |
| Input | New schema-valid Request payload with requested status `DRAFT`. |
| Success | Request created in `DRAFT`, revision `0`. |
| Idempotency | Required. Replay returns the same Request and revision. |
| Audit | `OBJECT_CREATED`. |
| Errors | Validation, version, extension, duplicate ID, actor mismatch. |

`CreateRequest` MUST NOT implicitly submit the Request.

### `SubmitRequest`

| Contract item | Requirement |
| --- | --- |
| Actor | `OWNER` or the Request Agent acting within Owner delegation. |
| Pre-state | Request `DRAFT` at `expected_revision`. |
| Success | Request `SUBMITTED`, revision incremented by `1`. |
| Idempotency | Required. |
| Audit | `STATE_TRANSITION`. |
| Errors | Actor mismatch, stale revision, invalid transition, unsupported extension. |

### `EvaluateAuthorization`

| Contract item | Requirement |
| --- | --- |
| Actor | `OWNER` or `AUTHORIZATION_EVALUATOR`. |
| Pre-state | Request `SUBMITTED`; current Request and Authorization revisions available. |
| Input | Request reference, Authorization reference, Agent, Owner, scope, profile, correlation. |
| Success | Authorization Decision created. `ALLOW` moves Request to `AUTHORIZED`; `DENY` moves it to `REJECTED`; `REQUIRES_ACTION` leaves it `SUBMITTED`. |
| Idempotency | Required for the exact Request and Authorization revisions. |
| Audit | `AUTHORIZATION_EVALUATED` and any Request `STATE_TRANSITION`. |
| Errors | Identity mismatch, inactive Authorization, stale revision, version or extension failure. |

### `ConvertRequest`

| Contract item | Requirement |
| --- | --- |
| Actor | `OWNER`, authorized `AGENT`, or `REFERENCE_NODE`. |
| Pre-state | Request `AUTHORIZED` at `expected_revision`; fresh `ALLOW` decision matching all references. |
| Success | Atomic Request `CONVERTED` plus Order `CREATED`; revisions returned. |
| Idempotency | Required. Replay returns the same Order. |
| Audit | Request transition, Order creation, decision reference. |
| Errors | Decision invalid, expired, revoked, mismatched, stale revision, duplicate commitment. |

### `ConfirmOrder`

| Contract item | Requirement |
| --- | --- |
| Actor | Assigned `PROVIDER`, `OWNER`, or authorized `REFERENCE_NODE` according to profile. |
| Pre-state | Order `CREATED` at `expected_revision`. |
| Success | Order `CONFIRMED`; revision incremented. |
| Idempotency | Required. |
| Audit | `PROVIDER_ACCEPTED` or `STATE_TRANSITION`. |
| Errors | Provider mismatch, stale revision, invalid transition. |

### `CancelOrder`

| Contract item | Requirement |
| --- | --- |
| Actor | `OWNER`, assigned `PROVIDER`, or authorized `REFERENCE_NODE`. |
| Pre-state | Order `CREATED`, `CONFIRMED`, or `PROCESSING` at `expected_revision`. |
| Success | Order `CANCELLED`; revision incremented. |
| Idempotency | Required. |
| Audit | `STATE_TRANSITION` with reason. |
| Errors | Actor mismatch, stale revision, terminal Order, invalid transition. |

### `CreateExecution`

| Contract item | Requirement |
| --- | --- |
| Actor | Assigned `PROVIDER` or authorized `REFERENCE_NODE`. |
| Pre-state | Order `CONFIRMED`; no conflicting active Execution for the same profile. |
| Success | Execution created in `PENDING`, revision `0`; Order remains `CONFIRMED`. |
| Idempotency | Required. Replay returns the same Execution. |
| Audit | `OBJECT_CREATED`. |
| Errors | Provider mismatch, Order state conflict, duplicate Execution. |

### `StartExecution`

| Contract item | Requirement |
| --- | --- |
| Actor | Assigned `PROVIDER` or authorized `REFERENCE_NODE`. |
| Pre-state | Execution `PENDING` and Order `CONFIRMED` at expected revisions. |
| Success | Atomic Execution `RUNNING` and Order `PROCESSING`. |
| Idempotency | Required. |
| Audit | Two correlated state transitions. |
| Errors | Provider mismatch, stale revision, invalid transition, atomic outcome failure. |

### `CompleteExecution`

| Contract item | Requirement |
| --- | --- |
| Actor | Assigned `PROVIDER` or authorized `REFERENCE_NODE`. |
| Pre-state | Execution `RUNNING` and Order `PROCESSING` at expected revisions. |
| Success | Atomic Execution `COMPLETED` and Order `FULFILLED`. |
| Idempotency | Required. |
| Audit | `PROVIDER_RESULT_RECORDED` and correlated state transitions. |
| Errors | Provider mismatch, stale revision, invalid transition, atomic outcome failure. |

### `FailExecution`

| Contract item | Requirement |
| --- | --- |
| Actor | Assigned `PROVIDER` or authorized `REFERENCE_NODE`. |
| Pre-state | Execution `PENDING` or `RUNNING` at `expected_revision`. |
| Success | Execution `FAILED`; Order state does not automatically become successful. |
| Idempotency | Required. |
| Audit | `PROVIDER_RESULT_RECORDED` and Execution transition. |
| Errors | Provider mismatch, stale revision, invalid transition. |

The selected profile must define whether a failed Execution permits another Execution attempt or requires Order cancellation. v0.3 does not add an Order failure state.

### `CreateSettlement`

| Contract item | Requirement |
| --- | --- |
| Actor | `OWNER`, assigned `PROVIDER`, or authorized `REFERENCE_NODE`. |
| Pre-state | Related Execution `COMPLETED`; no conflicting Settlement for the selected profile. |
| Success | Settlement created in `PENDING`, revision `0`. |
| Idempotency | Required. |
| Audit | `OBJECT_CREATED`. |
| Errors | Actor mismatch, Execution state conflict, duplicate Settlement. |

Settlement represents resource or financial reconciliation. This operation defines no payment, wallet, or accounting implementation.

### `Settle`

| Contract item | Requirement |
| --- | --- |
| Actor | Authorized `OWNER`, `PROVIDER`, or `REFERENCE_NODE`. |
| Pre-state | Settlement `PENDING` at `expected_revision`. |
| Success | Settlement `SETTLED`; revision incremented. |
| Idempotency | Required. |
| Audit | `SETTLEMENT_RECORDED`. |
| Errors | Actor mismatch, stale revision, invalid transition. |

### `RefundSettlement`

| Contract item | Requirement |
| --- | --- |
| Actor | Authorized `OWNER`, `PROVIDER`, or `REFERENCE_NODE`. |
| Pre-state | Settlement `SETTLED` at `expected_revision`. |
| Success | Settlement `REFUNDED`; revision incremented. |
| Idempotency | Required. |
| Audit | `SETTLEMENT_RECORDED` and state transition. |
| Errors | Actor mismatch, stale revision, invalid transition. |

This operation records a reconciliation outcome only. It does not perform a refund or payment.

## Required Discovery Operations

### `DescribeNode`

Returns:

- implementation identity and version;
- supported SAIF versions;
- supported profiles;
- required and optional operations;
- `idempotency_retention_seconds`;
- pagination capabilities; and
- security profile identifiers.

### `ListCapabilities`

Returns Provider-neutral capability identifiers and the actor roles allowed to request them. Discovery does not imply Authorization.

### `ListExtensions`

Returns verified Extension Manifests or stable references and digests.

## Required Query Operations

### `GetObject`

Returns an object payload plus profile-level revision, SAIF version, selected extensions, and correlation references.

### `GetObjectHistory`

Returns ordered object revisions and state transitions. Results use deterministic ordering by revision, then event ID.

### `QueryAuditEvents`

Returns only events the actor is authorized to read. Results use deterministic ordering by event time, then event ID.

### `GetOperation`

Returns the current or terminal operation outcome. This supports asynchronous completion and idempotent replay.

### `GetError`

Returns a Standard Error only when the actor is authorized to access its diagnostic context.

## Pagination

List and history operations use:

- requested page size bounded by the Node’s advertised maximum;
- an opaque continuation token;
- deterministic ordering; and
- a stable snapshot or explicit notice that the result may advance during pagination.

Bindings may encode pagination differently but must preserve these semantics. Continuation tokens are untrusted input and must not expose credentials or private storage keys.

## Asynchronous Operations

An operation that cannot complete synchronously returns `PENDING`, operation ID, correlation ID, and current affected-object references.

The caller uses `GetOperation` or a binding-defined notification carrying the same result envelope. A notification mechanism does not change operation semantics.

## Binding Rules

1. A binding maps every required field losslessly.
2. Binding authentication maps to an actor before core processing.
3. Binding success does not replace a protocol outcome.
4. Binding-native errors carry or reference a Standard Error.
5. Bindings do not expose lower-level storage mutation operations.
6. HTTP paths, MCP tool names, RPC methods, and message topics are non-normative until defined by an accepted binding specification.

## Conformance

Operation conformance vectors MUST cover:

- actor role allowed and denied cases;
- every pre-state and successful post-state;
- stale revisions;
- unchanged and changed-input idempotent replay;
- Authorization Decision freshness and mismatch;
- Provider result authority;
- atomic multi-object transitions;
- asynchronous `PENDING` and terminal results;
- unsupported required extensions; and
- Standard Error and Audit Event references.

No runtime is required by this specification. Independent implementations demonstrate conformance through observable operation records and vector outcomes.
