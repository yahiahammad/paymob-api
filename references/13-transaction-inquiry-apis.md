# Transaction Inquiry APIs

Retrieve full transaction details by transaction ID or order reference. Useful for reconciliation, support, and debugging.

> Postman Collection: https://github.com/PaymobAccept/API-Postman-Collections/blob/main/Transaction%20Inquiry%20API%20Final.postman_collection

## Authorization

All endpoints require a Bearer auth token.

**POST** `https://accept.paymob.com/api/auth/tokens/`
```json
{ "api_key": "your_api_key" }
```
Use as: `Authorization: Bearer <token>`

---

## Get Transaction by ID

**GET** `https://accept.paymob.com/api/acceptance/transactions/{transaction_id}/`

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `transaction_id` | string | ✅ | The Paymob transaction ID |

### Headers

```
Authorization: Bearer <auth_token>
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | number | Transaction ID |
| `pending` | boolean | Whether still awaiting completion |
| `amount_cents` | number | Amount in cents |
| `success` | boolean | Payment success status |
| `is_auth` | boolean | Authorization transaction |
| `is_capture` | boolean | Capture transaction |
| `is_standalone_payment` | boolean | Regular (non-auth/cap) payment |
| `is_voided` | boolean | Was voided |
| `is_refunded` | boolean | Was refunded |
| `is_3d_secure` | boolean | 3DS was applied |
| `integration_id` | number | Integration ID used |
| `terminal_id` | string | Terminal ID (POS) |
| `has_parent_transaction` | boolean | Part of auth/cap or split |
| `order` | object | Full order object (see below) |
| `created_at` | string | ISO creation timestamp |
| `paid_at` | string | ISO payment timestamp |
| `currency` | string | Currency code |
| `source_data` | object | `{ type, sub_type, pan }` |
| `api_source` | string | Source (e.g. `"IFRAME"`, `"OTHER"`) |
| `fees` | string | Paymob processing fees |
| `vat` | string | VAT on fees |
| `billing_data` | object | Customer billing info |
| `merchant_commission` | number | Merchant's commission |
| `accept_fees` | number | Paymob's acceptance fees |
| `is_split_payment` | boolean | Split payment transaction |
| `split_description` | array | Split details |
| `is_void` | boolean | Void transaction |
| `is_refund` | boolean | Refund transaction |
| `data` | object | Gateway-specific response data |
| `error_occured` | boolean | Error flag |
| `is_live` | boolean | Live or test |
| `refunded_amount_cents` | number | Amount refunded so far |
| `source_id` | number | Source card/wallet ID |
| `is_captured` | boolean | Capture settled |
| `captured_amount` | number | Captured amount |
| `updated_at` | string | Last update timestamp |
| `is_settled` | boolean | Funds settled to merchant |
| `owner` | number | Owner user ID |
| `parent_transaction` | string | Parent transaction ID (auth/cap) |
| `installment_info` | object | BNPL installment details |
| `card_type` | string | Card type string |
| `routing_bank` | string | Acquiring bank |

### Example Request (Node.js)

```javascript
async function getTransactionById(transactionId) {
  const token = await getAuthToken(); // see 19-auth-token-and-callbacks.md

  const res = await axios.get(
    `https://accept.paymob.com/api/acceptance/transactions/${transactionId}/`,
    { headers: { Authorization: `Bearer ${token}` } }
  );
  return res.data;
}
```

### Example Request (Python)

```python
import requests, os

def get_transaction_by_id(transaction_id):
    token = get_auth_token()  # see 19-auth-token-and-callbacks.md

    res = requests.get(
        f"https://accept.paymob.com/api/acceptance/transactions/{transaction_id}/",
        headers={'Authorization': f'Bearer {token}'}
    )
    res.raise_for_status()
    return res.json()
```

---

## Get Transaction by Order ID or Reference

**POST** `https://accept.paymob.com/api/ecommerce/orders/transaction_inquiry/`

### Headers

```
Authorization: Bearer <auth_token>
Content-Type: application/json
```

### Request Body

Pass **one** of these identifiers:

| Field | Type | Description |
|-------|------|-------------|
| `order_id` | number | Paymob order ID (from intention response or webhook) |
| `merchant_order_id` | string | Your `special_reference` from Create Intention |

```json
{
  "merchant_order_id": "ORDER-2024-001"
}
```

or

```json
{
  "order_id": 217503754
}
```

### Response

Returns an array of transaction objects associated with the order. Each object has the same structure as the By Transaction ID response above.

### Example Request (Node.js)

```javascript
async function getTransactionByReference(merchantOrderId) {
  const tokenRes = await axios.post(
    'https://accept.paymob.com/api/auth/tokens/',
    { api_key: process.env.PAYMOB_API_KEY }
  );
  const token = tokenRes.data.token;

  const res = await axios.post(
    'https://accept.paymob.com/api/ecommerce/orders/transaction_inquiry/',
    { merchant_order_id: merchantOrderId },
    { headers: { Authorization: `Bearer ${token}` } }
  );
  return res.data; // Array of transactions
}
```

### Example Request (Python)

```python
def get_transaction_by_reference(merchant_order_id):
    token = requests.post(
        'https://accept.paymob.com/api/auth/tokens/',
        json={'api_key': os.environ['PAYMOB_API_KEY']}
    ).json()['token']

    res = requests.post(
        'https://accept.paymob.com/api/ecommerce/orders/transaction_inquiry/',
        json={'merchant_order_id': merchant_order_id},
        headers={'Authorization': f'Bearer {token}'}
    )
    res.raise_for_status()
    return res.json()  # List of transaction dicts
```

> → For auth token caching pattern, see `19-auth-token-and-callbacks.md`.
> → For full transaction object field reference, see the By Transaction ID section above.
> → Prefer webhooks over polling this endpoint — see `04-webhooks-hmac.md`.

---

## When to Use Each

| Use Case | Endpoint |
|----------|----------|
| You have a Paymob transaction ID | By Transaction ID (GET) |
| You have your own order reference | By Order ID or Reference (POST) |
| You have a Paymob order ID | By Order ID or Reference (POST) |
| Reconciliation, support lookup | Either |
| Checking if an order was paid | By Order ID or Reference (POST) with `merchant_order_id` |
