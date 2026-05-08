# Authentication Request (Generate Auth Token)

Required prerequisite for all action APIs: Refund, Void, Capture, QuickLinks, Subscriptions, Transaction Inquiry.

## Endpoint

**POST** `https://accept.paymob.com/api/auth/tokens/`

## Request Body

| Field | Type | Required |
|-------|------|----------|
| `api_key` | string | ✅ |

```json
{ "api_key": "YOUR_API_KEY" }
```

## Response

```json
{
  "profile": { ... },
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

| Field | Description |
|-------|-------------|
| `token` | Bearer token — valid for **1 hour** |
| `profile` | Merchant profile object |

## Usage

```
Authorization: Bearer <token>
```

## Helpers (Node.js + Python)

```javascript
// Node.js — with caching (recommended)
let _token = null, _tokenExpiry = 0;
async function getAuthToken() {
  if (!_token || Date.now() > _tokenExpiry) {
    const res = await axios.post(
      'https://accept.paymob.com/api/auth/tokens/',
      { api_key: process.env.PAYMOB_API_KEY }
    );
    _token = res.data.token;
    _tokenExpiry = Date.now() + 55 * 60 * 1000; // 55 min buffer
  }
  return _token;
}
```

```python
# Python
import requests, os
def get_auth_token():
    res = requests.post(
        'https://accept.paymob.com/api/auth/tokens/',
        json={'api_key': os.environ['PAYMOB_API_KEY']}
    )
    res.raise_for_status()
    return res.json()['token']  # valid 1 hour
```

> → This token is used by: Refund (`05-payment-actions.md`), QuickLinks (`12-quicklink-apis.md`), Subscriptions (`07-subscriptions.md`), Transaction Inquiry (`13-transaction-inquiry-apis.md`).
> → The API key that generates this token is the same for test and live. See `14-getting-credentials.md`.

---

# Webhook Callbacks — Overview

## Two Callback Types

| Type | Direction | Format | Purpose |
|------|-----------|--------|---------|
| **Transaction Processed Callback** | Server-to-server POST | JSON body | Authoritative payment result — use this to update your DB |
| **Transaction Response Callback** | Client-side redirect (GET) | Query parameters | UX redirect after payment — show success/failure to user |

**The Transaction Processed Callback (POST) is the source of truth. Never rely on the redirect alone.**

## Where to Set Callback URLs

Two ways:
1. **Per-intention**: pass `notification_url` and `redirection_url` in the Create Intention request (overrides integration ID setting)
2. **Per-integration ID**: Dashboard → Developers → Payment Integrations → Edit integration → fill callback URLs

`notification_url` = Transaction Processed (POST)
`redirection_url` = Transaction Response (GET redirect)

> `notification_url` is supported only for card integration IDs. `redirection_url` supports card and wallet.

## Testing Callbacks Locally

Your `notification_url` must be a publicly accessible HTTPS URL. For local development use **ngrok** to tunnel localhost:

```bash
ngrok http 3000
# Use the generated https URL as your notification_url
```

Alternatively use **Webhook.site**, **RequestBin**, or **RequestWatch** to inspect raw callback payloads without a live server.

Always verify HMAC on every callback before trusting the data. See `04-webhooks-hmac.md`.

---

# Transaction Processed Callback — Full Payload

Sent as **POST** with JSON body to your `notification_url` after any payment event (success, fail, refund, void, capture).

## Top-Level Fields

| Field | Type | Description |
|-------|------|-------------|
| `type` | string | `"TRANSACTION"` for payment callbacks, `"TOKEN"` for card token callbacks |
| `obj` | object | Full transaction object (see below) |
| `issuer_bank` | string | Name/ID of card-issuing bank |
| `transaction_processed_callback_responses` | string | Downstream system responses |

## `obj` Fields (Transaction Object)

| Field | Type | Description |
|-------|------|-------------|
| `id` | number | **Transaction ID** — store this |
| `pending` | boolean | True if payment not yet completed (e.g. wallet OTP pending) |
| `amount_cents` | number | Transaction amount in cents |
| `success` | boolean | **True = payment succeeded** |
| `is_auth` | boolean | Authorization transaction (Auth/Cap step 1) |
| `is_capture` | boolean | Capture transaction (Auth/Cap step 2) |
| `is_standalone_payment` | boolean | Regular (non-auth/cap) payment |
| `is_voided` | boolean | Transaction was voided |
| `is_refunded` | boolean | Transaction was refunded |
| `is_3d_secure` | boolean | 3DS authentication was applied |
| `integration_id` | number | Integration ID used |
| `has_parent_transaction` | boolean | True for capture, refund, void (child of original) |
| `order.id` | number | **Paymob order ID** — correlate to your system |
| `order.merchant_order_id` | string | Your `special_reference` from Create Intention |
| `order.amount_cents` | number | Original order amount |
| `order.currency` | string | Currency code |
| `order.items` | array | Items from the intention |
| `created_at` | string | ISO timestamp |
| `currency` | string | Transaction currency |
| `source_data.type` | string | `"card"`, `"wallet"`, `"kiosk"` |
| `source_data.sub_type` | string | `"MasterCard"`, `"Visa"`, `"VodafoneCash"`, etc. |
| `source_data.pan` | string | Last 4 digits of card (masked) |
| `error_occured` | boolean | True if an error happened |
| `is_live` | boolean | True if live transaction, false if test |
| `refunded_amount_cents` | number | Total refunded so far (can be partial) |
| `captured_amount` | number | Total captured (for Auth/Cap) |
| `is_captured` | boolean | Whether capture has settled |
| `is_void` | boolean | This transaction is a void |
| `is_refund` | boolean | This transaction is a refund |
| `is_settled` | boolean | Funds settled to merchant account |
| `owner` | number | Merchant user ID |
| `parent_transaction` | string/null | Parent transaction ID (for child transactions) |
| `payment_key_claims.extra` | object | Your `extras` from Create Intention |
| `payment_key_claims.order_id` | number | Order ID |
| `payment_key_claims.billing_data` | object | Billing data from intention |
| `data` | object | Gateway-specific response (MIGS, etc.) |

## Full Payload Sample

```json
{
  "type": "TRANSACTION",
  "obj": {
    "id": 192036465,
    "pending": false,
    "amount_cents": 100000,
    "success": true,
    "is_auth": false,
    "is_capture": false,
    "is_standalone_payment": true,
    "is_voided": false,
    "is_refunded": false,
    "is_3d_secure": true,
    "integration_id": 4097558,
    "has_parent_transaction": false,
    "order": {
      "id": 217503754,
      "merchant_order_id": null,
      "amount_cents": 100000,
      "currency": "EGP",
      "items": []
    },
    "created_at": "2024-06-13T11:33:44.592345",
    "currency": "EGP",
    "source_data": {
      "pan": "2346",
      "type": "card",
      "sub_type": "MasterCard"
    },
    "payment_key_claims": {
      "extra": {},
      "order_id": 217503754,
      "amount_cents": 100000,
      "billing_data": {
        "first_name": "Ahmed",
        "last_name": "Hassan",
        "email": "ahmed@example.com",
        "phone_number": "+201125773493"
      }
    },
    "error_occured": false,
    "is_live": false,
    "refunded_amount_cents": null,
    "captured_amount": null,
    "is_captured": false,
    "is_settled": false,
    "owner": 302852,
    "parent_transaction": null
  },
  "issuer_bank": null,
  "transaction_processed_callback_responses": ""
}
```

## Transaction Response Callback (GET Redirect)

Sample redirect URL with query parameters:
```
https://yoursite.com/payment/result
  ?id=316004
  &pending=false
  &amount_cents=50000
  &success=true
  &is_auth=false
  &is_capture=false
  &is_standalone_payment=true
  &is_voided=false
  &is_refunded=false
  &is_3d_secure=true
  &integration_id=2936
  &order=378804
  &currency=EGP
  &error_occured=false
  &source_data.type=card
  &source_data.pan=2346
  &source_data.sub_type=MasterCard
  &hmac=8aa3e005de7f...
