# Getting Integration Credentials

All credentials live in: **Dashboard → Settings → Account Info**
Integration IDs live in: **Dashboard → Developers → Payment Integrations**

---

## Secret Key

Used for **backend API calls** (Create Intention, Refund, etc.)

**How to get it:**
1. Go to Dashboard → Settings
2. Toggle Live/Test mode to match your integration IDs
3. Click **View** beside Secret Key → copy the value

Auth header format: `Authorization: Token <secret_key>`

> ⚠️ Secret key is environment-specific. Test secret key only works with test integration IDs, and vice versa.

---

## Public Key

Used for **frontend** (Unified Checkout URL, Pixel SDK initialization)

**How to get it:**
1. Go to Dashboard → Settings
2. Toggle Live/Test mode to match your environment
3. Click **View** beside Public Key → copy the value

---

## API Key

Used to **generate auth tokens** for action endpoints (Refund, Void, Capture, QuickLinks, Subscriptions, Transaction Inquiry)

**How to get it:**
1. Go to Dashboard → Settings
2. Click **View** beside API Key → copy the value

> The API key is the **same for both Test and Live** environments.

**Generate auth token:**
```
POST https://accept.paymob.com/api/auth/tokens/
Body: { "api_key": "YOUR_API_KEY" }
Response: { "token": "eyJ..." }  ← valid for 1 hour
```

---

## HMAC Secret

Used to **verify webhook authenticity**. Never expose this.

**How to get it:**
1. Go to Dashboard → Settings
2. Click **View** beside HMAC Secret → copy the value

---

## Integration IDs

One per payment method per environment (test vs. live are separate).

**How to get them:**
1. Go to Dashboard → Developers → Payment Integrations
2. Toggle **Test / Live** mode
3. Each row is one integration — copy the numeric ID

**To add test integrations** (Card/MIGS, Wallet, Kiosk):
- Same tab → click "Add Test Integration"

**To update callback URLs on an integration:**
1. Dashboard → Developers → Payment Integrations
2. Select mode (Test/Live)
3. Click the integration → press **Edit**
4. Fill in the callback URLs → press **Submit**

---

## Credential Quick Reference

| Credential | Where | Env-specific? | Used in |
|------------|-------|---------------|---------|
| Secret Key | Settings | ✅ Yes | Backend API calls (`Authorization: Token`) |
| Public Key | Settings | ✅ Yes | Frontend Pixel / Unified Checkout |
| API Key | Settings | ❌ Same for both | Generating auth tokens |
| HMAC Secret | Settings | ❌ Same for both | Webhook verification |
| Integration ID | Developers → Payment Integrations | ✅ Yes | `payment_methods` in Create Intention |
