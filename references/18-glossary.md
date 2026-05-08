# Paymob Glossary

## A

**Acquirer** — A bank or processor that acquires card transactions and settles funds to the merchant.

**Accept (Paymob Accept / Checkout)** — Paymob's online checkout supporting cards, wallets, and BNPL. Available via hosted pages, plugins, SDKs, or APIs.

**API Keys (Secret / Public)**
- **Secret Key**: Server-side only. Used to call Paymob APIs from your backend. Rotate on go-live and during security incidents.
- **Public Key**: Safe for client-side initialization (Pixel, Unified Checkout). Cannot authorize payments on its own.

**Apple Pay** — Wallet-based payment method using tokenized credentials from Apple devices.

**Authorization (Auth)** — Real-time approval from the issuer that reserves funds. Can later be captured or voided.

**Auth / Capture (Two-Step Payments)** — Authorize funds first, capture later (full or partial). Useful when fulfillment happens after order placement.

## B

**BNPL (Buy Now Pay Later)** — Installment payment providers (vaLU, Souhoola, Sympl, Tabby, Tamara). Terms vary by region and currency.

**BIN / IIN (Bank Identification Number)** — First 6–8 digits of a card number. Used for routing, issuer identification, and eligibility checks (e.g. for convenience fee rules).

## C

**Callback (Webhook / HMAC-Signed)** — Server-to-server notifications for payment state changes. Must verify HMAC, support retries, and ensure idempotency.

**Capture** — Converts an approved authorization into a settled transaction. Can be full or partial.

**Card on File (CoF)** — Tokenized card reference stored for future charges (CIT or MIT). Never store raw PANs — use Paymob tokens only.

**Chargeback / Dispute** — Issuer or cardholder-initiated reversal after settlement. Different from refunds; involves reason codes and timelines.

**Hosted Checkout** — Paymob-hosted payment page handling payment methods, 3DS, and post-payment redirect.

**CIT (Customer-Initiated Transaction / Cardholder-Initiated Transaction)** — Payment initiated with the customer present. Requires 3DS. Used to save a card for future use.

## D

**Descriptor** — Text shown on the cardholder's bank statement. Must comply with acquirer length and character rules.

## E

**Environments (Test / Live)** — Separate base URLs, API keys, and Integration IDs. Never mix test and live credentials. Same base URL for both environments; environment is determined by credentials.

## G

**Google Pay** — Wallet payment method for Android and web. Requires gateway configuration and regional enablement. Not available in Egypt yet.

## H

**HMAC (Hash-based Message Authentication Code)** — Cryptographic signature using SHA-512 used to verify webhook authenticity and integrity.

## I

**Integration ID** — Unique identifier linking your account to a specific payment method and environment (test or live). Required in `payment_methods` when creating an intention.

**Intention (Payment Intention)** — Defines what the customer intends to pay (amount, currency, method, return URLs). Manages redirects, 3DS, and callbacks. Created via `POST /v1/intention/`.

**Issuer (Issuing Bank)** — The cardholder's bank that approves or declines transactions.

## K

**KYC (Know Your Customer)** — Compliance checks required for onboarding and payment method activation.

**KSA** — Regional marker for Saudi Arabia-specific behavior and requirements.

## L

**Live Mode (Production)** — Real-money environment. Verify descriptors, callbacks, and 3DS before enabling.

## M

**MCC (Merchant Category Code)** — Four-digit code defining the merchant's business type. Impacts risk rules and payment method availability.

**MIT (Merchant-Initiated Transaction)** — Off-session charge (e.g., subscriptions). Often 3DS-exempt after an initial authenticated CIT.

**MPGS (Mastercard Payment Gateway Services)** — Mastercard's modern gateway used by many acquirers.

**MIGS (Mastercard Internet Gateway Service)** — Legacy Mastercard gateway. Test card integration in Paymob = MIGS.

**MOTO (Mail Order / Telephone Order)** — Charge cards remotely using securely stored tokens. Used for hotels (post-checkout charges), service providers, medical clinics.

## N

**Network Token (Scheme Token)** — Token issued by card schemes to replace PANs and improve authorization rates.

## O

**Order vs. Transaction**
- **Order**: Your business reference (cart or invoice).
- **Transaction**: The payment event (auth, capture, or refund).
- Always reconcile both using `merchant_order_id` and `transaction_id`.

## P

