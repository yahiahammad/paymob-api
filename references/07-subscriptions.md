> All URLs use Egypt base (`https://accept.paymob.com`). See `01-overview.md` for other regions.

# Subscriptions

## Overview

Paymob's subscription module lets you configure recurring billing plans. Paymob automatically charges customers periodically without requiring intervention for each renewal.

---

## Create Subscription Plan

Define the recurring billing schedule. Plans are reusable — create once, attach many customers.

**POST** `https://accept.paymob.com/api/acceptance/subscription_plans`

`Authorization: Bearer <auth_token>`

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | ✅ | Plan display name |
| `amount_cents` | number | ✅ | Amount per billing cycle in cents |
| `currency` | string | ✅ | Currency code (e.g. `"EGP"`) |
| `frequency` | number | ✅ | Billing interval in days (e.g. `30` = monthly, `365` = annual, `7` = weekly) |
| `integration_id` | number | ✅ | MOTO integration ID for auto-charges |
| `number_of_deductions` | number | ❌ | Total billing cycles; `null` or omit = unlimited |

### Example Request

```json
{
  "name": "Monthly Premium Plan",
  "amount_cents": 9900,
  "currency": "EGP",
  "frequency": 30,
  "integration_id": 999001,
  "number_of_deductions": 12
}
```

### Response

Returns a plan object including:

| Field | Description |
|-------|-------------|
| `id` | **Plan ID** — use this when creating subscriptions |
| `name` | Plan name |
| `amount_cents` | Amount per cycle |
| `frequency` | Interval in days |
| `number_of_deductions` | Total cycles (null = unlimited) |
| `integration_id` | MOTO integration used |

---

## Create Subscription

Attach a customer to a plan (after their initial payment with a saved card).

**POST** `https://accept.paymob.com/api/acceptance/subscriptions`

### Request Body

| Field | Description |
|-------|-------------|
| `plan_id` | ID of the subscription plan |
| `integration_id` | MOTO integration ID |
| `initial_transaction` | Transaction ID from first/initial payment |
| `client_info` | Object: `{ email, full_name, phone_number }` |
| `starts_at` | ISO date string for first billing |
| `webhook_url` | URL to receive subscription event callbacks |

---

## Create Subscription

After a customer completes an initial payment (CIT) with a saved card, attach them to a plan to start recurring billing.

**POST** `https://accept.paymob.com/api/acceptance/subscriptions/`

`Authorization: Bearer <auth_token>`

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `plan_id` | number | ✅ | ID of the subscription plan (from Create Subscription Plan) |
| `integration_id` | number | ✅ | MOTO integration ID for future auto-charges |
| `initial_transaction` | number | ✅ | Transaction ID from the customer's first (3DS) payment — this is where the card token comes from |
| `client_info` | object | ✅ | `{ email, full_name, phone_number }` |
| `starts_at` | string | ✅ | ISO datetime — when billing should begin |
| `webhook_url` | string | ❌ | URL to receive subscription event callbacks |

### Example Request

```json
{
  "plan_id": 1186,
  "integration_id": 999001,
  "initial_transaction": 241322967,
  "client_info": {
    "email": "ahmed@example.com",
    "full_name": "Ahmed Hassan",
    "phone_number": "01012345678"
  },
  "starts_at": "2024-12-20T00:00:00",
  "webhook_url": "https://yoursite.com/webhook/subscription"
}
```

### Response

Returns a subscription object including:

| Field | Description |
|-------|-------------|
| `id` | **Subscription ID** — store this for all future subscription actions |
| `state` | `"active"`, `"suspended"`, `"cancelled"`, `"completed"` |
| `plan_id` | Plan this subscription is attached to |
| `amount_cents` | Billing amount per cycle |
| `frequency` | Billing interval in days |
| `next_billing` | ISO datetime of next charge |
| `starts_at` | Start datetime |
| `integration` | MOTO integration ID used |
| `initial_transaction` | Original CIT transaction ID |
| `client_info` | Customer info object |

### Full Subscription Setup Flow

```
1. Customer pays first time (normal 3DS card payment with save card enabled)
   → Paymob sends card TOKEN callback → you extract and store the token

2. Create Subscription Plan (POST /api/acceptance/subscription_plans)
   → get plan_id

3. Create Subscription (POST /api/acceptance/subscriptions)
   → pass plan_id + initial_transaction (from step 1) + MOTO integration_id
   → get subscription_id

4. Paymob auto-charges on next_billing date using the saved card (MIT)
   → you receive subscription callback on webhook_url

5. Handle subscription events (payment_success, payment_fail, suspended, etc.)
```

### Node.js Example

```javascript
async function createSubscription({ planId, initialTransactionId, customer }) {
  const token = await getAuthToken(); // see 19-auth-token-and-callbacks.md

  const res = await axios.post(
    'https://accept.paymob.com/api/acceptance/subscriptions/',
    {
      plan_id: planId,
      integration_id: parseInt(process.env.PAYMOB_MOTO_INTEGRATION_ID),
      initial_transaction: initialTransactionId,
      client_info: {
        email: customer.email,
        full_name: customer.name,
        phone_number: customer.phone,
      },
      starts_at: new Date().toISOString(),
      webhook_url: `${process.env.BASE_URL}/webhook/subscription`,
    },
    { headers: { Authorization: `Bearer ${token}` } }
  );

  return res.data; // { id, state, next_billing, ... }
}
```

