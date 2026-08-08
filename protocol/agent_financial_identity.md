# Agent Financial Identity Model v0.1

## Purpose

The Agent Financial Identity (AFI) model defines the minimum information required to represent an AI agent as a delegated economic actor. It links the agent to an accountable owner and to the policies that govern its financial activity.

An AFI record is a control and attribution object. It is not itself a wallet, payment instrument, legal person, or guarantee of creditworthiness.

## Model Principles

- **Owner-accountable:** every agent identity is linked to a verifiable human or organization.
- **Least authority:** permissions are explicit, narrow, time-bound, and revocable.
- **Budget-constrained:** financial actions are evaluated against enforceable limits.
- **Risk-aware:** policy requirements can vary by agent, action, asset, and counterparty.
- **Auditable:** transaction references and authorization decisions can be reconstructed.
- **Privacy-preserving:** records reveal only the information required by a verifier.

## Core Fields

### Agent ID

A globally unique, persistent identifier for the agent financial identity.

Recommended properties:

- URI-compatible format;
- stable across routine software upgrades;
- resolvable to an identity document or trusted registry entry;
- protected against unauthorized reassignment; and
- explicit lifecycle status such as `active`, `suspended`, or `revoked`.

Example: `saif:agent:agt_7f3a91c2`

### Owner Identity

A reference to the human or organization that owns, operates, or bears responsibility for the agent. The reference should support verification without requiring unnecessary personal or corporate data in the public identity record.

Minimum attributes:

- owner identifier;
- owner type (`human` or `organization`);
- verification method;
- relationship to the agent; and
- jurisdiction when required by policy.

### Authorization Scope

The set of financial actions delegated to the agent. Authorization is deny-by-default: an action is permitted only when it matches an active scope and all associated conditions.

A scope may constrain:

- actions, such as `quote`, `purchase`, `pay`, or `refund_request`;
- asset or currency;
- merchant category or named counterparty;
- per-transaction amount;
- valid time window;
- required human approval; and
- revocation or suspension status.

### Budget Profile

The spending envelope and allocation rules that apply to the agent. The profile should be evaluated in real time or against a strongly consistent reservation system before a transaction is committed.

A budget profile may include:

- base currency;
- per-transaction limit;
- daily, monthly, or task-level limits;
- category allocations;
- current utilization;
- reset schedule; and
- behavior when a limit is reached.

### Risk Level

A policy classification used to determine controls, monitoring, and approval requirements. The value is assigned by the owner or a trusted risk service and should be accompanied by its assessor and assessment time.

The v0.1 enum is:

- `low` — narrow, low-value, predictable activity;
- `medium` — broader or higher-value activity requiring enhanced monitoring;
- `high` — sensitive activity requiring strong approval and verification; and
- `restricted` — transactions are blocked except for explicitly approved recovery actions.

### Transaction History

A collection of tamper-evident transaction references associated with the agent identity. To reduce privacy and security risk, the identity document should store references or summaries rather than sensitive payment credentials or full transaction payloads.

Each reference may contain:

- transaction identifier;
- timestamp;
- action and amount;
- status;
- authorization decision reference;
- budget impact; and
- receipt, ledger, or audit-proof reference.

## JSON Example

```json
{
  "schemaVersion": "0.1",
  "agentId": "saif:agent:agt_7f3a91c2",
  "status": "active",
  "ownerIdentity": {
    "ownerId": "did:web:example-corp.com",
    "ownerType": "organization",
    "relationship": "operator",
    "verificationMethod": "did:web",
    "jurisdiction": "SG"
  },
  "authorizationScope": {
    "scopeId": "scope_procurement_001",
    "actions": [
      "quote",
      "purchase",
      "pay"
    ],
    "currencies": [
      "USD"
    ],
    "merchantCategories": [
      "cloud_infrastructure",
      "software_services"
    ],
    "counterpartyAllowlist": [
      "merchant:cloud-example"
    ],
    "perTransactionLimit": {
      "amount": "250.00",
      "currency": "USD"
    },
    "validFrom": "2026-08-01T00:00:00Z",
    "validUntil": "2026-12-31T23:59:59Z",
    "humanApprovalRequiredAbove": {
      "amount": "100.00",
      "currency": "USD"
    },
    "revocable": true
  },
  "budgetProfile": {
    "baseCurrency": "USD",
    "perTransactionLimit": "250.00",
    "dailyLimit": "500.00",
    "monthlyLimit": "5000.00",
    "monthlySpent": "1240.50",
    "categoryAllocations": [
      {
        "category": "cloud_infrastructure",
        "monthlyLimit": "3500.00"
      },
      {
        "category": "software_services",
        "monthlyLimit": "1500.00"
      }
    ],
    "resetSchedule": "monthly",
    "onLimitExceeded": "deny_and_notify_owner"
  },
  "riskLevel": {
    "level": "medium",
    "assessedBy": "did:web:example-corp.com:risk",
    "assessedAt": "2026-08-01T00:00:00Z",
    "controls": [
      "real_time_policy_check",
      "human_approval_above_threshold",
      "counterparty_allowlist"
    ]
  },
  "transactionHistory": [
    {
      "transactionId": "txn_01K1ABCDEF23456789",
      "timestamp": "2026-08-05T09:30:00Z",
      "action": "pay",
      "amount": "49.90",
      "currency": "USD",
      "status": "settled",
      "authorizationDecisionRef": "saif:audit:authz_81b2",
      "budgetImpact": "49.90",
      "receiptRef": "sha256:8b615f3d7d9a2f5d4f6f5d5947f8c2332b61058c2c2d4cb35f82be1a26fa4102"
    }
  ],
  "createdAt": "2026-08-01T00:00:00Z",
  "updatedAt": "2026-08-05T09:30:03Z"
}
```

## Validation Rules for v0.1

1. `agentId` must be unique and immutable for the lifetime of the identity.
2. `status` must be one of `active`, `suspended`, or `revoked`.
3. An active identity must contain a verifiable `ownerIdentity`.
4. Every transaction request must match an active authorization scope.
5. The requested amount must satisfy both scope limits and budget limits.
6. A stricter risk control overrides a broader authorization grant.
7. Revocation or suspension must prevent new financial commitments.
8. Transaction history must not contain secrets, private keys, or raw payment credentials.

## Open Questions

Future versions must define canonical serialization, signatures, key rotation, privacy-preserving disclosure, recovery, cross-registry resolution, dispute records, reputation portability, and compatibility with legal and compliance frameworks.
