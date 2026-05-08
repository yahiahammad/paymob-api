# Payment Methods — Regional Detail Reference

## Cards [All Regions]

### Supported Networks by Region

| Region | Networks |
|--------|----------|
| EGY | Visa, Mastercard, Amex |
| KSA | Visa, Mastercard, Amex, **MADA** |
| UAE | Visa, Mastercard, Amex |
| OMN | Visa, Mastercard, Amex, **Omannet** |

### Card Payment Types

| Type | Description |
|------|-------------|
| **Normal 3DS** | Customer completes 3D Secure (bank OTP) during checkout — standard flow |
| **MOTO** | Card-not-present, no customer interaction — passes OTP and CVV; used for MIT/back-to-back charges |
| **Card On File** | Uses securely stored card from a previous transaction — passes OTP only; used for CIT with saved token |
| **Auth/Cap** | Two-step: authorize (reserve funds) then capture (settle) — used when fulfillment takes time |
| **Verification** | Validates card details and available balance without charging or reserving funds — used to check card validity before future charges |

### Supported Actions
- **Void**: all payment types
- **Refund** (Full/Partial): all payment types
- **Capture** (Full/Partial): Auth/Cap only

### Integration Channels
APIs, Mobile SDKs, Plugins, Payment Links

### Integration IDs
Can have both **test** and **live** IDs.

---

## Mobile Wallets [EGY, KSA]

### Supported Wallets by Region

| Region | Wallets |
|--------|---------|
| EGY | Vodafone Cash, Orange Cash, e& money, We Pay, Bank Wallets |
| KSA | StcPay |

### Supported Actions
- **Refund** (Full/Partial)
- No Void or Capture

### Integration Channels
APIs, Mobile SDKs, Plugins, Payment Links

### Integration IDs
Can have both **test** and **live** IDs.

### Flow
Customer enters wallet phone number → OTP verification → payment confirmed.
Callback: `source_data.type = "wallet"`

---

## BNPLs [EGY, KSA, UAE]

Merchant receives **full payment upfront**. Paymob collects installments from the customer.

### Providers by Region

| Provider | Region | Refund Support | Integration Channels |
|----------|--------|---------------|----------------------|
| vaLU | EGY | Full + Partial | APIs, SDKs, Plugins, Payment Links |
| Souhoola | EGY | Full + Partial | APIs, SDKs, Plugins, Payment Links |
| Contact | EGY | Full + Partial | APIs, Plugins, Payment Links |
| Sympl | EGY | Full + Partial | APIs, Plugins, Payment Links |
| Aman | EGY | Full + Partial | APIs, Plugins, Payment Links |
| Forsa | EGY | Full + Partial | APIs, SDKs, Plugins, Payment Links |
| TRU | EGY | Full + Partial | APIs, Plugins, Payment Links |
| KLIVVR | EGY | Full + Partial | APIs, Plugins, Payment Links |
| MOGO (MID Takseet) | EGY | Full + Partial | APIs, Plugins, Payment Links |
| Halan | EGY | Full + Partial | APIs, Plugins, Payment Links |
| Premium | EGY | Full + Partial | APIs, SDKs, Plugins, Payment Links |
| Seven | EGY | Full only | APIs, Plugins, Payment Links |
| Tabby | KSA, UAE | Full + Partial | APIs, SDKs, Plugins, Payment Links |
| Tamara | KSA, UAE | **Full only** | APIs, SDKs, Plugins, Payment Links |

### BNPL Flow
1. Customer selects BNPL at checkout
2. Redirected to provider's UI for approval and plan selection
3. Merchant receives full amount from Paymob
4. Customer repays provider in installments over time

### Integration IDs
Live IDs only for most BNPL providers. Contact account manager to enable.

---

## Apple Pay [All Regions]

### Regional Availability
EGY, KSA, UAE, OMN

### Supported Actions
- **Refund** (Full/Partial)

### Integration Channels
APIs, Mobile SDKs, Plugins, Payment Links

### Integration IDs
**Live IDs only** — no test Apple Pay integration.

### Setup Requirements by Integration Path

| Integration | Requirements |
|-------------|-------------|
| Mobile SDK | Create Apple Pay certificates (see `15-apple-pay-setup.md`) |
| Pixel (Embedded) / WooCommerce Plugin | Domain verification + Apple Pay certificates |
| Unified Checkout | Configured by Paymob — contact account manager |

### Key Constraints
- **Not supported in WebView** — use native Mobile SDK instead
- `onPaymentCancel` callback needed in Pixel to handle user cancellation
- Customer uses Touch ID / Face ID to authenticate with card stored on Apple device

---

## Google Pay [KSA, UAE, OMN]

### Regional Availability
KSA, UAE, OMN — **not yet available in Egypt**

### Supported Actions
- **Refund** (Full/Partial)

### Integration Channels
APIs, Mobile SDKs, Plugins, Payment Links

### Integration IDs
**Live IDs only**.

### Setup Requirements
- Include Google Pay SDK separately when using Pixel: `import { GooglePay } from 'google-pay'`
- Pass `'google-pay'` in `paymentMethods` array in Pixel
- Not available in Pixel for Egypt — coming soon

---

## Kiosk [EGY]

### Regional Availability
Egypt only

### Supported Actions
- ❌ **No refunds supported**

### Integration Channels
APIs, Plugins, Payment Links (no Mobile SDK support)

### Integration IDs
Can have both **test** and **live** IDs.

### Flow
1. Customer selects Kiosk at checkout
2. Paymob returns a reference number
3. Customer goes to a physical kiosk (Aman, Masary, etc.) and pays cash using the reference
4. Kiosk confirms payment → Paymob sends webhook to your `notification_url`
5. Callback: `source_data.type = "kiosk"` with the reference number

> Payment is async — you must wait for the webhook before confirming the order. Do not confirm on checkout completion alone.

---

## Payment Method Integration ID Types Summary

| Method | Test ID Available | Live ID |
|--------|-----------------|---------|
| Cards | ✅ | ✅ |
| Wallets | ✅ | ✅ |
| Kiosk | ✅ | ✅ |
| Apple Pay | ❌ | ✅ |
| Google Pay | ❌ | ✅ |
| BNPLs | ❌ (most) | ✅ |
| Bank Installments | ❌ | ✅ |
