# Gotchas — Common Pitfalls & Non-Obvious Rules

Things that aren't obvious from the docs but burn developers regularly.

---

## Amounts

- **Everything is in cents (piastres), not pounds/riyals/dirhams**
  - EGP 50.00 = `5000`, not `50`
  - EGP 1.00 = `100`
  - Getting this wrong means overcharging by 100x — double-check every amount

- **Sum of item amounts must equal the total amount**
  - If `amount: 5000` and items sum to `4500`, Paymob will reject or behave unexpectedly
  - No explicit error for the mismatch — just silent weirdness

---

## Integration IDs

- **Test IDs only work with test secret keys, and vice versa**
  - Mixing test ID + live key (or live ID + test key) → 404 "Integration ID not found"
  - This is the #1 cause of 404 errors for new integrations

- **Integration IDs must be integers, not strings**
  - `payment_methods: [123456]` ✅
  - `payment_methods: ["123456"]` ❌ → 404

- **Each payment method needs its own integration ID**
  - You can't use your card integration ID for wallets
  - Apple Pay and Google Pay have **live IDs only** — no test IDs exist

- **MOTO integration IDs are different from regular card IDs**
  - You need to ask your account manager for a MOTO integration ID
  - Using a regular card integration ID for MIT will fail

---

## notification_url (Webhook)

- **Only works with card integration IDs**
  - Wallet and kiosk integrations do not trigger the `notification_url` webhook
  - For wallet payments, rely on the `redirection_url` callback params

- **Must be a public HTTPS URL**
  - localhost URLs silently fail — Paymob can't reach them
  - Use ngrok, Webhook.site, or a deployed URL for testing

- **Return HTTP 200 or Paymob will retry**
  - Any non-200 response causes Paymob to retry the callback
  - Process idempotently — you may receive the same callback more than once
  - Use `transaction_id` as the idempotency key

---

## client_secret

- **Expires in 1 hour**
  - If the customer takes too long or you cache it, it will expire
  - Create a fresh intention per payment session
  - Don't store `client_secret` persistently

- **Is unique per intention / order**
  - Never reuse a `client_secret` across different orders

---

## Webhook vs Redirect

- **Never update your database on the redirect**
  - The `redirection_url` is for UX only (showing success/failure to the user)
  - The redirect can be spoofed — anyone can hit your redirect URL with fake `success=true` params
  - Always verify HMAC on the webhook; the webhook POST is the source of truth

- **HMAC field names differ between POST and GET callbacks**
  - POST body uses `obj.id` and `obj.order.id`
  - GET redirect uses `id` and `order_id` (flat, no `obj` nesting)
  - Using the wrong field names = HMAC mismatch on one of the two

---

## Apple Pay

- **No test integration ID exists**
  - Apple Pay can only be tested in a live environment with real credentials
  - You can use test cards on the Apple Pay sheet in sandbox if Apple Pay sandbox is configured

- **Not supported in WebViews**
  - Apple enforces this — use the native iOS/Android SDK for in-app Apple Pay
  - Pixel embedded in a WebView will not show Apple Pay

- **Requires domain verification AND certificates**
  - Both must be done before Apple Pay will appear at checkout
  - Even one missing = Apple Pay silently hidden

---

## Google Pay

- **Not available in Egypt** (as of May 2026)
  - Passing `'google-pay'` in `paymentMethods` in Egypt will silently have no effect
  - Available: KSA, UAE, OMN

- **Requires separate Google Pay SDK included on the page**
  - Pixel alone is not enough — you must also load the Google Pay SDK

---

## Kiosk

- **No refunds supported**
  - `source_data.type = "kiosk"` transactions cannot be refunded
  - Design your business flow accordingly

- **Payment is async**
  - Customer gets a reference number, pays physically later
  - Don't confirm the order at checkout — wait for the webhook

---

## Void

- **Card payment method only**
  - Void is not supported for wallets, BNPL, or kiosk
  - For non-card methods, use Refund instead

- **Same business day only**
  - After settlement cutoff (varies by acquirer), void is no longer available
  - If the window has passed, use Refund

---

## Auth/Cap

- **Requires a separate Auth integration ID**
  - Your standard card integration ID does NOT support Auth/Cap
  - Ask your account manager for an Auth integration ID

- **Capture must happen before authorization expires**
  - Authorization expiry depends on the card network (typically 7–30 days)
  - If you miss the window, the funds are automatically released

---

## Refund

- **Tamara and Seven only support full refunds**
  - Partial refund on Tamara/Seven will be rejected
  - Check provider before attempting partial refund

- **Refunds cannot exceed original amount**
  - `refunded_amount_cents` tracks total refunded
  - Multiple partial refunds are fine as long as total ≤ original

---

## Subscriptions

- **Requires initial CIT transaction with saved card**
  - You can't create a subscription with a token from a different merchant
  - The `initial_transaction` must be a successful payment that generated a card token

- **Failed subscription payment → check webhook**
  - Paymob sends `payment_fail` subscription callback
  - Design a retry flow or notify the customer to update their card

---

## Split Amount

- **Child split must be ≤ 97%**
  - Splits leaving less than 3% for fees cause a negative balance error
  - Account for Paymob's fees when calculating split percentages

---

## Idempotency

- **Paymob does not have built-in idempotency keys on most endpoints**
  - Two identical Create Intention calls = two separate intentions
  - Use `special_reference` (your order ID) to track and deduplicate on your side
  - Your webhook handler MUST be idempotent — check if the transaction was already processed before updating DB

Pattern: return 200 immediately, then check if `transaction_id` already exists in your DB before processing. Full example in `00-quick-start.md`.

---

## Rate Limiting

- Paymob does not publicly document rate limits
- If you hit rate limits you'll receive HTTP `429 Too Many Requests`
- Best practices:
  - Don't poll Transaction Inquiry APIs in a tight loop — use webhooks instead
  - Cache auth tokens (valid 1 hour) instead of generating per-request
  - For bulk operations, add delays between requests

---

## Dashboard Mode Toggle

- **Always check Test/Live toggle before reading Integration IDs**
  - Dashboard → Developers → Payment Integrations has a Test/Live toggle
  - It's easy to copy a test integration ID while in live mode or vice versa
  - The toggle state is per tab, not global — double-check before copying any credential
