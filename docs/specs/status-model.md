# SAIF Status Model v0.3

## Purpose

SAIF separates three status layers so transport handling, protocol action results,
and business-object lifecycle states cannot be interpreted as interchangeable.

The layers are:

1. **Protocol Status** — whether a SAIF message or result was processed;
2. **Action Outcome** — the observable result of one state-changing operation;
3. **Action Lifecycle** — the current state of a Request, Order, Execution, or
   Settlement.

Each vocabulary has one normative definition. A binding MUST preserve the layer
and field meaning and MUST NOT map one layer to another implicitly.

## Protocol Status

Protocol Status uses `schemas/common-status.schema.json`. The filename is retained
for backward compatibility; the schema title identifies its v0.3 role.

| Value | Meaning |
| --- | --- |
| `SUCCESS` | The protocol message was understood and a result was produced. |
| `PENDING` | The protocol exchange was accepted but its result envelope is not yet available. |
| `FAILURE` | Version, validation, or protocol processing prevented creation of an action result. |

Protocol Status says nothing about whether a requested business action was
approved or completed. `SUCCESS` may accompany an Action Outcome of `REJECTED`,
`CONFLICT`, or `FAILED` because the protocol request was processed correctly.

When Protocol Status is `SUCCESS`, a state-changing operation result MUST contain
an `action_outcome`. When it is `FAILURE`, the result MUST contain or reference a
Standard Error and MUST NOT claim an Action Outcome. Protocol Status `PENDING`
indicates pending protocol response production; it is distinct from an Action
Outcome of `PENDING`.

## Action Outcome

Action Outcome uses the single normative enumeration in
`schemas/action-outcome.schema.json`.

| Value | Meaning |
| --- | --- |
| `APPLIED` | All required postconditions are durably observable. |
| `REPLAYED` | A prior result was returned without repeating effects. |
| `PENDING` | The action was accepted and exists, but is not terminal. |
| `REJECTED` | Authorization or another precondition did not permit the action. |
| `CONFLICT` | Revision, concurrency, or idempotency state conflicts with the action. |
| `FAILED` | The action began evaluation or execution but could not complete. |

Action Outcome is returned only for state-changing operations. It does not change
an object's lifecycle state unless the applicable operation contract defines that
postcondition.

## Action Lifecycle

Action Lifecycle is the state of a business object. Its values are defined only
by the [Protocol State Model](../state-machine.md) and the applicable object
schema:

- Request: `DRAFT`, `SUBMITTED`, `AUTHORIZED`, `REJECTED`, `CONVERTED`;
- Order: `CREATED`, `CONFIRMED`, `PROCESSING`, `FULFILLED`, `CANCELLED`;
- Execution: `PENDING`, `RUNNING`, `COMPLETED`, `FAILED`; and
- Settlement: `PENDING`, `SETTLED`, `REFUNDED`.

An identical token in different layers does not make the layers equivalent. For
example, Execution lifecycle `PENDING` describes an Execution object, while
Action Outcome `PENDING` describes the operation that created or advanced it.

## Result Envelope

A v0.3 state-changing result exposes the layers explicitly:

```json
{
  "protocol": {
    "status": "SUCCESS",
    "code": "SAIF-0000",
    "message": "Protocol result produced",
    "timestamp": "2026-08-08T13:00:00Z",
    "metadata": {}
  },
  "action_outcome": "APPLIED",
  "affected_objects": [
    {
      "type": "request",
      "id": "req_001",
      "lifecycle_state": "SUBMITTED",
      "revision": 1
    }
  ]
}
```

Bindings MAY change field placement but MUST preserve all three names and their
separate semantics.

## Audit Projection

The v0.2 Audit Event `outcome` field retains `SUCCESS`, `PENDING`, or `FAILURE`
for backward compatibility. In a v0.3 Reference Node profile it is a projection
of the action observation:

| Action Outcome | Audit Event `outcome` |
| --- | --- |
| `APPLIED`, `REPLAYED` | `SUCCESS` |
| `PENDING` | `PENDING` |
| `REJECTED`, `CONFLICT`, `FAILED` | `FAILURE` |

The exact Action Outcome MUST also be recorded as
`details.action_outcome`. Audit projection does not replace the Action Outcome in
an operation result.

## Conformance

Conformance MUST verify:

- Protocol Status `SUCCESS` with each permitted Action Outcome;
- Protocol Status `FAILURE` with a Standard Error and no Action Outcome;
- the distinction between protocol `PENDING`, action `PENDING`, and lifecycle
  `PENDING`;
- Audit Event projection with the exact Action Outcome preserved in details; and
- rejection of result envelopes that place lifecycle states in Action Outcome or
  Action Outcome values in lifecycle fields.

The canonical requirement-to-vector mapping is the
[v0.3 Reference Node Coverage matrix](../conformance.md#v03-reference-node-coverage).

This model defines protocol data semantics only. It does not define a runtime,
transport, queue, Provider workflow, commerce system, or payment system.
