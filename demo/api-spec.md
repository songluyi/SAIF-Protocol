# SAIF Agent Commerce Demo API

## Overview

This document defines the simulated API for SAIF Agent Commerce Demo v0.1. The API creates demo Agents, evaluates permissions, and executes purchases against a Sandbox Wallet and Demo Marketplace.

All endpoint data is synthetic. No endpoint processes real payments or creates real commercial obligations.

Requests and responses use `application/json`.

## Create Agent

`POST /agent/create`

Creates an AI Agent Identity and an owner-defined sandbox authorization.

### Request

```json
{
  "owner_id": "user001",
  "agent_name": "home-assistant",
  "agent_type": "personal-ai",
  "permissions": [
    "household_purchase"
  ],
  "monthly_budget": 2000
}
```

### Response

```json
{
  "status": "created",
  "agent_id": "agent001",
  "owner_id": "user001",
  "sandbox_wallet_id": "wallet001"
}
```

The returned Agent ID must be used for permission checks and purchase requests.

## Check Permission

`GET /agent/permission`

Checks whether an AI Agent has permission for a proposed transaction.

### Query Parameters

| Parameter | Required | Description |
| --- | --- | --- |
| `agent_id` | yes | Agent Identity to evaluate. |
| `category` | yes | Requested commerce category. |
| `amount` | yes | Proposed sandbox transaction amount. |

Example request:

```text
GET /agent/permission?agent_id=agent001&category=household_purchase&amount=50
```

### Allowed Response

```json
{
  "agent_id": "agent001",
  "allowed": true,
  "category": "household_purchase",
  "amount": 50,
  "remaining_monthly_budget": 1950,
  "permission_decision_id": "SAIF-PD001"
}
```

### Denied Response

```json
{
  "agent_id": "agent001",
  "allowed": false,
  "category": "hardware_purchase",
  "amount": 500,
  "reason": "category_not_authorized",
  "permission_decision_id": "SAIF-PD002"
}
```

A permission response is advisory until the Purchase endpoint revalidates the same rules and reserves sandbox budget atomically.

## Purchase

`POST /commerce/purchase`

Executes an approved simulated purchase through the Sandbox Wallet and Demo Marketplace.

### Request

```json
{
  "agent_id": "agent001",
  "product": "mineral_water",
  "quantity": 1,
  "amount": 50
}
```

The server resolves the product category from the Demo Marketplace catalog. It does not trust an Agent-supplied category to authorize the purchase.

### System Execution

1. Verify Agent Identity.
2. Check Owner Authorization and Permission.
3. Check Sandbox Wallet balance and Budget.
4. Create the Demo Marketplace order and Transaction Record.

### Approved Response

```json
{
  "status": "approved",
  "transaction_id": "SAIF-TX001"
}
```

### Denied Response

```json
{
  "status": "denied",
  "transaction_id": "SAIF-TX002",
  "reason": "monthly_budget_exceeded"
}
```

Denied requests are recorded so the owner can audit attempted activity.

## Transaction Record

The Purchase endpoint creates a ledger entry with the following logical structure:

```json
{
  "transaction_id": "SAIF-TX001",
  "owner_id": "user001",
  "agent_id": "agent001",
  "permission_decision_id": "SAIF-PD001",
  "product": "mineral_water",
  "category": "household_purchase",
  "quantity": 1,
  "amount": 50,
  "currency": "SANDBOX_CREDIT",
  "status": "approved",
  "created_at": "2026-08-08T08:00:00Z"
}
```

## Demo Error Codes

| Code | Meaning |
| --- | --- |
| `agent_not_found` | The Agent Identity does not exist. |
| `authorization_inactive` | The owner authorization is suspended or revoked. |
| `category_not_authorized` | The product category is outside the Agent permission scope. |
| `transaction_limit_exceeded` | The amount exceeds the single-transaction limit. |
| `monthly_budget_exceeded` | The amount exceeds the remaining monthly budget. |
| `sandbox_balance_insufficient` | The Sandbox Wallet does not have enough demo balance. |
| `product_not_found` | The product does not exist in the Demo Marketplace. |

## Security and Demo Constraints

- The implementation must revalidate permission during purchase execution.
- The Agent must not be able to update its own authorization or budget.
- Wallet balance changes and transaction creation should be atomic in the sandbox.
- API logs must not contain real payment credentials or private identity data.
- Demo tokens and balances have no monetary value.
