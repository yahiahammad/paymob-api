# Paymob Overview — Environments, Credentials, Integration Paths

## Base URLs by Region

All code examples use `https://accept.paymob.com` (Egypt). Replace with the region URL below for other markets.

| Region | URL |
|--------|-----|
| Egypt  | `https://accept.paymob.com/` |
| Oman   | `https://oman.paymob.com/` |
| KSA    | `https://ksa.paymob.com/` |
| UAE    | `https://uae.paymob.com/` |

Same URL for both sandbox and production — environment is determined by the **API keys and Integration IDs** used (test vs. live).

## Credentials You Need

| Credential | Purpose |
|------------|---------|
| Secret Key | Backend API calls (`Authorization: Token <secret_key>`) |
| Public Key | Frontend (Pixel/Unified Checkout `?publicKey=`) |
| HMAC Secret | Verifying webhook authenticity |
| Integration ID(s) | One per payment method (card, wallet, etc.); test vs. live separate |

Get all from: **Paymob Dashboard → Settings → Account Info**

Integration IDs from: **Dashboard → Developers → Payment Integrations**

## Integration Paths

| Path | Best For | Complexity |
|------|----------|-----------|
| Payment Links | No-code, invoicing | None |
| E-commerce Plugins | Shopify, WooCommerce, Magento, OpenCart, PrestaShop, Odoo | Low |
| Hosted Checkout (Unified) | Redirect-based web checkout | Medium |
| Pixel (Embedded) | Inline checkout in your own page | High |
| Mobile SDKs | Native iOS/Android/Flutter/RN apps | High |

## Payment Methods Available

- **Cards**: Visa, Mastercard, Amex, MADA (KSA), OmanNet — with 3DS
- **Mobile Wallets**: Vodafone Cash, Orange Cash, e& money, We Pay (Egypt); StcPay (KSA)
- **Quick Payments**: Apple Pay, Google Pay
- **BNPL**: vaLU, Sympl, Souhoola, Halan, TRU, MOGO (Egypt); Tabby, Tamara (UAE/KSA)
- **Kiosk**: Offline reference-based payments
- **In-Person**: Tap to Pay via Paymob App

Not all methods are enabled by default — contact your account manager.

## Dashboard Sections

- **Settings**: Branding, contact info, credentials (secret/public/API/HMAC keys), notifications
- **Checkout Customization**: Brand colors, fonts, button styles, payment method order, save card, split payment, terms & conditions
- **Payment Integrations**: View/edit integration IDs, set callback URLs (test vs. live mode toggle)
- **Transactions**: Filter by date/ID/status, view details, perform Refund/Void/Capture
- **Orders**: Filter by date/Order ID/Merchant Order ID
