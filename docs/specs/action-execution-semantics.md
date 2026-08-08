# SAIF Action Execution Semantics v0.3

## Status

This document defines transport-independent execution semantics for state-changing operations in the optional SAIF Reference Node v0.3 profile.

It specifies observable behavior only. It does not mandate a runtime, database, queue, lock service, transaction engine, or programming language.

## Purpose

Independent implementations must produce the same business outcome when an operation is retried, races with another operation, or creates multiple related protocol objects.

The profile therefore standardizes:

- operation context;
- object revisions and preconditions;
- idempotency scope and replay;
- atomic observable outcomes;
- asynchronous results;
- Provider result authority;
- Standard Errors; and
- required Audit Events.

## Operation Context

Every state-changing operation MUST carry or resolve:

| Field | Requirement |
| --- | --- |
| `operation_id` | Unique identifier for this operation attempt. |
| `saif_version` | Selected SAIF version. |
| `profile` | `saif-reference-node/0.3`. |
| `correlation_id` | Identifier shared across the business interaction. |
| `idempotency_key` | Caller-selected key for deterministic replay. |
| `actor` | Acting Party, Agent, Provider, or authorized system role. |
| `target` | Target object type, ID, and expected revision when applicable. |
| `authorization_decision_id` | Required when the operation creates or advances an authorized commitment. |
| `extensions` | Required and optional extension references. |
| `payload` | Operation-specific protocol data. |

Transport authentication context remains outside the portable business object. A binding MUST map the authenticated peer to `actor` before the operation is accepted.

## Object Revisions

The v0.3 Reference Node profile maintains an integer revision for each mutable protocol object without adding a required field to v0.2 object schemas.

- The first persisted representation has revision `0`.
- Each successful state-changing operation increments the revision by exactly `1`.
- A rejected, replayed, or failed operation does not increment the revision.
- An operation that targets an existing object MUST supply `expected_revision`.
- A mismatch MUST fail with `STATE_TRANSITION` / `SAIF-STATE-0002`.

Bindings may carry revisions as envelope fields, precondition headers, or message attributes. They MUST preserve the same numeric value and semantics.

## Idempotency

### Scope

The normative idempotency scope is:

```text
(actor.id, operation name, target type, target id, idempotency_key)
```

For create operations with no target ID, `operation_id` or the proposed new object ID occupies the target ID position.

### Replay Rules

When a Node receives the same scoped idempotency key:

- if the semantic operation input is unchanged, it MUST return the original outcome and MUST NOT repeat state mutation or Audit Event creation;
- the returned status is `REPLAYED`, with references to the original result;
- if the semantic input differs, it MUST reject the operation with `PROTOCOL` / `SAIF-PROTOCOL-0003`; and
- a retry after a `PENDING` result MUST return the current operation outcome using the same operation identity.

The comparison is over operation field values as interpreted by the selected binding profile. Until a canonical serialization profile exists, implementations MUST NOT claim cross-binding byte-level equivalence.

### Retention

A Reference Node MUST advertise `idempotency_retention_seconds` through capability discovery.

The v0.3 Reference Node profile requires a minimum of 86,400 seconds. A Node MAY advertise a longer interval. It MUST NOT reuse or forget a key earlier than the advertised interval.

After expiry, callers must not assume replay protection. Security or domain profiles MAY require longer retention.

## Operation Outcomes

Every state-changing operation returns one of:

| Status | Meaning |
| --- | --- |
| `APPLIED` | The operation completed and its postconditions are visible. |
| `REPLAYED` | A prior result was returned without repeating effects. |
| `PENDING` | The operation is accepted but not terminal. |
| `REJECTED` | Preconditions or Authorization did not permit the operation. |
| `CONFLICT` | Revision or concurrent state conflicted with the operation. |
| `FAILED` | A protocol or Provider failure prevented completion. |

An outcome includes:

- operation ID and status;
- selected SAIF version and profile;
- correlation ID;
- affected object references and resulting revisions;
- created object references;
- Authorization Decision reference when applicable;
- required Audit Event references;
- Standard Error reference for non-success outcomes; and
- extension versions selected for the operation.

Transport success MUST NOT be interpreted as `APPLIED` or business success without this outcome.

## Atomic Observable Outcomes

An **atomic observable outcome** means external readers cannot observe a successful subset of one protocol operation while another required subset is missing.

