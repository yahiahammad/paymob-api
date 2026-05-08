# Webhooks & HMAC Verification

## Overview

After a transaction, Paymob sends a **POST request** to your `notification_url` with full transaction details.

**Webhooks are the source of truth. Never rely on redirects to confirm payment.**

Three callback types:
1. **Processed callback** — transaction result (success/fail)
2. **Response callback** — additional response data
3. **Card token callback** — when a card is saved

All three include an HMAC query parameter for verification.

---

## HMAC Verification

HMAC (Hash-based Message Authentication Code) uses SHA-512 + your HMAC secret to sign the data.

**Always verify HMAC before trusting any callback.**

### Steps

1. Extract specific fields from the callback body (exact order matters)
2. Concatenate them into a string
3. Compute HMAC-SHA512 using your HMAC secret key
4. Compare with the `hmac` query parameter on the callback URL

## HMAC Transaction Callback (Processed & Response)

Two callback shapes:
- **Processed callback (POST)**: data as JSON body — fields nested under `obj`
- **Response callback (GET)**: data as query parameters — fields are flat (no `obj` wrapper)

> ⚠️ **Critical difference**: The field names differ between POST and GET callbacks.
> POST uses `obj.id` and `obj.order.id` (nested). GET uses `id` and `order_id` (flat).
> Using the wrong names = HMAC mismatch. See the field table below carefully.

### Fields to Concatenate (lexicographic order — exact)

```
amount_cents
created_at
currency
error_occured
has_parent_transaction
obj.id          ← for Processed (POST) | use "id" for Response (GET)
integration_id
is_3d_secure
is_auth
is_capture
is_refunded
is_standalone_payment
is_voided
order.id        ← for Processed (POST) | use "order_id" for Response (GET)
owner
pending
source_data.pan
source_data.sub_type
source_data.type
success
```

### Concatenated String Example (Processed callback)

```
1000002024-06-13T11:33:44.592345EGPfalsefalse1920364654097558truefalsefalsefalsetruefalse217503754302852false2346MasterCardcardtrue
```

### Node.js HMAC Verification Example

```javascript
const crypto = require('crypto');

function verifyPaymobHMAC(callbackBody, receivedHMAC) {
  const hmacSecret = process.env.PAYMOB_HMAC_SECRET;

  const fields = [
    String(callbackBody.amount_cents),
    String(callbackBody.created_at),
    String(callbackBody.currency),
    String(callbackBody.error_occured),
    String(callbackBody.has_parent_transaction),
    String(callbackBody.id),
    String(callbackBody.integration_id),
    String(callbackBody.is_3d_secure),
    String(callbackBody.is_auth),
    String(callbackBody.is_capture),
    String(callbackBody.is_refunded),
    String(callbackBody.is_standalone_payment),
    String(callbackBody.is_voided),
    String(callbackBody.order?.id),
    String(callbackBody.owner),
    String(callbackBody.pending),
    String(callbackBody.source_data?.pan),
    String(callbackBody.source_data?.sub_type),
    String(callbackBody.source_data?.type),
    String(callbackBody.success),
  ];

  const concatenated = fields.join('');
  const computed = crypto
    .createHmac('sha512', hmacSecret)
    .update(concatenated)
    .digest('hex');

  return computed === receivedHMAC;
}

// Express webhook handler
app.post('/webhook/paymob', express.json(), (req, res) => {
  const obj = req.body.obj; // note: POST uses obj.id / obj.order.id (not flat)
  if (!verifyPaymobHMAC(obj, req.query.hmac)) return res.status(401).end();
  res.status(200).end(); // 200 first — prevents retries
  if (obj.success && !obj.pending) {
    // Update DB using obj.order.id or obj.order.merchant_order_id
  }
});
```

### Python HMAC Verification Example

```python
import hmac
import hashlib
import os
from flask import Flask, request, jsonify

app = Flask(__name__)

def verify_paymob_hmac(obj, received_hmac):
    hmac_secret = os.environ['PAYMOB_HMAC_SECRET'].encode()

    fields = [
        str(obj.get('amount_cents', '')),
        str(obj.get('created_at', '')),
        str(obj.get('currency', '')),
        str(obj.get('error_occured', '')),
        str(obj.get('has_parent_transaction', '')),
        str(obj.get('id', '')),
        str(obj.get('integration_id', '')),
        str(obj.get('is_3d_secure', '')),
        str(obj.get('is_auth', '')),
        str(obj.get('is_capture', '')),
        str(obj.get('is_refunded', '')),
        str(obj.get('is_standalone_payment', '')),
        str(obj.get('is_voided', '')),
        str((obj.get('order') or {}).get('id', '')),
        str(obj.get('owner', '')),
        str(obj.get('pending', '')),
        str((obj.get('source_data') or {}).get('pan', '')),
        str((obj.get('source_data') or {}).get('sub_type', '')),
        str((obj.get('source_data') or {}).get('type', '')),
        str(obj.get('success', '')),
    ]

    concatenated = ''.join(fields).encode()
    computed = hmac.new(hmac_secret, concatenated, hashlib.sha512).hexdigest()
    return hmac.compare_digest(computed, received_hmac)

@app.route('/webhook/paymob', methods=['POST'])
def paymob_webhook():
    received_hmac = request.args.get('hmac', '')
    body = request.get_json()
    obj = body.get('obj', {})

    if not verify_paymob_hmac(obj, received_hmac):
        return jsonify({'error': 'Invalid HMAC'}), 401

    response = jsonify({'received': True}), 200

    if obj.get('success') and not obj.get('pending'):
        order_id = (obj.get('order') or {}).get('id')
        print(f'Payment success, order: {order_id}')
        # Update DB here

    return response
```

