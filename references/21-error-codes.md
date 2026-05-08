# Error Codes & Troubleshooting Reference

All Paymob API errors in one place, grouped by endpoint category.

---

## HTTP Status Codes

| Code | Meaning | When it occurs |
|------|---------|----------------|
| `200` | OK | Successful GET / action |
| `201` | Created | Intention created successfully |
| `400` | Bad Request | Missing or invalid parameters |
| `401` | Unauthorized | Invalid/missing API key or auth token |
| `403` | Forbidden | Correct credentials but no permission |
| `404` | Not Found | Wrong endpoint, wrong integration ID |
| `429` | Too Many Requests | Rate limit exceeded |
| `500` | Server Error | Paymob-side issue — retry with backoff |

---

## Intention API Errors (POST /v1/intention/)

### 404 — Wrong or Misconfigured Integration ID
```json
{
  "detail": "Integration ID/Name does not exist in our system. You can find the list of Integration ID's/Names from Merchant Dashboard under Developers → Payment Integrations Tab"
}
```
**Causes & fixes:**
- Integration ID doesn't belong to your account → check Dashboard
- Integration ID mode (test) doesn't match secret key mode (live) or vice versa → switch both to same mode
- Integration ID passed as string instead of integer → use `[123456]` not `["123456"]`
- Integration ID is disabled or not fully configured → contact support

### 400 — Missing item name
```json
{ "items": { "name": ["This field is required."] } }
```
Fix: Include `name` in every object inside the `items` array.

### 400 — Missing item amount
```json
{ "items": { "amount": ["This field is required."] } }
```
Fix: Include `amount` (in cents) in every item. Sum of all item amounts must equal the total `amount`.

### 400 — Missing phone number
```json
{ "billing_data": { "phone_number": ["This field is required."] } }
```
Fix: Include `phone_number` in `billing_data`.

### 400 — Item amounts don't sum to total
No explicit error message — transaction may behave unexpectedly.
Fix: Ensure `sum(items[].amount) === amount` (top-level total).

---

## Update Intention Errors (PUT /v1/intention/{client_secret}/)

### 400 — Missing accept_order_id
```json
{ "accept_order_id": ["This field is required."] }
```
Fix: Pass `accept_order_id` (Paymob order ID from the original Create Intention response) in the body.

### 400 — Missing item fields within items array
Same as Create Intention — `name` and `amount` required per item.

### 400 — Missing phone number in billing_data
Same as Create Intention.

---

## Auth Token Errors (POST /api/auth/tokens/)

### 401 — Invalid API Key
```json
{ "detail": "incorrect credentials" }
```
Fix: Use the correct API key from Dashboard → Settings. The API key is the same for test and live.

### Token Expiry
Auth tokens expire after **1 hour**. Use the caching pattern in `19-auth-token-and-callbacks.md` to avoid regenerating every request.

---

## Refund / Void / Capture Errors

### 400 — Insufficient balance for refund
```json
{ "message": "Insufficient balance" }
```
Fix: Ensure your Paymob account has sufficient balance. Contact support if you believe this is incorrect.

### 400 — Transaction already refunded
Fix: Check `is_refunded` and `refunded_amount_cents` on the transaction before attempting refund.

### 400 — Void window expired
Void is mostly available on the same business day. If outside the window, use Refund instead.

### 404 — Transaction not found
Fix: Verify the `transaction_id` using the Transaction Inquiry API before attempting action.

### How to debug refund failures
1. Open browser DevTools → Network tab
2. Retry the refund from dashboard
3. Find the red failed request named `refund`
4. Copy: Request Headers + Response body + `x-paymob-id` header value
5. Email to support@paymob.com

---

## QuickLink Errors

### 400 — Duplicate reference_id
```json
{ "message": "Reference ID already exists." }
```
Fix: Use a unique `reference_id` per link. Append timestamp or UUID.

### 400 — Expiry date in past
```json
{ "errors": { "expires_at": ["expires_at can't be in the past."] } }
```
Fix: Ensure `expires_at` is a future datetime.

### 401 — Invalid auth token
```json
{ "detail": "incorrect credentials" }
```
Fix: Auth tokens expire in 1 hour — generate a fresh one.

### 404 — Integration ID mismatch
Same as Intention API — integration ID mode must match `is_live`.

---

## HMAC Verification Failures

### Computed HMAC doesn't match received HMAC

Most common causes:
1. **Wrong field order** — fields must be in exact lexicographic order (see `04-webhooks-hmac.md`)
2. **Wrong field names for GET callback** — GET redirect uses `id` and `order_id`; POST body uses `obj.id` and `obj.order.id`
3. **Type coercion** — all fields must be converted to string including booleans (`"true"` not `true`)
4. **Wrong HMAC secret** — verify you're using the HMAC Secret (not API key or secret key)
5. **Extra whitespace** — don't trim or add spaces to field values

Debug approach:
```javascript
// Log the concatenated string to compare
console.log('HMAC string:', fields.join(''));
console.log('Computed:', computed);
console.log('Received:', receivedHMAC);
```

---

## Webhook Not Received

| Cause | Fix |
|-------|-----|
| `notification_url` not set | Add it to Create Intention or integration ID callback settings |
| URL not publicly accessible | Use ngrok for local dev: `ngrok http 3000` |
| URL returning non-200 | Paymob retries on non-200; ensure your handler returns `200` |
| Card integration only | `notification_url` only works with card integration IDs — not wallet or kiosk |
| Callback URL not saved | Dashboard → Developers → Payment Integrations → Edit → Submit |

---

## Subscription Errors

### Create Subscription fails
- Ensure `initial_transaction` refers to a **successful** card transaction with a saved card token
- Ensure `integration_id` is a **MOTO** integration ID (not a regular card integration)
- Ensure `plan_id` exists and belongs to your account

### MIT charge fails
- The saved card token may have expired — trigger a new CIT to refresh
- Insufficient funds on the customer's card — handle `payment_fail` subscription callback

---

## General Debugging Tips

1. **Always log `transaction_id` and `order.id`** from webhooks — these are your primary keys for support queries
2. **Use the Dashboard** (Transactions tab) to inspect raw transaction state before filing a support ticket
3. **Include `x-paymob-id`** header when contacting support — it's Paymob's internal request identifier
4. **Postman collection**: https://github.com/PaymobAccept/API-Postman-Collections — test endpoints directly
5. **Webhook.site**: Use during development to inspect raw callback payloads before wiring up your handler
