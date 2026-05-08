# Checkout Experiences — Unified Checkout & Pixel

After creating an intention, present one of these to the customer.

---

## Option 1: Unified Checkout (Hosted Redirect)

Redirect the customer to Paymob's hosted page. Fastest integration.

### URL Format
```
https://accept.paymob.com/unifiedcheckout/?publicKey={publicKey}&clientSecret={clientSecret}
```

### Example (Egypt)
```
https://accept.paymob.com/unifiedcheckout/?publicKey=egy_pk_test_XXXX&clientSecret=egy_csk_test_XXXX
```

### Query Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `publicKey` | ✅ | From Dashboard → Settings → Public Key |
| `clientSecret` | ✅ | From Create Intention response |

### Customization (via Dashboard)
Go to **Dashboard → Checkout Customization**:

**Branding:**
- Business logo, colors, font, button style (rounded/rectangular)

**Payment:**
- Tab or list layout for payment methods
- Enable/disable specific payment methods
- Show/hide billing address, item details
- Save card checkbox
- Split Payment (multi-card)
- Terms & conditions link
- Allow/disable payment retrials

**Post Payment:**
- Custom thank you message
- After-payment redirect behavior

> Click "Apply changes" to save. Click "Reset" to restore defaults.

---

## Option 2: Pixel (Embedded Checkout)

Embed Paymob's checkout UI directly inside your page — no redirect.

### Include Script & Styles
```html
<!-- Add to your HTML <head> -->
<script src="https://accept.paymob.com/v1/pay/sdk/pixel.js"></script>
<!-- (also include the required stylesheet from Paymob docs) -->
```

### Mount Point
```html
<div id="paymob-elements"></div>
```

### Initialize Pixel

```javascript
new Pixel({
  publicKey: 'egy_pk_live_XXXX',
  clientSecret: 'egy_csk_live_XXXX',  // from Create Intention response
  paymentMethods: ['card', 'google-pay', 'apple-pay'],
  elementId: 'paymob-elements',

  // Optional behavior flags
  disablePay: false,        // true = use your own Pay button
  showSaveCard: true,       // show "Save card" checkbox to user
  forceSaveCard: true,      // always save without asking

  // Lifecycle hooks
  beforePaymentComplete: async (paymentMethod) => {
    console.log('About to pay with:', paymentMethod);
    return true; // must return true to proceed
  },
  afterPaymentComplete: async (response) => {
    console.log('Payment done:', response);
  },
  onPaymentCancel: () => {
    // Apple Pay only — user cancelled
    console.log('Payment cancelled');
  },
  cardValidationChanged: (isValid) => {
    console.log('Card valid?', isValid);
  },

  // Optional custom styles
  customStyle: {
    Font_Family: 'Gotham',
    Font_Size_Label: '16',
    Font_Size_Input_Fields: '16',
    Font_Size_Payment_Button: '14',
    Font_Weight_Label: 400,
    Font_Weight_Input_Fields: 200,
    Font_Weight_Payment_Button: 600,
    Color_Container: '#FFF',
    Color_Border_Input_Fields: '#D0D5DD',
    Color_Border_Payment_Button: '#A1B8FF',
    Radius_Border: '8',
    Color_Disabled: '#A1B8FF',
    Color_Error: '#CC1142',
    Color_Primary: '#144DFF',
    Color_Input_Fields: '#FFF',
    Text_Color_For_Label: '#000',
    Text_Color_For_Payment_Button: '#FFF',
    Text_Color_For_Input_Fields: '#000',
    Color_For_Text_Placeholder: '#667085',
    Width_of_Container: '100%',
    Vertical_Padding: '40',
    Vertical_Spacing_between_components: '18',
    Container_Padding: '0',
  },
});
```

### Pixel Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `publicKey` | string | ✅ | Your Paymob public key |
| `clientSecret` | string | ✅ | From Create Intention; unique per order, expires in 1 hour |
| `paymentMethods` | string[] | ✅ | `'card'`, `'google-pay'`, `'apple-pay'` |
| `elementId` | string | ✅ | ID of the container div |
| `disablePay` | boolean | ❌ | `true` = hide Paymob's button, use `payFromOutside` event |
| `showSaveCard` | boolean | ❌ | Show save card checkbox |
| `forceSaveCard` | boolean | ❌ | Auto-save without user consent |
| `customStyle` | object | ❌ | Style overrides (see above) |

### Pixel Functions

| Function | Trigger | Description |
|----------|---------|-------------|
| `beforePaymentComplete` | Before Paymob processes | Run custom logic; must return `true` |
| `afterPaymentComplete` | After Paymob processes | Update UI |
| `onPaymentCancel` | Apple Pay cancel | Handle user cancellation |
| `cardValidationChanged` | Card field changes | `isValid: boolean` |
| `updateIntentionData` | After Update Intention API | Sync SDK with new intention data |

### Custom Pay Button (payFromOutside)

If `disablePay: true`, trigger payment from your own button:

```javascript
document.getElementById('myPayBtn').addEventListener('click', () => {
  window.dispatchEvent(new Event('payFromOutside'));
});
```

### updateIntentionData() — Sync Pixel After Updating Intention

When you update the intention (e.g. customer changes their cart), call `updateIntentionData()` after the Update Intention API succeeds to sync the Pixel component with the new data:

```javascript
// 1. Call Update Intention API on your backend
const updated = await fetch('/api/update-intention', {
  method: 'POST',
  body: JSON.stringify({ newItems, newAmount }),
});

// 2. Sync Pixel with updated intention
window.dispatchEvent(new CustomEvent('updateIntentionData', {
  detail: { clientSecret: updated.client_secret }
}));
```

Without calling `updateIntentionData()`, the Pixel will still show the old amount and items — the customer could pay the wrong amount.

### Notes
- If `google-pay` is in `paymentMethods`, you **must** also include the Google Pay SDK.
- Google Pay is **not yet available** in Egypt.
- `client_secret` is unique per order and expires in **1 hour**.
- For intention updates (e.g. changed cart), call `updateIntentionData()` after calling the Update Intention API.

---

## Cross-References

- Create the intention first → `02-intention-api.md`
- Handle the webhook after payment → `04-webhooks-hmac.md`
- Apple Pay setup (domain + certs) required before Apple Pay shows → `15-apple-pay-setup.md`
- Google Pay not available in Egypt → `23-gotchas.md`
- Pixel customization options → `customStyle` object in Pixel section above