The following groups are atomic in the v0.3 Reference Node profile:

### Request Conversion

- Request `AUTHORIZED → CONVERTED`;
- Request revision increment;
- Order creation in `CREATED` at revision `0`;
- Authorization Decision reference;
- idempotency outcome; and
- required Audit Events.

### Execution Start

- Execution `PENDING → RUNNING`;
- Execution revision increment;
- Order `CONFIRMED → PROCESSING` and revision increment; and
- required Audit Events.

### Execution Completion

- Execution `RUNNING → COMPLETED`;
- Execution revision increment;
- Order `PROCESSING → FULFILLED` and revision increment; and
- required Audit Events.

### Settlement Transition

- Settlement state and revision change;
- idempotency outcome; and
- required Audit Events.

An implementation may use a transaction, event log, saga, or another mechanism. It MUST NOT return `APPLIED` until the complete observable group is available.

If internal recovery is in progress, it returns `PENDING` for the same operation and idempotency key. If the group cannot be completed, it returns `FAILED` with `PROTOCOL` / `SAIF-PROTOCOL-0004` and MUST prevent the partial group from being represented as successful.

## Concurrency

For two operations targeting the same object revision:

1. at most one may apply a conflicting state change;
2. the winner increments the object revision;
3. the loser receives `CONFLICT` with `SAIF-STATE-0002`;
4. neither operation may overwrite the other’s history; and
5. only valid terminal state combinations may be exposed.

Concurrency control technology is implementation-defined. The observable result is normative.

## Asynchronous Execution

A Provider operation may be asynchronous.

- acceptance returns `PENDING` and an operation ID;
- progress and terminal results reference the same operation, correlation, Order, Execution, and Provider;
- retries use the original idempotency key;
- a terminal result changes the outcome to `APPLIED` or `FAILED`; and
- a timeout does not imply Provider failure unless the selected profile defines that result.

Bindings may use polling, callbacks, events, or another mechanism. No callback transport is required by the core profile.

## Provider Result Authority

A Provider result is accepted only when:

- the authenticated peer maps to the declared Provider ID or an explicitly delegated system role;
- the Provider supports the Order capability;
- Order, Execution, operation, and correlation references match;
- the expected Execution revision matches;
- the transition is valid; and
- required extensions and security checks pass.

A Provider MUST NOT create or modify Owner Authorization, rewrite the original Request, or change unrelated objects.

An unauthorized result fails with `AUTHORIZATION` / `SAIF-AUTHORIZATION-0007`.

## Standard Errors

| Condition | Category | Code | Retryable without input change |
| --- | --- | --- | --- |
| Stale object revision | `STATE_TRANSITION` | `SAIF-STATE-0002` | No |
| Idempotency key reused with different input | `PROTOCOL` | `SAIF-PROTOCOL-0003` | No |
| Atomic outcome cannot complete | `PROTOCOL` | `SAIF-PROTOCOL-0004` | Profile-dependent |
| Unauthorized Provider result | `AUTHORIZATION` | `SAIF-AUTHORIZATION-0007` | No |
| Operation uses unsupported required extension | `EXTENSION` | `SAIF-EXTENSION-0001` | No |

Errors follow the [Standard Error Model](../error-model.md).

## Audit Requirements

An initial operation attempt emits the appropriate Audit Events for accepted decisions and state changes.

An unchanged idempotent replay MUST NOT duplicate those events. It may emit a separate operational log outside the portable Audit Event sequence, but that log is not a second business action.

Audit correlation MUST preserve:

- operation ID;
- idempotency key reference or protected digest;
- actor;
- object IDs and revisions;
- previous and new states;
- Authorization Decision ID;
- Provider ID when applicable; and
- outcome and Standard Error reference.

Raw idempotency secrets or sensitive transport credentials MUST NOT appear in portable events.

## Conformance

Conformance vectors MUST test:

- first application and unchanged replay;
- key reuse with changed input;
- expected revision success and stale revision conflict;
- concurrent terminal transitions;
- atomic Request conversion;
- atomic Execution completion;
- asynchronous `PENDING` to terminal behavior;
- authorized and unauthorized Provider results; and
- no duplicate Audit Events on replay.

This specification requires no runtime code. A conformance harness may evaluate recorded inputs and expected observable outcomes.
