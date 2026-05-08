---
name: paymob-api
description: >
  Use when integrating with the Paymob payment API. Covers intention API,
  client_secret, checkout experiences, HMAC webhook verification, refunds,
  voids, captures, saved cards, CIT/MIT, subscriptions, QuickLinks,
  transaction inquiry, Apple Pay, Google Pay, mobile SDKs, split payments,
  regional payment methods (MADA, wallets, BNPL), credentials setup,
  error codes, gotchas, and go-live checklist. Supports Egypt, Oman, KSA, UAE.
license: MIT
metadata:
  author: yahia
---

# Paymob API Skill

Last fetched: May 2026 | Support: support@paymob.com
Postman collection: https://github.com/PaymobAccept/API-Postman-Collections

---

## Decision Tree

```
What are you trying to do?
|
+-- "First time / getting started"
|   -> READ: 00-quick-start.md
|
+-- "I need to accept a payment"
|   +-- Create the payment (backend) -> READ: 02-intention-api.md
|   +-- Show checkout to customer -> READ: 03-checkout-experiences.md
|   +-- Mobile app -> READ: 09-mobile-sdks.md
|
+-- "Webhooks / callbacks"
|   +-- Set up handler + understand callback types -> READ: 19-auth-token-and-callbacks.md
|   +-- HMAC verification (Node.js + Python) -> READ: 04-webhooks-hmac.md
|   +-- 3DS flow explanation -> READ: 19-auth-token-and-callbacks.md
|
+-- "Action on existing transaction"
|   +-- Refund / Void / Capture (rules + code) -> READ: 05-payment-actions.md
|   +-- Look up transaction -> READ: 13-transaction-inquiry-apis.md
|   +-- Payment link -> READ: 12-quicklink-apis.md
|
+-- "Saved cards / recurring payments"
|   +-- Save card / CIT / MIT (full endpoint) -> READ: 06-saved-cards.md
|   +-- Subscriptions (all actions) -> READ: 07-subscriptions.md
|   +-- Business-level understanding -> READ: 22-core-features-conceptual.md
|
+-- "Payment method setup"
|   +-- Regional availability + BNPL providers -> READ: 20-payment-methods-regional.md
|   +-- Apple Pay domain + certificates -> READ: 15-apple-pay-setup.md
|   +-- Split payment / split amount -> READ: 10-split-features.md
|
+-- "Debugging / errors"
|   +-- Specific error message -> READ: 21-error-codes.md
|   +-- Weird behavior / non-obvious rules -> READ: 23-gotchas.md
|   +-- Common issues / FAQ -> READ: 16-common-issues-and-checklist.md
|
+-- "Credentials / environment"
|   +-- Where to find keys and IDs -> READ: 14-getting-credentials.md
|   +-- Test cards and sandbox -> READ: 11-test-credentials.md
|   +-- Going live checklist -> READ: 16-common-issues-and-checklist.md
|
+-- "Term I don't recognise"
    -> READ: 18-glossary.md
```

---

## Table of Contents (references/)

### Getting Started
- `00-quick-start.md` — Zero to first payment with full code (Node.js)

### Developer Reference (API / SDK)
- `01-overview.md` — Base URLs, environments, credentials, integration paths
- `02-intention-api.md` — Create/Update Intention; full request, full response schema, payment_keys, Python + Node.js
- `03-checkout-experiences.md` — Unified Checkout + Pixel (all props, customStyle, updateIntentionData)
- `04-webhooks-hmac.md` — HMAC (transaction POST vs GET field names, card token, subscription); Node.js + Python
- `05-payment-actions.md` — Refund, Void, Capture; rules, per-method support, dashboard steps; Node.js + Python
- `06-saved-cards.md` — Tokenization, CIT, full MIT 2-step endpoint; Node.js
- `07-subscriptions.md` — Plans (full fields), Create Subscription, all 12 subscription action endpoints
- `09-mobile-sdks.md` — iOS, Android, Flutter, React Native SDKs
- `10-split-features.md` — Split Payment and Split Amount with callback structure
- `11-test-credentials.md` — Test cards, wallets, sandbox tips
- `12-quicklink-apis.md` — Create/Cancel QuickLink; Node.js
- `13-transaction-inquiry-apis.md` — By Transaction ID, By Order ID; Node.js + Python
- `19-auth-token-and-callbacks.md` — Auth Token (Node.js + Python + caching), callback types, full payload, 3DS flow
- `20-payment-methods-regional.md` — All methods: cards (5 types), wallets, BNPL (14 providers), Apple/Google Pay, Kiosk

### Documentation / Support
- `14-getting-credentials.md` — Every credential: where to find it, environment specificity
- `15-apple-pay-setup.md` — Domain verification + both certificate types (full OpenSSL commands)
- `16-common-issues-and-checklist.md` — Go-live checklist, 5 common errors + fixes, FAQ answers
- `17-convenience-fee-and-bank-installments.md` — Convenience fee config + Bank Installments [EGY]
- `18-glossary.md` — Full A-W glossary + May 2026 additions (client_secret, payment_key, 3DS, etc.)
- `21-error-codes.md` — All HTTP error shapes, fixes, token caching, debug tips
- `22-core-features-conceptual.md` — Auth/Cap, Subscriptions, Saved Cards, Split Features, Reports (business-level)
- `23-gotchas.md` — Non-obvious rules: amounts in cents, integration ID mismatch, webhook-only DB updates, idempotency, rate limits, void card-only, Apple Pay no WebView, Google Pay not in Egypt, and more

> `08-payment-methods.md` is deprecated — see `20-payment-methods-regional.md`

---

## Core Flow (Always True)

```
1. Backend: POST /v1/intention/  ->  get client_secret
   Header: Authorization: Token <secret_key>

2. Frontend: redirect to Unified Checkout OR embed Pixel with client_secret

3. Customer completes payment (3DS handled by Paymob automatically)

4. Paymob POST -> notification_url (webhook) -> VERIFY HMAC -> update DB

5. Paymob redirect -> redirection_url -> show success/fail UI to customer
```

Webhook is truth. Redirect is UX only. Never update DB on redirect.

---

## Key Constants

| Region | Base URL |
|--------|----------|
| Egypt  | https://accept.paymob.com/ |
| Oman   | https://oman.paymob.com/ |
| KSA    | https://ksa.paymob.com/ |
| UAE    | https://uae.paymob.com/ |

| Auth Type | Header | Used For |
|-----------|--------|---------|
| Secret Key | Authorization: Token <secret_key> | Intention API only |
| Bearer Token | Authorization: Bearer <token> | Refund, Void, Capture, QuickLinks, Subscriptions, Inquiry |

| Credential | Env-specific? | Get From |
|------------|--------------|---------|
| Secret Key | Yes (test vs live) | Dashboard -> Settings |
| Public Key | Yes (test vs live) | Dashboard -> Settings |
| API Key | No - same for both | Dashboard -> Settings |
| HMAC Secret | No - same for both | Dashboard -> Settings |
| Integration IDs | Yes (test vs live) | Dashboard -> Developers -> Payment Integrations |

## Top 5 Gotchas (see 23-gotchas.md for full list)

1. Amounts are in CENTS not pounds (EGP 50 = 5000)
2. Test integration ID + live secret key (or vice versa) -> 404
3. notification_url only fires for CARD integrations, not wallets/kiosk
4. POST callback uses obj.id / obj.order.id; GET redirect uses id / order_id (different field names for HMAC)
5. Never update DB on redirect — only on webhook after HMAC verification
