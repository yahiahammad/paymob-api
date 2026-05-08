# Common Issues, Inquiries & Integration Checklist

---

## Integration Checklist (Go-Live Steps)

Complete in sequence — each step is required before moving to the next.

| Step | What Happens |
|------|-------------|
| 1. Account Creation | Merchant account created; dashboard + test environment access granted |
| 2. Paperwork Validation | Business and legal documents reviewed for compliance |
| 3. Contract Finalization | Commercial terms confirmed; contract signed by both parties |
| 4. Risk Approvals | Risk team assesses merchant based on business model and transaction activity |
| 5. Integration Cycle | Merchant integrates via APIs, SDKs, or plugins in **test environment** |
| 6. Test & Validation | Test transactions completed to confirm correct payment flow and behavior |
| 7. Technical Approval | Integration reviewed for technical and security requirements |
| 8. Live Credentials | Production credentials issued |
| 9. Going Live | Merchant starts accepting real customer payments |

---

## Common Issues

### "Unable to authorize store ownership" in Shopify

**Cause:** Logging into the Paymob Shopify app with different credentials than those used on first login.

**Fix:**
- Use the same username and password as your first successful Paymob dashboard login
- Switching accounts isn't supported directly in the app — contact support@paymob.com or your account manager

---

### Integration ID/Name does not exist (404)

**Error:**
```json
{ "detail": "Integration ID/Name does not exist in our system..." }
```

**Fix — must satisfy all 4 criteria:**
1. Integration ID belongs to your account
2. Integration ID mode (Test/Live) **matches** the secret key mode
3. Integration ID is passed as an **integer**, not a string (e.g. `[123456]` not `["123456"]`)
4. Integration ID is properly configured — if all above pass, contact support

---

### Refund Failure

**Cause:** Insufficient account balance or gateway-level issue.

**Fix:**
1. Verify your account has sufficient balance
2. Check the error message in the dashboard or API response
3. If you see a generic "Oops, something went wrong" message:
   - Open Browser Dev Tools → Network tab
   - Retry the refund
   - Find the red failed request named `refund`
   - Copy Request Headers + Response + the `x-paymob-id` header value
   - Submit to support@paymob.com

---

### "accept.paymobsolutions.com refused to connect" in Shopify

**Fix:**
1. Log in to Shopify Admin Dashboard
2. Go to Settings → Payments
3. Manage the Paymob Shopify app from there (don't access it directly)

---

### "Next user doesn't exist" when creating a Quick Link

**Fix:** Contact your account manager or support@paymob.com to configure your account for the new dashboard experience.

---

### Can't create IFrames

**Answer:** IFrame integrations are legacy. New merchants should use **Unified Checkout** instead. If you're an existing IFrame merchant needing a new IFrame, contact support — it must be handled by Paymob's team.

---

## Common Inquiries

### Does Paymob support global accounts?

No. Accounts are region-specific. You must create a separate account per region.

---

### How do I confirm payment status after a transaction completes?

Paymob sends two callbacks:

1. **Transaction Processed Callback (POST to your backend)** — authoritative source of truth. Includes `success`, `order.id`, `transaction_id`, and all transaction data. Use this to update your DB.

2. **Transaction Response Callback (redirect)** — redirects the customer back to your `redirection_url` with query parameters. Use for UX only. Mobile SDKs handle this automatically.

Always verify HMAC before trusting either callback. See `04-webhooks-hmac.md`.

---

### How do I set custom callback URLs per payment?

Pass them in the Create Intention request:
- `notification_url` → Transaction Processed Callback (POST, card integrations only)
- `redirection_url` → Transaction Response Callback (redirect, card + wallet only)

These **override** the URLs set on the integration ID for that specific intention.

---

### How do I look up a specific transaction?

Use the Transaction Inquiry API. See `13-transaction-inquiry-apis.md`.
- By Transaction ID: `GET /api/acceptance/transactions/{id}/`
- By Order ID or your reference: `POST /api/ecommerce/orders/transaction_inquiry/`

---

### How do I get the Subscription ID after creation?

Two ways:
1. From the **Subscription Callback** received after creation (`subscription_data.id`)
2. From **List Subscription Details** API using the initial 3DS transaction ID

---

### Can I force-save a customer's card without showing a checkbox?

Yes — requires a dedicated integration ID configured to enforce card saving. Contact your account manager or support@paymob.com to set this up. Also configurable in Pixel via `forceSaveCard: true` (see `03-checkout-experiences.md`).

---

### How do I configure callback URLs for an integration ID?

Dashboard → Developers → Payment Integrations → select mode (Test/Live) → click integration → Edit → fill callback URLs → Submit.
