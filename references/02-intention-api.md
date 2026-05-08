> All URLs use Egypt base (`https://accept.paymob.com`). See `01-overview.md` for other regions.

# Paymob Intention API

Every payment flow starts here — backend only.

## Create Intention

**POST** `https://accept.paymob.com/v1/intention/`

### Authorization
```
Authorization: Token <your_secret_key>
Content-Type: application/json
```

### Request Body

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `amount` | number | ✅ | Total in **cents** (e.g. EGP 50.00 = `5000`) |
| `currency` | string | ✅ | Must match Integration ID currency (e.g. `"EGP"`) |
| `payment_methods` | array | ✅ | Integration IDs as integers (`[1256]`) or names (`["card"]`). Test IDs with test secret key. |
| `items` | array | ✅ | Array of item objects (see below) |
| `billing_data` | object | ✅ (partial) | Customer info (see below) |
| `extras` | object | ❌ | Custom key-value pairs returned in callback under `payment_key_claims` |
| `special_reference` | string | ❌ | Your internal order reference; returned as `merchant_order_id` in callback |
| `expiration` | number | ❌ | Seconds until intention expires |
| `notification_url` | string | ❌ | Webhook URL for transaction result (card integrations only) |
| `redirection_url` | string | ❌ | Redirect URL after payment (card + wallet only) |

#### items[] object

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `name` | string | ✅ | Max 50 chars |
| `amount` | number | ✅ | In cents. Sum of all items must equal total `amount` |
| `description` | string | ❌ | Max 255 chars |
| `quantity` | number | ❌ | Number of units |

#### billing_data object

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `first_name` | string | ✅ | Max 50 chars |
| `last_name` | string | ✅ | Max 50 chars |
| `email` | string | ✅ | Customer email |
| `phone_number` | string | ✅ | International or domestic format |
| `country` | string | ❌ | Country name |

### Example Request

```json
{
  "amount": 5000,
  "currency": "EGP",
  "payment_methods": [123456],
  "items": [
    {
      "name": "Burger Meal",
      "amount": 5000,
      "description": "Double cheeseburger combo",
      "quantity": 1
    }
  ],
  "billing_data": {
    "first_name": "Ahmed",
    "last_name": "Hassan",
    "email": "ahmed@example.com",
    "phone_number": "01012345678"
  },
  "special_reference": "ORDER-2024-001",
  "notification_url": "https://yoursite.com/webhook/paymob",
  "redirection_url": "https://yoursite.com/payment/result"
}
```

### Response (201 Created)

Full response object fields:

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | **Intention ID** — use for correlation (alternative to order ID) |
| `client_secret` | string | **Critical** — pass to Unified Checkout or Pixel. Expires in **1 hour**. |
| `intention_order_id` | number | **Paymob Order ID** — returned in all webhooks as `order.id` |
| `status` | string | `"unconfirmed"` initially; changes as payment progresses |
| `confirmed` | boolean | Whether payment has been confirmed |
| `amount_cents` | number | Total amount in cents |
| `currency` | string | Currency code |
| `payment_methods` | array | Integration IDs included in this intention |
| `payment_keys` | array | Array of payment key objects (one per integration ID) — see below |
| `special_reference` | string | Your `special_reference` echoed back (appears as `merchant_order_id` in callbacks) |
| `extras` | object | Your `extras` echoed back |
| `expiration` | number | Expiry in seconds |
| `notification_url` | string | Your webhook URL |
| `redirection_url` | string | Your redirect URL |

#### payment_keys[] structure

Each object in `payment_keys` corresponds to one integration ID:

| Field | Description |
|-------|-------------|
| `key` | **Payment token** — required for MIT (`payment_token` in `/api/acceptance/pay`) |
| `integration` | Integration ID this key is for |
| `next_payment_intention` | Reference for next payment using this same order |

```json
{
  "id": "pi_test_abc123",
  "client_secret": "egy_csk_test_XXXX",
  "intention_order_id": 98765432,
  "status": "unconfirmed",
  "confirmed": false,
  "amount_cents": 5000,
  "currency": "EGP",
  "payment_keys": [
    {
      "key": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "integration": 123456,
      "next_payment_intention": "pi_test_xyz789"
    }
  ],
  "special_reference": "ORDER-2024-001",
  "extras": {},
  "notification_url": "https://yoursite.com/webhook/paymob",
  "redirection_url": "https://yoursite.com/payment/result"
}
```

