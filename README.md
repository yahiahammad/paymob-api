# paymob-api

A Claude Code skill for integrating with the [Paymob](https://paymob.com) payment gateway across Egypt, Oman, KSA, and UAE.

## What this skill does

When active, this skill gives Claude deep, up-to-date knowledge of the Paymob API — far beyond what exists in general training data. It covers the full payment lifecycle from creating an intention to verifying webhooks, and includes real, runnable code examples in Node.js and Python.

## When Claude uses it

Claude automatically loads this skill when you mention anything related to:

- Paymob integration, dashboard, or API
- Accepting payments in Egypt, Oman, KSA, or UAE
- `client_secret`, `intention API`, `payment_key`
- HMAC webhook verification
- Unified Checkout or Pixel checkout embed
- Refunds, voids, captures on Paymob
- Saved cards, CIT/MIT, or subscriptions
- QuickLinks, MADA, mobile SDKs

## Coverage

| Area | What's covered |
|------|---------------|
| **Intention API** | Full request/response schema, `payment_keys`, multi-region base URLs |
| **Checkout** | Unified Checkout iframe + Pixel embed, all props, `customStyle`, `updateIntentionData` |
| **Webhooks** | HMAC verification (Node.js + Python), POST vs GET field name differences, 3DS callback flow |
| **Payment Actions** | Refund, void, capture — rules per method, dashboard steps, code samples |
| **Saved Cards** | Tokenization, CIT flow, full MIT 2-step endpoint |
| **Subscriptions** | Plan creation, all 12 subscription action endpoints |
| **Mobile SDKs** | iOS, Android, Flutter, React Native |
| **Regional Methods** | Cards, wallets, BNPL (14 providers), Apple Pay, Google Pay, Kiosk — by country |
| **Apple Pay** | Domain verification + both certificate types (full OpenSSL commands) |
| **Split Payments** | Split Payment and Split Amount with callback structure |
| **Credentials** | Where to find every key/ID, test vs live environment differences |
| **Error Codes** | All HTTP error shapes, fixes, debug tips |
| **Gotchas** | Amounts in cents, integration ID mismatch, webhook-only DB updates, rate limits, and more |
| **Test Cards** | Full sandbox test cards, wallets, and tips |

## Quick example: what you can ask

```
"How do I create a Paymob intention and show Unified Checkout in React?"
"Write HMAC verification middleware for Paymob webhooks in Express"
"How do I issue a refund via the Paymob API?"
"Set up saved cards with MIT for a subscription flow"
"What test cards does Paymob sandbox support?"
"Why am I getting a 404 on the intention API?"
"How do I verify Apple Pay domain for Paymob in Egypt?"
"What's the difference between client_secret and payment_key?"
```

## Core flow (always true)

```
1. Backend  POST /v1/intention/  →  get client_secret
2. Frontend  show Unified Checkout or Pixel with client_secret
3. Customer completes payment (3DS handled by Paymob)
4. Paymob POST → notification_url (webhook) → VERIFY HMAC → update DB
5. Paymob redirect → redirection_url → show success/fail UI
```

> Webhook is truth. Redirect is UX only. Never update your database on redirect.

## Regions supported

| Region | Base URL |
|--------|----------|
| Egypt  | https://accept.paymob.com/ |
| Oman   | https://oman.paymob.com/ |
| KSA    | https://ksa.paymob.com/ |
| UAE    | https://uae.paymob.com/ |

## Resources

- Postman collection: https://github.com/PaymobAccept/API-Postman-Collections
- Support: support@paymob.com
