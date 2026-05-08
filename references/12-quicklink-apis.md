# QuickLink APIs

QuickLinks are payment links generated via API — shareable URLs that let customers pay without a full checkout integration.

## Authorization

All endpoints require a Bearer auth token — see `19-auth-token-and-callbacks.md`.
`Authorization: Bearer <token>`

---

## Create QuickLink

**POST** `https://accept.paymob.com/api/ecommerce/quick-link/`

### Headers

| Header | Value |
|--------|-------|
| `Authorization` | `Bearer <auth_token>` |
| `Content-Type` | `application/json` |

### Request Body

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `amount_cents` | string/number | ❌ | Amount in cents; omit for open-amount links |
| `currency` | string | ❌ | e.g. `"EGP"` |
| `payment_methods` | array | ❌ | Integration IDs |
| `reference_id` | string | ❌ | Your unique reference — **must be unique per link** |
| `full_name` | string | ❌ | Pre-fill customer name |
| `email` | string | ❌ | Pre-fill customer email |
| `phone_number` | string | ❌ | Pre-fill customer phone |
| `description` | string | ❌ | Link description shown to customer |
| `payment_link_image` | string | ❌ | URL of image shown on the link page |
| `expires_at` | string | ❌ | ISO datetime (must be in future) |
| `is_live` | boolean | ❌ | `true` for live, `false` for test |
| `notification_url` | string | ❌ | Webhook URL for payment result |
| `redirection_url` | string | ❌ | Redirect URL after payment |

### Example Request

```json
{
  "amount_cents": "5000",
  "currency": "EGP",
  "payment_methods": [123456],
  "reference_id": "LINK-ORDER-001",
  "full_name": "Ahmed Hassan",
  "email": "ahmed@example.com",
  "phone_number": "01012345678",
  "description": "Payment for catering order",
  "expires_at": "2024-12-31T23:59:59",
  "is_live": false,
  "notification_url": "https://yoursite.com/webhook/paymob",
  "redirection_url": "https://yoursite.com/payment/result"
}
```

### Response (200 OK)

| Field | Description |
|-------|-------------|
| `id` | QuickLink ID |
| `shorten_url` | **The shareable payment URL** |
| `client_url` | Full checkout URL |
| `reference_id` | Your reference |
| `amount_cents` | Amount in cents |
| `state` | Link state (`"active"`, `"paid"`, `"cancelled"`, `"expired"`) |
| `currency` | Currency |
| `expires_at` | Expiry datetime |
| `paid_at` | When it was paid (null if unpaid) |
| `order` | Associated order ID |
| `client_info` | `{ first_name, last_name, email, phone_number }` |
| `description` | Description |
| `payment_link_image` | Image URL |
| `created_at` | Creation timestamp |

### Common Errors

**400 — Duplicate reference_id**
```json
{ "message": "Reference ID already exists." }
```
Fix: Use a unique `reference_id` per link.

**401 — Invalid auth token**
```json
{ "detail": "incorrect credentials" }
```
Fix: Get a fresh auth token (tokens expire in 1 hour).

**400 — Expiry date in the past**
```json
{
  "message": "expires_at - expires_at can't be in the past.",
  "errors": { "expires_at": ["expires_at can't be in the past."] }
}
```

**404 — Integration ID status mismatch**
```json
{ "detail": "Integration ID/Name does not exist in our system..." }
```
Fix: Ensure integration ID's live/test status matches `is_live`.

---

## Cancel QuickLink

**POST** `https://accept.paymob.com/api/ecommerce/quick-link/{id}/cancel/`

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | integer | ✅ | QuickLink ID from Create response |

### Headers

```
Authorization: Bearer <auth_token>
```

### Response
Returns the updated QuickLink object with `state: "cancelled"`.

---

## Node.js Example

```javascript
async function createQuickLink(orderData) {
  // Get auth token
  const tokenRes = await axios.post(
    'https://accept.paymob.com/api/auth/tokens/',
    { api_key: process.env.PAYMOB_API_KEY }
  );
  const token = tokenRes.data.token;

  // Create QuickLink
  const linkRes = await axios.post(
    'https://accept.paymob.com/api/ecommerce/quick-link/',
    {
      amount_cents: String(orderData.totalCents),
      currency: 'EGP',
      payment_methods: [parseInt(process.env.PAYMOB_INTEGRATION_ID)],
      reference_id: `ORDER-${orderData.id}`,
      full_name: orderData.customer.name,
      email: orderData.customer.email,
      phone_number: orderData.customer.phone,
      description: `Payment for order ${orderData.id}`,
      is_live: false,
      notification_url: `${process.env.BASE_URL}/webhook/paymob`,
      redirection_url: `${process.env.BASE_URL}/payment/result`,
    },
    { headers: { Authorization: `Bearer ${token}` } }
  );

  return linkRes.data.shorten_url; // Share this URL with the customer
}
```
