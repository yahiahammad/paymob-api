# Convenience Fee & Bank Installments

---

## Convenience Fee

A convenience fee is an **additional charge on top of the base transaction amount**, used by merchants to offset payment processing costs.

**Contact your Paymob account manager to enable and configure convenience fees on your account.**

### Configuration Options

#### General

| Option | Description |
|--------|-------------|
| **Fee Type** | Percentage (%), Fixed amount, or a combination of both |
| **Maximum Fee** | Optional cap on the fee amount |
| **Refundable** | Whether the fee is returned on refund |

#### Card Payments — Fee Variations

| Option | Description |
|--------|-------------|
| Uniform fee | Same fee for all card types |
| Debit vs. Credit | Different fees for debit and credit cards |
| BIN-based | Different fees for specific card number ranges |
| Domestic vs. International | Different fees for local vs. cross-border transactions |

#### Wallet Payments

Currently supports **uniform fees only** — one fee applies to all wallet types.

### How It Works

- Fee is calculated and shown to the customer **at checkout** before they confirm payment
- Fee appears as a separate line item on the checkout page
- If refundable is enabled, the fee is returned alongside the principal on refund
- Fee configuration is set per integration ID by Paymob

---

## Bank Installments [EGY only]

Bank installments allow customers to split purchases into **monthly payments** through partner banks, increasing affordability for higher-value transactions.

### Key Facts

| | |
|--|--|
| **Region** | Egypt only |
| **Integration channels** | APIs, Plugins, Payment Links |
| **Integration IDs** | Live IDs only (no test integration IDs available) |
| **Supported actions** | Refund (full and partial) |

### How It Works

1. Customer selects "Bank Installments" at checkout
2. Paymob presents available installment plans from partner banks
3. Customer selects their preferred plan (e.g., 6 months, 12 months)
4. Bank approves and the transaction is processed
5. Customer pays the bank monthly; merchant receives full amount upfront (minus fees)

### Integration Notes

- Use a **Bank Installments integration ID** in `payment_methods` when creating the intention
- Since only **live IDs** exist, you cannot test this method in sandbox mode
- Available via Unified Checkout and Pixel (embedded)
- Not available via mobile SDKs directly — redirect to web checkout if needed
- Contact your account manager to get a Bank Installments integration ID enabled on your account