```

Parse these query parameters to display success/failure to the user. Always verify `hmac` — same calculation as POST callback but using GET field names (`id` not `obj.id`, `order_id` not `order.id`). See `04-webhooks-hmac.md`.

---

# 3DS Flow — How It Works

3D Secure (3DS) is the cardholder authentication step (bank OTP or biometric) that happens during card checkout. Paymob handles all of this automatically — you don't need to implement 3DS yourself.

## What Happens During Checkout

```
1. Customer enters card details on Unified Checkout or Pixel
2. Paymob submits to the card network
3. If 3DS is required:
   a. Customer is redirected to their bank's 3DS page (or OTP popup)
   b. Customer enters OTP / completes biometric
   c. Bank returns authentication result to Paymob
4. Paymob processes the payment
5. Paymob sends webhook to your notification_url
6. Paymob redirects customer to your redirection_url
```

## 3DS in Callbacks

| Field | Meaning |
|-------|---------|
| `is_3d_secure: true` | 3DS authentication was performed and passed |
| `is_3d_secure: false` | 3DS was not required (e.g. low-risk transaction, MOTO) |
| `success: false` + `error_occured: true` | 3DS failed or customer abandoned |
| `pending: true` | 3DS not yet completed (rare, wallet-like state) |

## 3DS Failure Handling

If `success: false`:
- The transaction exists but was not charged
- The customer's card was not debited
- You should show a "Payment failed" message and allow retry
- Create a new intention for the retry (don't reuse the same `client_secret`)

## Which Transactions Skip 3DS

- **MIT (Merchant Initiated)** — merchant charges saved card without customer present; 3DS bypassed; merchant takes chargeback liability
- **MOTO** — same as MIT
- Some low-value or low-risk transactions may be exempted by the issuer (SCA exemptions)

## 3DS and Mobile SDKs

All mobile SDKs handle 3DS natively:
- iOS: presents 3DS as a native in-app web view or redirect
- Android: same
- Flutter/React Native: same

No extra configuration needed — 3DS is automatic.
