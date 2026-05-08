# Saved Cards — Tokenization, CIT & MIT

## Overview

Paymob supports saving card details for future use via **tokens**.

| Type | Description |
|------|-------------|
| **CIT** (Customer Initiated Transaction) | Customer is present; they choose a saved card at checkout |
| **MIT** (Merchant Initiated Transaction) | Merchant charges saved card automatically (subscriptions, auto-pay) |

---

## Create Card Token

When a customer pays and "Save card" is enabled, Paymob sends a **card token callback** to your `notification_url`.

### Enabling Save Card

**Via Pixel:**
```javascript
new Pixel({
  showSaveCard: true,    // show checkbox
  forceSaveCard: true,   // save automatically without checkbox
  ...
});
```

**Via Unified Checkout:**
Enable "Save card checkbox" in Dashboard → Checkout Customization.

**Via Create Intention (for SDKs):**
Pass `card_tokens` array with existing tokens in the intention creation request.

### Card Token Callback

When a card is saved, you receive a separate callback with `card_token` data alongside the transaction callback. Extract and store:

```json
{
  "id": "tok_abc123",
  "masked_pan": "512345xxxxxx2346",
  "card_subtype": "MasterCard",
  "order_id": 98765432,
  ...
}
```

---

## CIT (Customer Initiated Transaction)

Customer selects a previously saved card at checkout. Pass the saved token to the intention:

### Create Intention with Saved Tokens

```json
{
  "amount": 5000,
  "currency": "EGP",
  "payment_methods": [123456],
  "card_tokens": ["tok_abc123"],
  "items": [...],
  "billing_data": {...}
}
```

The Unified Checkout or Pixel will display the saved card as an option.

---

## MIT (Merchant Initiated Transaction)

Charge a saved card without customer interaction. Used for recurring billing, subscription renewals, and auto-replenishment.

### Prerequisites
- A saved card token from a previous CIT (see above — extract from TOKEN callback)
- A **MOTO integration ID** (different from regular card IDs — get from account manager)
- An auth token (see `19-auth-token-and-callbacks.md`)

### Step 1 — Create Intention with MOTO Integration ID

**POST** `https://accept.paymob.com/v1/intention/`

```json
{
  "amount": 5000,
  "currency": "EGP",
  "payment_methods": [MOTO_INTEGRATION_ID],
  "items": [{ "name": "Renewal", "amount": 5000 }],
  "billing_data": {
    "first_name": "Ahmed",
    "last_name": "Hassan",
    "email": "ahmed@example.com",
    "phone_number": "01012345678"
  },
  "notification_url": "https://yoursite.com/webhook/paymob"
}
```

From the response, extract `payment_keys[0].key` — this is your `payment_token`.

### Step 2 — Charge the Saved Card

**POST** `https://accept.paymob.com/api/acceptance/pay`

```
Authorization: Bearer <auth_token>
Content-Type: application/json
```

```json
{
  "source": {
    "identifier": "tok_abc123",
    "subtype": "TOKEN"
  },
  "payment_token": "<payment_keys[0].key from intention>"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `source.identifier` | string | ✅ | The saved card token |
| `source.subtype` | string | ✅ | Always `"TOKEN"` |
| `payment_token` | string | ✅ | `payment_keys[0].key` from the intention response |

### Response
Returns a transaction object. Check `success: true` to confirm charge. Also verify via webhook callback.

### Node.js MIT Example

```javascript
async function chargeWithSavedCard(savedToken, customerData, amountCents) {
  // Step 1: Create intention with MOTO integration ID
  const intentionRes = await axios.post(
    'https://accept.paymob.com/v1/intention/',
    {
      amount: amountCents,
      currency: 'EGP',
      payment_methods: [parseInt(process.env.PAYMOB_MOTO_INTEGRATION_ID)],
      items: [{ name: 'Auto-charge', amount: amountCents }],
      billing_data: {
        first_name: customerData.firstName,
        last_name: customerData.lastName,
        email: customerData.email,
        phone_number: customerData.phone,
      },
      notification_url: `${process.env.BASE_URL}/webhook/paymob`,
    },
    { headers: { Authorization: `Token ${process.env.PAYMOB_SECRET_KEY}` } }
  );

  const paymentToken = intentionRes.data.payment_keys[0].key;

  // Step 2: Get auth token (see 19-auth-token-and-callbacks.md)
  const bearerToken = await getAuthToken();

  // Step 3: Charge saved card
  const payRes = await axios.post(
    'https://accept.paymob.com/api/acceptance/pay',
    {
      source: { identifier: savedToken, subtype: 'TOKEN' },
      payment_token: paymentToken,
    },
    { headers: { Authorization: `Bearer ${bearerToken}` } }
  );

  return payRes.data; // verify success via webhook too
}
```

---

## Key Notes

- Store token IDs and `masked_pan` in your DB — never store full card numbers
- Tokens are merchant-scoped (can't use one merchant's token on another)
- CIT requires customer presence (3DS may apply)
- MIT bypasses 3DS — merchant takes liability

---

## Cross-References

- TOKEN callback structure and HMAC → `04-webhooks-hmac.md` (HMAC Card Token section)
- For subscriptions using a saved card token → `07-subscriptions.md`
- For MIT idempotency and gotchas → `23-gotchas.md`
- MOTO integration ID — how to get it → `14-getting-credentials.md`