---

## Plan Actions (via API)

| Action | Description |
|--------|-------------|
| Update plan | Change `number_of_deductions`, `amount_cents`, or `integration_id` |
| Suspend plan | Pause recurring billing temporarily |
| Resume plan | Restart a suspended plan |
| List plans | Get all plans for your account |

---

## Subscription Actions (via API)

All endpoints: `Authorization: Bearer <auth_token>` — see `19-auth-token-and-callbacks.md`.
Base: `https://accept.paymob.com/api/acceptance/subscriptions/{subscription_id}/`

### Update Subscription

**PUT** `https://accept.paymob.com/api/acceptance/subscriptions/{subscription_id}/`

| Body Field | Type | Description |
|------------|------|-------------|
| `amount_cents` | number | New subscription amount in cents |
| `ends_at` | string | New end date (ISO datetime) |

Response fields: `id`, `client_info`, `frequency`, `plan_id`, `state`, `amount_cents`, `starts_at`, `next_billing`, `ends_at`, `suspended_at`, `resumed_at`, `webhook_url`, `integration`, `initial_transaction`

---

### List Subscription Details

**GET** `https://accept.paymob.com/api/acceptance/subscriptions/{subscription_id}/`

Returns full subscription object for one subscription.

---

### Suspend Subscription

**POST** `https://accept.paymob.com/api/acceptance/subscriptions/{subscription_id}/suspend/`

No request body required. Sets state to `"suspended"` and stops future billing.

---

### Resume Subscription

**POST** `https://accept.paymob.com/api/acceptance/subscriptions/{subscription_id}/resume/`

No request body required. Resumes a suspended subscription.

---

### Cancel Subscription

**POST** `https://accept.paymob.com/api/acceptance/subscriptions/{subscription_id}/cancel/`

Permanently cancels the subscription. Cannot be undone.

---

### List Subscription Cards

**GET** `https://accept.paymob.com/api/acceptance/subscriptions/{subscription_id}/cards/`

Returns all saved cards attached to the subscription (primary + secondary).

---

### Add Secondary Card

Allows adding an additional card as a backup/secondary for the subscription.

Refer to the Paymob dashboard or contact support for the CIT flow used to capture and attach a new card token to an existing subscription.

---

### Delete Secondary Card

**POST** `https://accept.paymob.com/api/acceptance/subscriptions/{subscription_id}/cards/{card_id}/delete/`

Removes a secondary card from the subscription.

---

### Change Subscription Primary Card

**POST** `https://accept.paymob.com/api/acceptance/subscriptions/{subscription_id}/cards/{card_id}/set-primary/`

Promotes a secondary card to be the primary billing card. On next billing cycle, Paymob will charge this card.

---

### Register Webhook

**POST** `https://accept.paymob.com/api/acceptance/subscriptions/{subscription_id}/register-webhook/`

| Body Field | Type | Description |
|------------|------|-------------|
| `webhook_url` | string | URL to receive subscription event callbacks |

---

### Last Transaction for Subscription

**GET** `https://accept.paymob.com/api/acceptance/subscriptions/{subscription_id}/last-transaction/`

Returns the most recent transaction object for this subscription.

---

### List Subscription Transactions

**GET** `https://accept.paymob.com/api/acceptance/subscriptions/{subscription_id}/transactions/`

Returns all transactions associated with this subscription (all billing cycles).

---

## Subscription Callback

Triggered by any subscription state change. Sent as POST to your `webhook_url`.

```json
{
  "paymob_request_id": "df9e4ecf-12e0-4925-b258-65423f32bc98",
  "subscription_data": {
    "id": 1264,
    "plan_id": 1186,
    "state": "suspended",
    "amount_cents": 330,
    "frequency": 365,
    "next_billing": "2024-12-20",
    "starts_at": "2024-12-20",
    "client_info": {
      "email": "test@test.com",
      "full_name": "Ahmed Hassan",
      "phone_number": "01010101010"
    },
    "initial_transaction": 241322967
  },
  "trigger_type": "suspended",
  "hmac": "dd5b3018..."
}
```

**`trigger_type` values:** `"created"`, `"suspended"`, `"resumed"`, `"payment_success"`, `"payment_fail"`, `"completed"`

### HMAC for Subscription Callbacks

HMAC is in the **body** (not query param):

```javascript
const str = `${trigger_type}for${subscription_data.id}`;
// e.g. "suspendedfor1264"
const hmac = crypto.createHmac('sha512', HMAC_SECRET).update(str).digest('hex');
// Compare with callback body's `hmac` field
```

---

## Cross-References

- Initial CIT transaction + card token required → `06-saved-cards.md`
- Subscription callback HMAC calculation → `04-webhooks-hmac.md`
- Auth token needed for all subscription endpoints → `19-auth-token-and-callbacks.md`
- Business-level flow and use cases → `22-core-features-conceptual.md`
- Subscription failure handling → `23-gotchas.md`
