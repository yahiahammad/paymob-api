# Split Features

Two distinct split types — both configured via the Create Intention API.

---

## Split Amount

Split a single payment across **multiple merchant accounts** (sub-merchants). Useful for marketplace models.

### How It Works
- Customer pays one total amount
- Paymob distributes portions to multiple MIDs (Merchant IDs)

### Add to Create Intention Request

```json
{
  "amount": 10000,
  "currency": "EGP",
  "payment_methods": [123456],
  "split_amounts": [
    {
      "mid": 111111,
      "amount_cents": 6000,
      "description": "Vendor A"
    },
    {
      "mid": 222222,
      "amount_cents": 4000,
      "description": "Vendor B"
    }
  ],
  "items": [...],
  "billing_data": {...}
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `split_amounts` | array | ✅ | Array of split objects |
| `split_amounts[].mid` | integer | ✅ | Connected Merchant ID to receive funds |
| `split_amounts[].amount_cents` | integer | ✅ | Amount in cents for this merchant |
| `split_amounts[].description` | string | ❌ | Description for this split |

> **Must contact Paymob support to enable Split Amount on your account.**

---

## Split Payment

Allow customer to pay with **up to 3 different cards** for a single order.

### How It Works
- Customer splits payment across multiple cards they own
- Each card payment uses Auth → Capture flow
- Enabled via Auth integration IDs

### Add to Create Intention Request

```json
{
  "amount": 10000,
  "currency": "EGP",
  "payment_methods": [123456],
  "split_payment_methods": [999001, 999002],
  "items": [...],
  "billing_data": {...}
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `split_payment_methods` | integer[] | ✅ | Array of Auth integration IDs for each card slot |

### Callback for Split Payment

The callback includes a `transactions` array. For **each sub-payment**, you receive **2 transactions**:
- One `is_auth: true` (authorization)
- One `is_capture: true` (capture)

```json
{
  "order_id": 453550057,
  "is_split_payment": true,
  "transactions": [
    {
      "id": 399618139,
      "is_auth": false,
      "is_capture": true,
      "amount_cents": 500,
      "success": true,
      "parent_transaction": 399617899,
      ...
    },
    {
      "id": 399617899,
      "is_auth": true,
      "is_capture": false,
      "amount_cents": 500,
      "success": true,
      ...
    }
  ]
}
```

> **Must contact Paymob support to enable Split Payment on your account.**

---

## Dashboard Configuration

Both split features can also be toggled from:
**Dashboard → Checkout Customization → Payment → Enable Split Payment feature**

You can control the number of cards allowed (up to 3).
