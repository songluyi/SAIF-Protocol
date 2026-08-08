# SAIF Authorization Decision Contract v0.3

## Status

This document defines the Authorization Decision contract for the optional SAIF Reference Node v0.3 profile.

It adds a portable evaluation result without changing the meaning or required fields of v0.2 Authorization, Request, or Order objects.

## Purpose

An Authorization Decision records whether a specific revision of a Request may proceed under a specific revision of an Authorization.

The contract prevents implementations from treating a transport credential, stale policy check, or unrelated Authorization as permission to create a business commitment.

An Authorization Decision is protocol evidence. It is not an identity credential, payment approval, commercial policy engine, or legal opinion.

## Object Shape

The normative schema is [`schemas/authorization-decision.schema.json`](../../schemas/authorization-decision.schema.json).

| Field | Meaning |
| --- | --- |
| `id` | Unique decision identifier. |
| `type` | Always `authorization_decision`. |
| `created_at` | Decision object creation time. |
| `metadata` | Non-normative extension data. |
| `saif_version` | SAIF version used for evaluation. |
| `request_ref` | Request ID and exact evaluated revision. |
| `authorization_ref` | Authorization ID and exact evaluated revision. |
| `agent_id` | Agent permitted or denied by the decision. |
| `owner_id` | Accountable Owner that issued the Authorization. |
| `decision` | `ALLOW`, `DENY`, or `REQUIRES_ACTION`. |
| `evaluated_scope` | Scope values evaluated for the Request. |
| `reason_codes` | Stable reasons supporting the decision. |
| `evaluated_at` | Time at which the policy inputs were evaluated. |
| `expires_at` | Expiry for an `ALLOW` decision; otherwise optional or `null`. |
| `correlation_id` | Correlation across Request, decision, Order, errors, and Audit Events. |
| `policy_profile` | Identifier of the policy profile used for evaluation. |

## Decision Values

### `ALLOW`

The evaluated Request revision is permitted by the referenced Authorization revision.

An `ALLOW` decision MUST include a non-null `expires_at`. It does not permanently reserve authority. The decision remains usable only while all freshness checks pass.

### `DENY`

The evaluated Request is not permitted. The Request transitions from `SUBMITTED` to `REJECTED` when the evaluation is applied.

A denied decision MUST include at least one reason code. It MUST NOT be accepted by `ConvertRequest`.

### `REQUIRES_ACTION`

Additional Owner or policy action is needed. The Request remains `SUBMITTED` and no Order is created.

The action required may be represented in `reason_codes` and non-sensitive `metadata`. The contract does not define user-interface or workflow implementation.

## Evaluation Inputs

A conforming evaluation MUST resolve and verify:

1. the exact Request ID and revision;
2. the exact Authorization ID and revision;
3. the Agent identified by the Request and Authorization;
4. the accountable Owner identified by the Authorization and Agent relationship;
5. current identity and Authorization status;
6. requested action, scope, limits, and rules;
7. declared SAIF version and required extensions; and
8. the correlation ID for the operation.

Transport authentication MAY supply evidence for identity resolution, but it MUST NOT substitute for the Owner-Agent-Authorization relationship.

## Freshness and Revocation

Before `ConvertRequest` uses an `ALLOW` decision, the Reference Node MUST verify:

- the current time is earlier than `expires_at`;
- Request ID and revision still match `request_ref`;
- Authorization ID and revision still match `authorization_ref`;
- the Authorization remains active and unrevoked;
- Agent and Owner references remain active and consistent;
- the decision was produced for the selected SAIF version and profile; and
- required extensions remain supported.

A failed freshness check invalidates use of the decision. The implementation MUST return a Standard Error and MUST NOT create an Order.

## Request State Effects

Authorization evaluation has the following observable outcomes:

| Decision | Request pre-state | Request post-state | Order created |
| --- | --- | --- | --- |
| `ALLOW` | `SUBMITTED` | `AUTHORIZED` | No |
| `DENY` | `SUBMITTED` | `REJECTED` | No |
| `REQUIRES_ACTION` | `SUBMITTED` | `SUBMITTED` | No |

The decision and any Request state change MUST share the operation correlation ID and produce Audit Events.

## Binding to Request Conversion

`ConvertRequest` MUST receive or resolve:

- Request ID and current revision;
- Authorization Decision ID;
- expected Request revision;
- operation ID, idempotency key, and correlation ID; and
- required extension declarations.

The operation succeeds only when:

1. the decision is `ALLOW`;
2. the decision is fresh;
3. all decision references match current objects;
4. the Request is `AUTHORIZED`; and
5. the caller is permitted to invoke Request conversion.

Successful conversion atomically exposes Request `CONVERTED`, Order `CREATED`, their revisions, and required Audit Events. See [Action Execution Semantics](action-execution-semantics.md).

## Reason Codes

Reason codes are stable machine-readable strings. Initial profile codes include:

- `SCOPE_ALLOWED`
- `SCOPE_NOT_ALLOWED`
- `LIMIT_ALLOWED`
- `LIMIT_EXCEEDED`
- `AUTHORIZATION_INACTIVE`
- `AUTHORIZATION_REVOKED`
- `AGENT_MISMATCH`
- `OWNER_MISMATCH`
- `REQUEST_REVISION_MISMATCH`
- `AUTHORIZATION_REVISION_MISMATCH`
- `ADDITIONAL_ACTION_REQUIRED`

Implementations MAY add namespaced reason codes. Private codes MUST NOT redefine a standard code.

## Standard Errors

| Condition | Category | Code |
| --- | --- | --- |
| Request or Authorization revision mismatch | `AUTHORIZATION` | `SAIF-AUTHORIZATION-0002` |
| Decision expired | `AUTHORIZATION` | `SAIF-AUTHORIZATION-0003` |
| Authorization revoked or inactive | `AUTHORIZATION` | `SAIF-AUTHORIZATION-0004` |
| Agent or Owner mismatch | `AUTHORIZATION` | `SAIF-AUTHORIZATION-0005` |
| Decision not `ALLOW` during conversion | `AUTHORIZATION` | `SAIF-AUTHORIZATION-0006` |
| Decision replayed against another Request | `AUTHORIZATION` | `SAIF-AUTHORIZATION-0008` |

Errors MUST follow the [SAIF Standard Error Model](../error-model.md) and include the relevant decision, Request, or Authorization reference where safe.

## Audit Requirements

Evaluation MUST emit `AUTHORIZATION_EVALUATED` with:

- actor;
- Request and Authorization references and revisions;
- decision ID and result;
- reason codes;
- previous and new Request state;
- correlation ID; and
- evaluation and expiry times.

Conversion MUST emit state transition and object creation events that reference the Authorization Decision ID.

## Security and Privacy

- The decision MUST NOT contain credentials, private keys, complete prompts, or private Provider data.
- `metadata` MUST NOT override decision, scope, revision, expiry, or identity semantics.
- A decision MUST NOT be reusable for another Request, revision, Agent, Owner, or Authorization.
- An implementation MUST protect decisions from unauthorized alteration and access.

## Conformance

Conformance vectors MUST cover:

- valid `ALLOW`, `DENY`, and `REQUIRES_ACTION` decisions;
- missing or malformed required fields;
- expired decisions;
- mismatched Request and Authorization revisions;
- revoked Authorization after evaluation;
- decision replay against another Request; and
- conversion with a non-`ALLOW` decision.

The contract does not require a runtime or policy implementation. Conformance evaluates portable objects and observable outcomes.
