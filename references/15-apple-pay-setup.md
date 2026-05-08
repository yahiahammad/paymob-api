# Apple Pay Setup — Domain Verification & Certificates

Two separate setup tasks required before Apple Pay goes live:
1. **Domain Verification** — proves you own the domain serving Apple Pay
2. **Certificates Creation** — cryptographic certificates sent to Paymob

---

## Part 1: Apple Pay Domain Verification

Apple requires you to host a verification file on your domain before Apple Pay can be used.

### Steps

1. **Get the verification file** from Paymob support or your account manager (a `.txt` or `.well-known` file specific to your Merchant ID)

2. **Host the file** at this exact path on your domain:
   ```
   https://yourdomain.com/.well-known/apple-developer-merchantid-domain-association
   ```
   - Must be accessible over **HTTPS**
   - No redirect — must return the file directly at that URL
   - No authentication required on that path

3. **Register the domain** in the Apple Developer Portal under your Merchant ID → "Merchant Domains"

4. **Verify** — Apple checks the file exists and is valid

5. **Notify Paymob** once domain is verified so they can complete configuration on their end

> ⚠️ Apple Pay is **not supported in WebViews** — use the native mobile SDK for in-app Apple Pay.

---

## Part 2: Apple Pay Certificates Creation

Two certificates must be created and sent to Paymob:

| Certificate | Purpose |
|-------------|---------|
| **Merchant Identity Certificate** | Verifies your business identity |
| **Payment Processing Certificate** | Encrypts payment data (EC key) |

### Prerequisites

Verify OpenSSL is installed:
```bash
openssl version
```

Install if missing:
- **macOS**: `brew install openssl`
- **Ubuntu/Debian**: `sudo apt update && sudo apt install openssl`
- **Windows**: Download from https://slproweb.com/products/Win32OpenSSL.html

---

### Certificate 1: Merchant Identity Certificate

**Step 1 — Generate RSA private key:**
```bash
openssl genpkey -algorithm RSA -out merchant_identity.key
```

**Step 2 — Generate CSR:**
```bash
openssl req -new -key merchant_identity.key -out merchant_identity.csr
```

**Step 3 — Upload CSR to Apple:**
1. Apple Developer Portal → Your Merchant ID
2. Scroll to **"Apple Pay Merchant Identity Certificate"**
3. Click **Create Certificate** → upload `merchant_identity.csr`
4. Download the generated `merchant_id.cer`

**Step 4 — Convert to PEM:**
```bash
openssl x509 -inform DER -in merchant_id.cer -out merchant_certificate.pem
```

**You now have:**
- `merchant_identity.key` (private key)
- `merchant_id.cer` (certificate from Apple)
- `merchant_certificate.pem` (converted certificate)

---

### Certificate 2: Payment Processing Certificate

**Step 1 — Generate EC private key:**
```bash
openssl ecparam -genkey -name prime256v1 -out payment_processing.key
```

**Step 2 — Generate CSR:**
```bash
openssl req -new -key payment_processing.key -out payment_processing.csr
```
Critical fields: Country Name (2 letters), Common Name (your exact domain). Skip optional fields with Enter.

**Step 3 — Upload CSR to Apple:**
1. Apple Developer Portal → Your Merchant ID
2. Scroll to **"Apple Pay Payment Processing Certificate"**
3. Click **Create Certificate** → upload `payment_processing.csr`
4. Download the generated `apple_pay.cer`

**Step 4 — Convert to PEM:**
```bash
openssl x509 -inform DER -in apple_pay.cer -out payment_certificate.pem
```

**You now have:**
- `payment_processing.key` (private key)
- `apple_pay.cer` (certificate from Apple)
- `payment_certificate.pem` (converted certificate)

---

### Send to Paymob

You'll have **6 files total**:

| File | Type |
|------|------|
| `merchant_identity.key` | Merchant private key |
| `merchant_id.cer` | Merchant certificate |
| `merchant_certificate.pem` | Merchant certificate (PEM) |
| `payment_processing.key` | Payment private key |
| `apple_pay.cer` | Payment certificate |
| `payment_certificate.pem` | Payment certificate (PEM) |

**Secure transfer:**
1. Compress all 6 files into a **password-protected ZIP**
2. Email the ZIP to **support@paymob.com**
3. Send the ZIP password via a **separate channel** (SMS or WhatsApp)
