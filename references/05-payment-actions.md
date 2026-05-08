# Payment Actions — Refund, Void, Capture, Auth/Cap

## Refund / Void / Capture — Rules & Constraints

### When to Use Each

| Action | When | Timing | Reverses |
|--------|------|--------|---------|
| **Void** | Cancel before settlement | Same business day (varies by acquirer) | Authorization hold released immediately |
| **Refund** | Return funds after settlement | Any time after successful charge | Funds returned to customer (1-5 business days) |
| **Capture** | Collect authorized funds | After a successful Auth transaction | N/A — settles funds |

### Refund Rules by Payment Method

| Method | Full Refund | Partial Refund | Notes |
|--------|-------------|----------------|-------|
| Cards | ✅ | ✅ | Multiple partial refunds allowed up to original amount |
| Mobile Wallets | ✅ | ✅ | |
| Apple Pay | ✅ | ✅ | |
| Google Pay | ✅ | ✅ | |
| vaLU / Souhoola / most BNPL | ✅ | ✅ | |
| Tamara / Seven | ✅ | ❌ | Full refund only |
| Kiosk | ❌ | ❌ | No refunds supported |
| Bank Installments | ✅ | ✅ | |

### Capture Rules
- Capture amount must be **≤ authorized amount**
- Multiple captures allowed up to the authorized total
- After capture: use Refund (not Void) to return funds
- Un-captured remainder is voided automatically at settlement cutoff

### Partial Refund Tracking
- `refunded_amount_cents` in the transaction object tracks total refunded so far
- Each refund creates a new **child transaction** with `is_refund: true` and `has_parent_transaction: true`
- You can refund multiple times as long as total refunded ≤ original `amount_cents`



| Action | When to Use |
|--------|-------------|
| **Refund** | Return funds to customer (any time after successful charge) |
| **Void** | Cancel a transaction — mostly same business day |
| **Capture** | Collect funds from a previously authorized (Auth) transaction |

All can be done from **Dashboard → Transactions** or via API.

### Void Rules
- **Card payment method only** — wallets, BNPL, and kiosk do not support void
- Available **same business day** before settlement cutoff
- After cutoff: use Refund instead
- Cannot void a transaction that has already been captured — use Refund

### Dashboard Steps

**Refund (Full):** Transactions → select transaction → Refund button → Refund in popup

**Refund (Partial):** Transactions → select transaction → Refund button → check "Make a partial refund" → enter amount → Refund

**Void:** Transactions → select card transaction → Void button → Void in popup

**Capture:** Transactions → select Auth transaction → Capture button → enter amount → Capture

---

Use **Auth Token** (not secret key) for action endpoints.

### Get Auth Token

**POST** `https://accept.paymob.com/api/auth/tokens/`

```json
{
  "api_key": "your_api_key_from_dashboard"
}
```

Response:
```json
{
  "token": "eyJhbGci..."
}
```

Use as: `Authorization: Bearer <token>`

---

## Refund

**POST** `https://accept.paymob.com/api/acceptance/void_refund/refund`

### Authorization
```
Authorization: Bearer <auth_token>
Content-Type: application/json
```

### Request Body

```json
{
  "transaction_id": 123456789,
  "amount_cents": 5000
}
```

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `transaction_id` | integer | ✅ | Original successful transaction ID |
| `amount_cents` | integer | ✅ | Amount to refund in cents (can be partial) |

### Response
Returns the refund transaction object. Check `success: true` and `is_refunded: true`.

---

## Void

**POST** `https://accept.paymob.com/api/acceptance/void_refund/void`

Same-day cancellation. Check with Paymob if void is available after cutoff.

### Request Body

```json
{
  "transaction_id": 123456789
}
```

---

## Capture

Used after **Authorization** (Auth/Cap flow) to settle funds.

**POST** `https://accept.paymob.com/api/acceptance/capture`

### Request Body

```json
{
  "transaction_id": 123456789,
  "amount_cents": 5000
}
```

| Field | Notes |
|-------|-------|
| `transaction_id` | The **auth** transaction ID (where `is_auth: true`) |
| `amount_cents` | Amount to capture — can be less than authorized amount |

---

## Auth/Cap (Authorization & Capture)

Two-step payment flow:
1. **Authorize** — reserve funds without charging
2. **Capture** — collect reserved funds later

### How to Authorize
Pass an **Auth Integration ID** in `payment_methods` when creating the intention.
The resulting transaction will have `is_auth: true`.

### Key Auth/Cap Rules
- Capture amount can be ≤ authorized amount
- Void an auth before capturing if you need to cancel
- Auth transactions appear in callbacks with `is_auth: true`, `is_capture: false`
- Capture transactions have `is_capture: true`, `has_parent_transaction: true`

---

## Node.js Refund Example

```javascript
async function refundTransaction(transactionId, amountCents) {
  const tokenRes = await axios.post(
    'https://accept.paymob.com/api/auth/tokens/',
    { api_key: process.env.PAYMOB_API_KEY }
  );
  const token = tokenRes.data.token;

  const refundRes = await axios.post(
    'https://accept.paymob.com/api/acceptance/void_refund/refund',
    { transaction_id: transactionId, amount_cents: amountCents },
    { headers: { Authorization: `Bearer ${token}` } }
  );

  return refundRes.data;
}
```

## Python Refund Example

```python
import requests
import os

def refund_transaction(transaction_id, amount_cents):
    # Get auth token
    token_res = requests.post(
        'https://accept.paymob.com/api/auth/tokens/',
        json={'api_key': os.environ['PAYMOB_API_KEY']}
    )
    token_res.raise_for_status()
    token = token_res.json()['token']

    # Refund
    refund_res = requests.post(
        'https://accept.paymob.com/api/acceptance/void_refund/refund',
        json={'transaction_id': transaction_id, 'amount_cents': amount_cents},
        headers={'Authorization': f'Bearer {token}'}
    )
    refund_res.raise_for_status()
    return refund_res.json()

def void_transaction(transaction_id):
    token_res = requests.post(
        'https://accept.paymob.com/api/auth/tokens/',
        json={'api_key': os.environ['PAYMOB_API_KEY']}
    )
    token = token_res.json()['token']

    void_res = requests.post(
        'https://accept.paymob.com/api/acceptance/void_refund/void',
        json={'transaction_id': transaction_id},
        headers={'Authorization': f'Bearer {token}'}
    )
    void_res.raise_for_status()
    return void_res.json()
```

> → For Void/Capture rules and method support matrix, see top of this file.
> → For error handling on failed refunds, see `21-error-codes.md`.
