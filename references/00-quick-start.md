# Quick Start — Zero to First Payment

**Goal**: Accept your first card payment in the shortest path possible.
**Time**: ~30 minutes assuming your Node.js backend is already running.

---

## What You Need First

From **Dashboard → Settings → Account Info**:
- `SECRET_KEY` — starts with `egy_sk_test_...`
- `PUBLIC_KEY` — starts with `egy_pk_test_...`

From **Dashboard → Developers → Payment Integrations** (Test mode):
- `INTEGRATION_ID` — a number like `123456` (add a test MIGS/Card integration if none exists)

From **Dashboard → Settings**:
- `HMAC_SECRET` — for webhook verification

---

## Step 1 — Create an Intention (Backend)

```javascript
// POST https://accept.paymob.com/v1/intention/
const response = await axios.post(
  'https://accept.paymob.com/v1/intention/',
  {
    amount: 5000,           // EGP 50.00 in cents
    currency: 'EGP',
    payment_methods: [parseInt(process.env.INTEGRATION_ID)],
    items: [{ name: 'Test Item', amount: 5000, quantity: 1 }],
    billing_data: {
      first_name: 'Test',
      last_name: 'User',
      email: 'test@example.com',
      phone_number: '01012345678',
    },
    notification_url: 'https://YOUR-DOMAIN/webhook/paymob',
    redirection_url: 'https://YOUR-DOMAIN/payment/result',
  },
  { headers: { Authorization: `Token ${process.env.SECRET_KEY}` } }
);

const { client_secret } = response.data;
// Store client_secret — send it to frontend
```

---

## Step 2 — Redirect to Checkout (Frontend)

```javascript
// Simplest possible: redirect to Unified Checkout
const checkoutUrl =
  `https://accept.paymob.com/unifiedcheckout/` +
  `?publicKey=${process.env.PUBLIC_KEY}` +
  `&clientSecret=${clientSecret}`;

window.location.href = checkoutUrl;
```

Customer sees Paymob's hosted payment page, enters card details, completes 3DS OTP.

---

## Step 3 — Handle the Webhook (Backend)

```javascript
const crypto = require('crypto');

app.post('/webhook/paymob', express.json(), (req, res) => {
  // 1. Verify HMAC
  const received = req.query.hmac;
  const obj = req.body.obj;

  const fields = [
    obj.amount_cents, obj.created_at, obj.currency, obj.error_occured,
    obj.has_parent_transaction, obj.id, obj.integration_id, obj.is_3d_secure,
    obj.is_auth, obj.is_capture, obj.is_refunded, obj.is_standalone_payment,
    obj.is_voided, obj.order?.id, obj.owner, obj.pending,
    obj.source_data?.pan, obj.source_data?.sub_type, obj.source_data?.type, obj.success,
  ].map(String).join('');

  const computed = crypto
    .createHmac('sha512', process.env.HMAC_SECRET)
    .update(fields).digest('hex');

  if (computed !== received) return res.status(401).send('Invalid HMAC');

  // 2. Check payment result
  if (obj.success && !obj.pending) {
    const orderId = obj.order?.merchant_order_id; // your special_reference
    // ✅ Update order to PAID in your DB
  }

  res.status(200).send('OK');
});
```

---

## Step 4 — Handle the Redirect (Frontend)

```javascript
// Customer lands on your redirection_url with query params
const params = new URLSearchParams(window.location.search);
const success = params.get('success') === 'true';
const pending = params.get('pending') === 'true';

if (success && !pending) {
  showMessage('Payment successful! ✅');
} else if (pending) {
  showMessage('Payment pending — we\'ll confirm shortly.');
} else {
  showMessage('Payment failed. Please try again.');
}
// ⚠️ Don't update DB here — only use the webhook (Step 3) for that
```

---

## Test Cards

| Card | Number | Expiry | CVV |
|------|--------|--------|-----|
| Mastercard | `5123456789012346` | `01/39` | `123` |
| Visa | `4111111111111111` | `01/39` | `123` |

---

## Common First-Time Mistakes

| Mistake | Fix |
|---------|-----|
| 404 on Create Intention | Integration ID mode (test/live) doesn't match secret key mode |
| HMAC mismatch | Field order is wrong — follow exactly: `amount_cents, created_at, currency...` |
| Webhook not received | `notification_url` must be a public HTTPS URL; use ngrok for local dev |
| Order not updating | You're updating DB on redirect, not webhook — redirect is UX only |
| `client_secret` expired | They expire in 1 hour — create a new intention if expired |

---

## Next Steps

- Add more payment methods → add their integration IDs to `payment_methods` array
- Save cards for returning users → `06-saved-cards.md`
- Embed checkout in your page → `03-checkout-experiences.md` (Pixel section)
- Go live → `16-common-issues-and-checklist.md` (go-live checklist)