### Key Fields in Callback Body

| Field | Description |
|-------|-------------|
| `id` | Transaction ID |
| `success` | `true` if payment succeeded |
| `pending` | `true` if still awaiting completion |
| `amount_cents` | Transaction amount in cents |
| `currency` | Currency code |
| `order.id` | Paymob order ID |
| `order.merchant_order_id` | Your `special_reference` from intention |
| `is_refunded` | Whether refunded |
| `is_voided` | Whether voided |
| `is_auth` | Authorization transaction |
| `is_capture` | Capture transaction |
| `source_data.type` | `"card"`, `"wallet"`, etc. |
| `source_data.sub_type` | `"MasterCard"`, `"Visa"`, etc. |
| `source_data.pan` | Last 4 digits of card |
| `payment_key_claims.extra` | Your `extras` from intention |
| `payment_key_claims` object | Contains all intention metadata |

---

## HMAC Card Token Callback

When a card is saved, Paymob sends a separate **TOKEN callback** (POST) to your `notification_url`. HMAC is in the **query parameter** `?hmac=`.

### Fields to Concatenate (lexicographic order)

```
card_subtype
created_at
email
id
masked_pan
merchant_id
order_id
token
```

### Example Concatenated String

```
MasterCard2024-11-13T12:32:23.859982test@test.com8555026xxxx-xxxx-xxxx-2346246628264064419e98aceb96f5a370ddf46460db9d555f88bf12448f80e1839b39f78ab
```

### Card Token Callback Sample

```json
{
  "type": "TOKEN",
  "obj": {
    "id": 8555026,
    "token": "e98aceb96f5a370ddf46460db9d555f88bf12448f80e1839b39f78ab",
    "masked_pan": "xxxx-xxxx-xxxx-2346",
    "merchant_id": 246628,
    "card_subtype": "MasterCard",
    "created_at": "2024-11-13T12:32:23.859982",
    "email": "test@test.com",
    "order_id": "264064419",
    "user_added": false,
    "next_payment_intention": "pi_test_2a9c29ead1734ce8ad09ae4936019992"
  }
}
```

### Node.js Card Token HMAC

```javascript
function verifyCardTokenHMAC(obj, receivedHMAC) {
  const fields = [
    String(obj.card_subtype),
    String(obj.created_at),
    String(obj.email),
    String(obj.id),
    String(obj.masked_pan),
    String(obj.merchant_id),
    String(obj.order_id),
    String(obj.token),
  ];
  const computed = crypto
    .createHmac('sha512', process.env.PAYMOB_HMAC_SECRET)
    .update(fields.join(''))
    .digest('hex');
  return computed === receivedHMAC;
}
```

---

## Webhook Testing Tool

Paymob provides a built-in Webhook Testing Tool in the dashboard to simulate callbacks without making real transactions.

**Location**: Dashboard → Developers → Webhook Testing Tool

Use it to:
- Send sample transaction callbacks to your `notification_url`
- Verify your HMAC calculation logic
- Debug webhook handler responses
- Test different transaction states (success, fail, pending)

---

## Subscription Callback HMAC

For subscription callbacks, HMAC is in the **body** (not query param) as `hmac`.

### Subscription HMAC Calculation

```
string = "{trigger_type}for{subscription_data.id}"
// e.g. "suspendedfor1264"

hmac = SHA512(string, hmacSecret)
```

### Subscription Callback Example

```json
{
  "paymob_request_id": "df9e4ecf-12e0-4925-b258-65423f32bc98",
  "subscription_data": {
    "id": 1264,
    "state": "suspended",
    "plan_id": 1186,
    "amount_cents": 330,
    "frequency": 365,
    "next_billing": "2024-12-20"
  },
  "trigger_type": "suspended",
  "hmac": "dd5b3018888d9f985..."
}
```

---

## Cross-References

- Full transaction callback payload and all field definitions → `19-auth-token-and-callbacks.md`
- Card token callback structure → this file (HMAC Card Token Callback section above)
- HMAC field names differ POST vs GET — see "Critical difference" note above
- For subscription callback HMAC → this file (Subscription Callback HMAC section above)
- Common HMAC mismatch causes → `21-error-codes.md`
- Idempotent webhook handler pattern → `23-gotchas.md`