**PCI DSS** — Card data security standard. Using Paymob's hosted checkout/tokenization keeps you out of scope.

**Pixel (Native Payment Experience / JS SDK)** — Embedded payment component for web. Keeps secrets server-side. Initialized with `client_secret` and `public_key`.

**Payment Link (Quick Link)** — Hosted URL to collect payments without a website. Good for freelancers, small businesses, events, delivery services.

## R

**Refund** — Merchant-initiated return of captured funds. BNPL and wallets may impose restrictions.

**Reconciliation** — Matching orders, transactions, and payouts. Always log both `merchant_order_id` and `transaction_id`.

## S

**Saved Card Token** — A token representing a card for future charges. Scope and expiry depend on Paymob and the acquirer. Never store raw PANs.

**Settlement / Payout** — Transfer of funds from acquirer to merchant. Timing varies by payment method and currency.

**Split Payments** — Distribute one payment across multiple beneficiaries (Split Amount) or allow a customer to pay with up to 3 cards (Split Payment).

**Subscriptions** — Recurring billing using an initial CIT followed by MITs. Use cases: streaming, gyms, education.

## T

**Tamara / Tabby** — Regional BNPL providers in UAE and KSA with unique onboarding, settlement, and refund rules.

**Tokenization** — Replacing PANs with tokens to reduce PCI scope and enable saved cards.

**Transaction Reference IDs** — `merchant_order_id` (your reference) and Paymob `transaction_id`. Always log both.

## U

**UAE** — Regional marker for United Arab Emirates-specific behavior.

## V

**Void** — Cancels an authorization before capture. Releases funds immediately. Not the same as a refund.

## W

**Wallets (Apple Pay, Google Pay, etc.)** — Tokenized device or browser payments. Require region-specific setup and merchant/domain verification.

**Webhooks** — Payment lifecycle notifications. Always verify HMAC, handle retries idempotently, log all events.

---

## Additional Terms (Added May 2026)

**client_secret** — A short-lived token (expires in 1 hour) returned by Create Intention. Used to initialize Unified Checkout or Pixel on the frontend. Never reuse across different orders. Format: `egy_csk_test_...` or `egy_csk_live_...`

**payment_key** — The JWT inside `payment_keys[0].key` from the intention response. Required as `payment_token` when making MIT charges via `/api/acceptance/pay`. Different from `client_secret`.

**client_secret lifecycle** — Created with intention → passed to frontend → used once by customer to pay → expires in 1 hour. If expired, create a new intention.

**MOTO Integration ID** — A special integration ID for Mail Order / Telephone Order transactions. Required for MIT and subscriptions. Different from regular card integration IDs. Must be requested from account manager.

**Verification Transaction** — A card transaction type that validates card details and checks available balance without reserving or charging funds. Used to verify a card is valid before storing it for future charges. Requires a Verification-type integration ID.

**special_reference** — Your internal order ID passed in Create Intention. Appears as `merchant_order_id` in all webhook callbacks. Use to correlate Paymob transactions to your own orders.

**notification_url** — Backend URL that receives the Transaction Processed Callback (POST with JSON body + HMAC). The authoritative payment result. Card integrations only.

**redirection_url** — Frontend URL where Paymob redirects the customer after payment (GET with query params + HMAC). For UX only — do not update DB here.

**intention_order_id** — Paymob's internal order ID, returned in Create Intention response. Appears as `order.id` in webhook callbacks.

**x-paymob-id** — Paymob's internal request identifier, present in response headers. Include this when contacting support to help them trace the request.

**Test mode vs Live mode** — Determined entirely by which credentials and integration IDs you use. Same base URL for both. Test secret key + test integration IDs = sandbox. Live secret key + live integration IDs = production.

**3DS (3D Secure)** — Authentication protocol used by card networks (Visa Secure, Mastercard Identity Check) to verify the cardholder. Paymob handles 3DS automatically during card checkout. Indicates `is_3d_secure: true` in callback. MIT transactions bypass 3DS.

**Settlement** — Transfer of cleared funds from the acquirer to the merchant's bank account. Timing varies (typically T+1 to T+3 business days). Void is only possible before settlement occurs.

**Chargeback** — Issuer-initiated reversal after settlement, typically triggered by the cardholder disputing a transaction. Different from refund — involves the card network and reason codes. MIT transactions carry higher chargeback risk since 3DS is bypassed.
