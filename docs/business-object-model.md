# SAIF Business Object Model v0.1

## Overview

SAIF introduces a business object model for AI-native economic activities.

The model separates:

- Intelligence
- Authorization
- Execution
- Settlement

The separation allows an AI Agent to express a business requirement without assuming that the Agent owns funds, has independent legal status, or controls the provider that fulfills the request.

Core Objects:

1. Party
2. Agent
3. Request
4. Authorization
5. Order
6. Execution
7. Settlement

## Common Object Envelope

Every SAIF v0.1 object includes:

| Field | Type | Purpose |
| --- | --- | --- |
| `id` | string | Unique identifier for the object. |
| `type` | string | Business subtype or action represented by the object. |
| `created_at` | string | RFC 3339 date-time when the object was created. |
| `metadata` | object | Optional protocol-neutral extension data. |

Object-specific fields are deliberately small in v0.1. Providers may place non-normative extension data in `metadata`, but portable implementations should not depend on private metadata fields.

## Party

### Purpose

Represent a real-world entity.

### Types

- Individual
- Organization
- AI System
- Service Provider

The Party that owns or authorizes an Agent should normally be an Individual or Organization. `ai_system` identifies a system participating in a workflow; it does not grant the system independent legal personality.

### Example

```json
{
  "id": "party_001",
  "type": "individual",
  "created_at": "2026-08-08T08:00:00Z",
  "metadata": {}
}
```

Schema: [`party.schema.json`](../schemas/party.schema.json)

## Agent

### Purpose

Represent an AI execution entity.

SAIF does not require a specific AI provider.

Agent may originate from:

- OpenAI
- Anthropic
- Google
- DeepSeek
- Local AI systems
- Robotics systems

The `provider` field is descriptive, not a protocol dependency. An Agent remains linked to its human or organizational owner through `owner_id`.

### Example

```json
{
  "id": "agent_001",
  "type": "software_agent",
  "created_at": "2026-08-08T08:01:00Z",
  "metadata": {},
  "owner_id": "party_001",
  "provider": "OpenAI"
}
```

Schema: [`agent.schema.json`](../schemas/agent.schema.json)

## Request

### Purpose

Represent a business requirement.

A Request describes desired business intent before authorization, commercial commitment, provider execution, or settlement.

### Example

User:

> “Buy one box of water.”

Request:

```json
{
  "id": "req_001",
  "type": "purchase",
  "created_at": "2026-08-08T08:02:00Z",
  "metadata": {},
  "agent_id": "agent_001",
  "status": "SUBMITTED",
  "requirement": {
    "item": "water",
    "quantity": 1
  }
}
```

Schema: [`request.schema.json`](../schemas/request.schema.json)

## Authorization

### Purpose

Represent permission for an Agent to act.

### Includes

- Owner
- Agent
- Scope
- Limits
- Rules

Authorization is issued by or on behalf of a human or organization. The Agent cannot create or expand its own authorization.

### Example

```json
{
  "id": "auth_001",
  "type": "delegated",
  "created_at": "2026-08-08T08:03:00Z",
  "metadata": {},
  "owner_id": "party_001",
  "agent_id": "agent_001",
  "scope": [
    "purchase"
  ],
  "limit": 500,
  "rules": {}
}
```

Schema: [`authorization.schema.json`](../schemas/authorization.schema.json)

## Order

### Purpose

Represent a business commitment generated from an authorized Request.

### Examples

- Purchase Order
- Service Order
- Maintenance Order

An Order references both the original Request and the Authorization used to approve it. This link makes the resulting commitment attributable and auditable.

### Example

```json
{
  "id": "order_001",
  "type": "purchase_order",
  "created_at": "2026-08-08T08:04:00Z",
  "metadata": {},
  "request_id": "req_001",
  "authorization_id": "auth_001",
  "status": "CREATED",
  "details": {
    "item": "water",
    "quantity": 1
  }
}
```

Schema: [`order.schema.json`](../schemas/order.schema.json)

## Execution

### Purpose

Represent real-world execution status.

### Examples

- Shipment
- Printing
- Robot operation
- Service completion

Execution is provider-independent. A commerce provider, document provider, or robotics provider may use different internal mechanisms while returning the same SAIF execution object shape.

### Example

```json
{
  "id": "execution_001",
  "type": "shipment",
  "created_at": "2026-08-08T08:05:00Z",
  "metadata": {},
  "order_id": "order_001",
  "provider_id": "party_provider_001",
  "status": "RUNNING",
  "result": {}
}
```

Schema: [`execution.schema.json`](../schemas/execution.schema.json)

## Settlement

### Purpose

Represent final settlement.

### Examples

- Payment
- Refund
- Invoice
- Accounting Record

Settlement is a representation of outcome, not payment processing code. SAIF does not prescribe a wallet, payment rail, currency network, or accounting provider.

### Example

```json
{
  "id": "settlement_001",
  "type": "invoice",
  "created_at": "2026-08-08T08:06:00Z",
  "metadata": {},
  "execution_id": "execution_001",
  "status": "SETTLED",
  "details": {
    "reference": "invoice_demo_001"
  }
}
```

Schema: [`settlement.schema.json`](../schemas/settlement.schema.json)

## Object Relationships

```text
Party owns or authorizes Agent
Agent creates Request
Authorization permits Request
Order commits authorized Request
Provider performs Execution
Settlement records final outcome
```

Each object has its own identity and timestamp. Implementations should preserve references between objects so an outcome can be traced back to the responsible Party, acting Agent, original Request, and applicable Authorization.

## v0.1 Boundaries

The v0.1 model defines portable business objects only. It does not define transport, storage, authentication, payment processing, marketplace behavior, provider discovery, user interfaces, or MCP implementation.
