# SAIF Commerce Scenarios

These scenarios show how a human-authorized AI Agent or robot can access simulated commerce through SAIF. All products, budgets, wallet balances, orders, and transaction records are sandbox data.

## Scenario 1: AI Purchases Household Water

### User

> “My home water supply is running low. Please order a new box of water.”

### AI Agent

The home Agent searches the Demo Marketplace and submits a purchase request for one box of mineral water.

### SAIF Execution

#### Step 1: Verify Agent Identity

SAIF resolves `agent001` and confirms that the Agent belongs to `user001` and is active.

#### Step 2: Check Owner Authorization

SAIF confirms that the owner granted the Agent the `household_purchase` permission.

#### Step 3: Check Commerce Rules

- **Category:** Household Goods
- **Budget:** Available
- **Limit:** Allowed

The Permission Engine confirms that the product category is authorized, the amount is within the transaction limit, and sufficient monthly sandbox budget remains.

#### Step 4: Create Transaction Record

The Sandbox Wallet simulates the debit, the Demo Marketplace creates the order, and the Transaction Ledger records the owner, Agent, permission decision, product, amount, time, and outcome.

### Final Response

> “Your water order has been created successfully.”

## Scenario 2: AI Purchases Software Subscription

### User

> “Renew my AI software subscription.”

### AI Agent

The Agent identifies the subscription in the Demo Marketplace and submits a renewal request.

### SAIF Checks

- **Subscription permission:** the Agent has `software_subscription` permission.
- **Budget:** the Sandbox Wallet has enough monthly budget.
- **Renewal rule:** the product and renewal period match the owner’s authorization.

### Execution

SAIF approves the request, simulates the subscription purchase, updates the sandbox budget, and creates a Transaction Record linked to the owner and Agent.

### Final Response

> “Your AI software subscription has been renewed in the sandbox.”

## Scenario 3: Robot Maintenance Purchase

### Robot

> “Maintenance part required.”

### Robot Agent

The robot identifies an approved replacement part and submits a maintenance purchase request.

### SAIF Checks

- **Robot owner:** the Robot Agent Identity is linked to the operating company.
- **Maintenance authorization:** the owner granted `robot_maintenance` permission.
- **Spending limit:** the part cost is within the single-transaction and monthly limits.

### Execution

SAIF reserves sandbox budget, creates a simulated order with the maintenance provider, and records the complete authorization and transaction chain.

### Final Response

> “The maintenance part order has been created successfully in the sandbox.”

## Expected Demo Evidence

Each scenario should produce:

1. an Agent Identity reference;
2. an Owner Authorization reference;
3. a Permission Engine decision;
4. a Sandbox Wallet balance change;
5. a Demo Marketplace order result; and
6. a Transaction Ledger record.

Together, these artifacts demonstrate the central SAIF claim: an AI Agent can perform a commerce transaction as an authorized executor without becoming the legal or economic owner.
