# Core Features — Conceptual Reference

Business-level understanding of each core feature: when to use it, how it works, and key rules.

---

## Auth/Cap (Authorization & Capture)

### What It Is
A two-step payment flow:
1. **Authorization** — reserve funds on the customer's card (no charge yet)
2. **Capture** — collect the reserved funds (full or partial amount)

### When to Use
- Final order amount is unknown at checkout (e.g. hotel room, variable service fees)
- Fulfillment happens after the order (e.g. physical goods, marketplace)
- You want to verify the card and reserve funds before confirming the order
- Partial fulfillment may occur (capture only what you shipped)

### How It Works
1. Create Intention using an **Auth integration ID** (separate from regular card ID)
2. Customer completes checkout → transaction created with `is_auth: true`
3. You confirm the final amount → call Capture API
4. Remaining un-captured amount is automatically voided at settlement

### Key Rules
- Capture amount can be **≤ authorized amount** (partial capture supported)
- You can make **multiple captures** against one authorization (up to the authorized amount)
- To cancel before capture → call **Void** (releases the hold immediately)
- After capture → use **Refund** to return funds (not Void)
- Auth transactions appear in callbacks with `is_auth: true, is_capture: false`
- Capture transactions have `is_capture: true, has_parent_transaction: true`

### Callback Identification

| Field | Auth transaction | Capture transaction |
|-------|-----------------|---------------------|
| `is_auth` | `true` | `false` |
| `is_capture` | `false` | `true` |
| `is_standalone_payment` | `false` | `false` |
| `has_parent_transaction` | `false` | `true` |


---

## Subscriptions

### What It Is
Paymob handles **automatic recurring charges** on a schedule without requiring you to manually trigger each payment.

### Key Components

| Component | What it is |
|-----------|-----------|
| **Subscription Plan** | Blueprint: defines amount, interval (in days), and number of cycles |
| **Subscription** | Instance: one customer enrolled in a plan; Paymob auto-charges on schedule |

### When to Use
- SaaS / membership fees (monthly, annual)
- Gym memberships
- Digital content platforms
- Utility or service recurring billing

### How It Works
1. Customer makes first payment via normal 3DS checkout with card saving enabled
2. You receive a card token in the callback
3. You create a Subscription Plan (once, reusable for all customers)
4. You create a Subscription linking the customer's initial transaction to the plan
5. Paymob auto-charges using MIT on each `next_billing` date
6. You receive subscription callbacks for every event (payment_success, payment_fail, suspended, etc.)

### Key Rules
- Requires a **MOTO integration ID** for the recurring charges
- Failed payments: Paymob will retry (contact support for retry configuration)
- State values: `active`, `suspended`, `cancelled`, `completed`
- `number_of_deductions: null` = unlimited recurring
- 
---

## Pay With Saved Cards

### What It Is
Store a customer's card token after first payment and use it for faster future checkouts.

### Two Transaction Types

| Type | Customer present? | 3DS required? | Use case |
|------|------------------|---------------|---------|
| **CIT** (Customer Initiated) | Yes | Yes | Returning customer at checkout selecting saved card |
| **MIT** (Merchant Initiated) | No | No (merchant liability) | Auto-charge (subscriptions, auto-renewal) |

### How Save Card Works
1. First payment: customer checks "Save card" checkbox (or `forceSaveCard: true` in Pixel)
2. Paymob sends a **TOKEN callback** to your `notification_url` alongside the transaction callback
3. You extract and store: `token`, `masked_pan`, `card_subtype`
4. On future visits: pass the token in `card_tokens` array in Create Intention
5. Paymob shows the saved card as a payment option at checkout (CIT)
6. For MIT: use the saved token directly to charge without customer present

### Key Rules
- Never store raw PANs — only Paymob tokens
- Tokens are merchant-scoped (can't be reused across different merchant accounts)
- CIT requires customer to be present and may trigger 3DS
- MIT bypasses 3DS — merchant takes liability for chargebacks
- 
---

## Split Features

### Split Amount
**One payment → multiple merchant accounts receive their share**

- Merchant receives one transaction, Paymob distributes to sub-merchants
- Customer sees a normal checkout — no split logic visible to them
- Each sub-merchant gets their `amount_cents` portion
- Child split percentage must be **≤ 97%** (to avoid negative balance on fees)
- Use case: marketplaces, commission-based platforms, revenue sharing

### Split Payment
**One payment → customer uses up to 3 cards**

- Customer splits the bill themselves across their own cards
- Each card is authorized then captured (Auth/Cap flow per card)
- Callback contains a `transactions` array with auth + capture per card
- Use case: high-value purchases, customers with card limits, corporate shared payments

### Key Rules (Both)
- Must be **enabled on your account** by Paymob — contact support@paymob.com
- Also configurable in Dashboard → Checkout Customization → Enable Split Payment
- 
---

## Transaction Inquiry & Reports

### Three Ways to Access Transaction Data

#### 1. Callbacks (Primary — always use this)
Paymob sends callbacks after every payment event. This is the **source of truth**. Use callbacks to update your DB in real time. See `04-webhooks-hmac.md` and `19-auth-token-and-callbacks.md`.

#### 2. Transaction Inquiry APIs (Fallback / manual checks)
Use when a callback was missed or you need to verify status programmatically:
- By Transaction ID: `GET /api/acceptance/transactions/{id}/`
- By Order ID or Reference: `POST /api/ecommerce/orders/transaction_inquiry/`

See `13-transaction-inquiry-apis.md`.

#### 3. Dashboard Reports (Finance / periodic)
Generate downloadable reports from the Dashboard:

**How to generate:**
1. Dashboard → Reports tab (left sidebar)
2. Choose report type: **Transactions** or **Transfers**
3. If Transactions: select status filter (All / Successful / Declined)
4. Set date range (**max 1 month** per report)
5. Press **Generate** → report queued with "Pending" status
6. Refresh — when status changes to "Ready", download the file

**Report use cases:** accounting, auditing, reconciliation, monthly summaries

> Callbacks should always be your first choice. APIs are for fallback. Reports are for periodic finance review.
