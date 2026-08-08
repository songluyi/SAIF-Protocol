# Agent Financial Identity Model v0.1.1

## Purpose

The Agent Financial Identity (AFI) model represents an AI agent or autonomous machine as an authorized economic executor. It binds the agent to an accountable human or organizational owner and to the permissions that govern its activity.

An AFI record is not legal personhood, asset ownership, a bank account, or an independent financial identity. It is a control and attribution record used to verify that an agent action is owner-authorized, budget-constrained, risk-checked, and auditable.

## Model Principles

- **Owner anchored:** every agent identity resolves to a verified human or organization.
- **Executor role:** the agent acts on behalf of the owner and does not own the delegated funds.
- **Least authority:** permissions are explicit, narrow, and deny-by-default.
- **Budget constrained:** transaction and period limits are enforced before execution.
- **Risk governed:** approval and restriction rules apply according to context.
- **Revocable:** the owner can suspend or remove authority.
- **Auditable:** authorization decisions and transaction outcomes can be reconstructed.

## Core Identity Representation

### Agent ID

A unique and persistent identifier for the AI agent or robot. The identifier should remain stable across routine software updates and have an explicit lifecycle status such as `active`, `suspended`, or `revoked`.

### Owner Identity

A reference to the individual or organization that authorizes the agent and remains economically accountable. The identity record should include an owner identifier, owner type, and a suitable verification method.

### Authorization Scope

The actions delegated to the agent. The scope may define:

- permitted purchase or service categories;
- allowed actions and counterparties;
- single-transaction and recurring limits;
- valid time windows;
- approval thresholds; and
- prohibited or restricted actions.

### Budget Profile

The resource envelope available for authorized execution. It may include a base currency, spending account reference, single-transaction limit, daily or monthly limit, category allocation, utilization, and reset policy.

### Risk Rules

Rules that impose additional controls based on amount, merchant, category, location, jurisdiction, behavior, or anomaly signals. A risk rule may deny the request, request human approval, or allow it with enhanced monitoring.

### Transaction History

Tamper-evident references to the agent’s transaction requests and outcomes. The history should record only the data required for accountability and reconciliation; it must not contain private keys, unrestricted credentials, or unnecessary sensitive payment data.

## Minimal v0.1.1 Data Model

```json
{
  "agent_id": "home-agent-001",
  "owner_type": "individual",
  "owner_id": "user-001",
  "authorization": {
    "category": [
      "household_purchase",
      "software_subscription"
    ],
    "monthly_limit": 2000,
    "single_transaction_limit": 500
  },
  "audit_required": true
}
```

## Field Semantics

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `agent_id` | string | yes | Unique identity of the authorized AI agent or machine. |
| `owner_type` | string | yes | Type of accountable owner, initially `individual` or `organization`. |
| `owner_id` | string | yes | Reference to the verified owner identity. |
| `authorization` | object | yes | Active economic permissions delegated by the owner. |
| `authorization.category` | string array | yes | Purchase or service categories the agent may access. |
| `authorization.monthly_limit` | number | yes | Maximum aggregate spend in the budget currency per month. |
| `authorization.single_transaction_limit` | number | yes | Maximum permitted amount for one transaction. |
| `audit_required` | boolean | yes | Whether authorization and execution evidence must be recorded. |

Production profiles will additionally require currency, identity status, authorization identifier, validity period, risk rules, approval policy, spending account reference, and transaction history references.

## Extended Authorization Example

```json
{
  "agent_id": "procurement-agent-042",
  "status": "active",
  "owner_type": "organization",
  "owner_id": "company-001",
  "authorization": {
    "authorization_id": "auth-2026-042",
    "actions": [
      "request_quote",
      "create_order",
      "pay"
    ],
    "category": [
      "cloud_service",
      "software_subscription"
    ],
    "currency": "USD",
    "monthly_limit": 10000,
    "single_transaction_limit": 1000,
    "approval_required_above": 500,
    "valid_from": "2026-08-01T00:00:00Z",
    "valid_until": "2026-12-31T23:59:59Z",
    "restricted_actions": [
      "cash_withdrawal",
      "fund_transfer_to_unapproved_account"
    ]
  },
  "risk_rules": {
    "counterparty_allowlist_required": true,
    "deny_on_identity_mismatch": true,
    "notify_owner_on_each_transaction": true
  },
  "spending_account_id": "spend-001",
  "transaction_history_ref": "saif:audit:agent-042",
  "audit_required": true
}
```

## Validation Rules

1. `agent_id` must be unique within the issuing identity domain.
2. `owner_id` must resolve to a verified, active human or organization.
3. An agent must not issue or expand its own authorization.
4. A requested action must match an allowed category and action.
5. The amount must satisfy both the single-transaction limit and remaining period budget.
6. Required human or organizational approval must be recorded before execution.
7. A suspended or revoked agent must not create new economic commitments.
8. Risk restrictions override broader authorization grants.
9. Every executed transaction must reference the agent, owner, authorization, and audit record.

## Open Questions

Future versions must define canonical serialization, signatures, authorization delegation chains, privacy-preserving owner verification, key rotation, recovery, cross-registry resolution, dispute records, currency semantics, and compliance profiles for different jurisdictions.
