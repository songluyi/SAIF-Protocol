# SAIF Audit Event Model v0.2

## Purpose

The SAIF Audit Event Model defines a portable record of significant protocol activity. It allows independent implementations to attribute actions to Parties and Agents, correlate lifecycle changes, and explain outcomes without prescribing a database, ledger technology, logging platform, or commercial audit product.

An Audit Event is evidence about protocol activity. It is not itself an Authorization, Order, Execution, or Settlement object.

## Audit Event Object

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Unique Audit Event identifier. |
| `type` | string | yes | Audit event type. |
| `created_at` | string | yes | RFC 3339 event time. |
| `metadata` | object | yes | Non-normative extension data. |
| `saif_version` | string | yes | SAIF version used to interpret the event. |
| `actor` | object | yes | Party, Agent, Provider, or system that initiated the action. |
| `action` | string | yes | Stable action name. |
| `object` | object | yes | SAIF object affected by the action. |
| `outcome` | string | yes | `SUCCESS`, `FAILURE`, or `PENDING`. |
| `authorization_id` | string or null | yes | Relevant Authorization, or `null` when not applicable. |
| `previous_state` | string or null | yes | State before the action. |
| `new_state` | string or null | yes | State after the action. |
| `correlation_id` | string | yes | Identifier connecting events in one protocol interaction. |
| `details` | object | yes | Structured, non-sensitive event detail. |

## Actor Model

The `actor` object contains:

- `type` — `PARTY`, `AGENT`, `PROVIDER`, or `SYSTEM`;
- `id` — identifier of the acting entity; and
- `owner_id` — accountable owner when the actor is an Agent, otherwise optional.

An AI model name or client product is not a substitute for Agent and Owner identity.

## Audit Event Types

Recommended v0.2 event types include:

- `OBJECT_CREATED`
- `AUTHORIZATION_EVALUATED`
- `STATE_TRANSITION`
- `PROVIDER_ACCEPTED`
- `PROVIDER_RESULT_RECORDED`
- `SETTLEMENT_RECORDED`
- `ERROR_RECORDED`

Implementations may record additional namespaced events, but must not redefine the meaning of standard event types.

## State Transition Events

A state transition event records both `previous_state` and `new_state`. The pair must represent an allowed transition in the [SAIF Protocol State Model](state-machine.md).

For an action that does not change state, both fields may contain the current state or may be `null` when state is not applicable. Implementations must document which convention they use in their conformance profile.

## Authorization Attribution

When an action depends on delegated authority, the event must identify the applicable `authorization_id`. A transport credential, MCP session, or Provider account is not a replacement for the SAIF Authorization reference.

## Correlation

All Audit Events produced by one business interaction should share a `correlation_id`. Implementations may use internal trace identifiers, but a private trace format must not be required to interpret the portable event.

## Ordering and Integrity

Implementations should preserve event order for each object and correlation ID. They may add signatures, hashes, sequence numbers, or immutable storage proofs as extensions. SAIF v0.2 does not mandate a cryptographic system or ledger technology.

## Data Minimization

Audit Events must not contain:

- private keys or payment credentials;
- authentication secrets;
- complete model prompts unless explicitly required by a separate policy;
- unnecessary personal information; or
- confidential Provider implementation details.

References should be used instead of duplicating full business objects.

## Failure Events

When `outcome` is `FAILURE`, `details` should include a standard error ID or code. The corresponding error follows the [SAIF Standard Error Model](error-model.md).

## Example

See [`examples/audit-example.json`](../examples/audit-example.json).
