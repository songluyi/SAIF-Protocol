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
| `outcome` | string | yes | Protocol-status projection defined by the v0.3 Status Model. |
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

The v0.3 API roles map to portable Audit Event actors as follows:

| API role | Audit actor type | Required retained identity |
| --- | --- | --- |
| `OWNER` | `PARTY` | Resolved Party ID. |
| `AGENT` | `AGENT` | Agent ID and accountable `owner_id`. |
| `PROVIDER` | `PROVIDER` | Resolved Provider ID. |
| `AUTHORIZATION_EVALUATOR` | `SYSTEM` | System ID and role in `details.actor_role`. |
| `REFERENCE_NODE` | `SYSTEM` | Node identity and role in `details.actor_role`. |
| `AUDITOR` | `SYSTEM` or `PARTY` | Resolved identity and role in `details.actor_role`. |

Bindings MUST NOT record only an API role when a resolved Party, Agent, Provider,
or system identity is available.

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

For the v0.3 Reference Node profile, an authorized execution trace MUST follow
the [Audit Correlation](specs/audit-correlation.md) contract. Operation,
Request, Authorization Decision, Order, Execution, Provider, outcome, error,
and Audit Event references MUST form one consistent correlation chain. Complete
trace evidence validates against
`schemas/audit-correlation.schema.json`.

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

## v0.3 Status Projection

The v0.3 Reference Node profile uses the [SAIF Status Model](specs/status-model.md)
to distinguish Protocol Status, Action Outcome, and business-object lifecycle.
Audit Event `outcome` retains the Protocol Status vocabulary for backward
compatibility. When an event represents a state-changing operation,
`details.action_outcome` MUST preserve the exact Action Outcome.

## Example

See [`examples/audit-example.json`](../examples/audit-example.json).