### Common Errors

**404 — Wrong/misconfigured Integration ID**
```json
{
  "detail": "Integration ID/Name does not exist in our system. You can find the list of Integration ID's/Names from Merchant Dashboard under Developers → Payment Integrations Tab"
}
```
**Fix**: Ensure the integration ID status (test/live) matches the secret key status.

**400 — Missing item name or amount**
```json
{ "items": { "name": ["This field is required."] } }
{ "items": { "amount": ["This field is required."] } }
```

**400 — Missing phone number**
```json
{ "billing_data": { "phone_number": ["This field is required."] } }
```

---

## Update Intention

Used primarily with **Pixel (embedded checkout)** to update order details after intention creation.

**PUT** `https://accept.paymob.com/v1/intention/{client_secret}/`

### Authorization
Same as Create: `Authorization: Token <secret_key>`

### Path Parameter
Pass `client_secret` as path parameter: `/v1/intention/egy_csk_test_XXXX/`

### Request Body

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `accept_order_id` | number | ✅ | Paymob order ID linked to this client_secret |
| `amount` | number | ❌ | New total in cents |
| `items` | array | ❌ | Same structure as Create; `name` and `amount` required within items |
| `billing_data` | object | ❌ | Same structure as Create; `phone_number` required within billing_data |
| `special_reference` | string | ❌ | Update your reference |
| `notification_url` | string | ❌ | |
| `redirection_url` | string | ❌ | |

### Common Errors

**400 — Missing accept_order_id**
```json
{ "accept_order_id": ["This field is required."] }
```

---

## Node.js Example (Create Intention)

```js
const axios = require('axios');

async function createIntention(orderData) {
  const response = await axios.post(
    'https://accept.paymob.com/v1/intention/',
    {
      amount: orderData.totalCents,
      currency: 'EGP',
      payment_methods: [parseInt(process.env.PAYMOB_INTEGRATION_ID)],
      items: orderData.items.map(item => ({
        name: item.name,
        amount: item.priceCents,
        quantity: item.quantity,
      })),
      billing_data: {
        first_name: orderData.customer.firstName,
        last_name: orderData.customer.lastName,
        email: orderData.customer.email,
        phone_number: orderData.customer.phone,
      },
      special_reference: orderData.orderId,
      notification_url: `${process.env.BASE_URL}/webhook/paymob`,
      redirection_url: `${process.env.BASE_URL}/payment/result`,
    },
    {
      headers: {
        Authorization: `Token ${process.env.PAYMOB_SECRET_KEY}`,
        'Content-Type': 'application/json',
      },
    }
  );
  return response.data; // { client_secret, intention_order_id, payment_keys, ... }
}
```

## Python Example (Create Intention)

```python
import requests
import os

def create_intention(order_data):
    response = requests.post(
        'https://accept.paymob.com/v1/intention/',
        json={
            'amount': order_data['total_cents'],
            'currency': 'EGP',
            'payment_methods': [int(os.environ['PAYMOB_INTEGRATION_ID'])],
            'items': [
                {
                    'name': item['name'],
                    'amount': item['price_cents'],
                    'quantity': item['quantity'],
                }
                for item in order_data['items']
            ],
            'billing_data': {
                'first_name': order_data['customer']['first_name'],
                'last_name': order_data['customer']['last_name'],
                'email': order_data['customer']['email'],
                'phone_number': order_data['customer']['phone'],
            },
            'special_reference': order_data['order_id'],
            'notification_url': f"{os.environ['BASE_URL']}/webhook/paymob",
            'redirection_url': f"{os.environ['BASE_URL']}/payment/result",
        },
        headers={
            'Authorization': f"Token {os.environ['PAYMOB_SECRET_KEY']}",
            'Content-Type': 'application/json',
        }
    )
    response.raise_for_status()
    return response.json()  # { client_secret, intention_order_id, payment_keys, ... }
```

---

## Cross-References

- Next step after getting `client_secret` → `03-checkout-experiences.md`
- To verify the payment result → `04-webhooks-hmac.md`
- For MIT using `payment_keys[0].key` → `06-saved-cards.md`
- Common errors with this endpoint → `21-error-codes.md`
- Amounts-in-cents gotcha and other pitfalls → `23-gotchas.md`
