# Test Credentials

Use these credentials while in **sandbox/test mode**. Test and live environments share the same base URL — the mode is determined by which keys and integration IDs you use.

---

## Test Cards

### Mastercard — Card 1

| Field | Value |
|-------|-------|
| Card Number | `5123456789012346` |
| Cardholder Name | `Test Account` |
| Expiry Month | `01` |
| Expiry Year | `39` |
| CVV | `123` |

### Mastercard — Card 2

| Field | Value |
|-------|-------|
| Card Number | `5123450000000008` |
| Cardholder Name | `Test Account` |
| Expiry Month | `01` |
| Expiry Year | `39` |
| CVV | `123` |

### Visa

| Field | Value |
|-------|-------|
| Card Number | `4111111111111111` |
| Cardholder Name | `Test Account` |
| Expiry Month | `01` |
| Expiry Year | `39` |
| CVV | `123` |

---

## Test Mobile Wallet

| Field | Value |
|-------|-------|
| Wallet Number | `01010101010` |
| MPin Code | `123456` |
| OTP | `123456` |

---

## Getting Test Integration IDs

From **Dashboard → Payment Integrations** (switch to Test mode):
- Test Card Integration = MIGS test integration
- Test Wallet Integration
- Test Kiosk Integration

If you don't have test integrations, you can add them from the same tab.

---

## Tips

- Always set `notification_url` and `redirection_url` to your local/dev endpoints while testing
- Use [ngrok](https://ngrok.com) or similar to expose localhost for webhook testing
- Check transactions in Dashboard → Transactions (Test mode) to debug
- 3DS test: on test card flows, you may be prompted with a test 3DS page — proceed normally
