# Mobile SDKs

All SDKs follow the same flow:
1. **Backend**: Create Intention → get `client_secret`
2. **App**: Pass `client_secret` + `public_key` to SDK
3. **SDK**: Presents native checkout UI
4. **Result**: SDK callback (UI update) + backend webhook (source of truth)

> **Always verify payment via backend webhook, not SDK callback alone.**

Set response callback URL in integration ID to:
`https://accept.paymob.com/api/acceptance/post_pay`

---

## iOS SDK

**Supported**: Cards, Wallets, Apple Pay, vaLU, Souhoola, Forsa, Premium6

### Installation — CocoaPods (Recommended)

```ruby
# Podfile
pod 'Paymob'
```

```bash
pod install
# Open .xcworkspace (not .xcodeproj)
```

In Xcode: General → Frameworks → Change `PaymobSDK.xcframework` from **Do Not Embed** to **Embed & Sign**

### Usage

```swift
import PaymobSDK

class ViewController: UIViewController, PaymobSDKDelegate {

  func startPayment() {
    let paymob = PaymobSDK()
    paymob.delegate = self

    // Customization (optional)
    paymob.paymobSDKCustomization.appName = "MyApp"
    paymob.paymobSDKCustomization.buttonBackgroundColor = .black
    paymob.paymobSDKCustomization.buttonTextColor = .white
    paymob.paymobSDKCustomization.showSaveCard = true
    paymob.paymobSDKCustomization.saveCardDefault = false

    let clientSecret = "egy_csk_live_XXXX"  // from Create Intention
    let publicKey = "egy_pk_live_XXXX"

    do {
      try paymob.presentPayVC(VC: self, PublicKey: publicKey, ClientSecret: clientSecret)
    } catch {
      print(error)
    }
  }

  // Delegate callbacks
  func transactionAccepted(transactionDetails: [String: Any]) {
    print("Success:", transactionDetails)
    // Update UI only — confirm via backend webhook
  }
  func transactionRejected() { print("Rejected") }
  func transactionPending()  { print("Pending") }
}
```

---

## Android SDK

**Supported**: Cards, Wallets, vaLU, Souhoola, Forsa, Premium6

### Installation

1. Download SDK `.aar` from Paymob portal
2. Place in `app/libs/`
3. `settings.gradle.kts`:
```kotlin
repositories {
  maven { url = rootProject.projectDir.toURI().resolve("libs") }
  maven { url = uri("https://jitpack.io") }
}
```
4. `app/build.gradle.kts`:
```kotlin
dependencies {
  implementation("com.paymob.sdk:Paymob-SDK:{{version}}")
}
android {
  buildFeatures { dataBinding = true }
}
```

### Usage

```kotlin
import com.paymob.paymob_sdk.PaymobSdk
import com.paymob.paymob_sdk.ui.PaymobSdkListener

class MainActivity : AppCompatActivity(), PaymobSdkListener {

  fun startPayment() {
    val sdk = PaymobSdk.Builder(
      context = this,
      clientSecret = "CLIENT_SECRET",
      publicKey = "PUBLIC_KEY",
      paymobSdkListener = this
    )
      .setButtonBackgroundColor(Color.BLACK)
      .setButtonTextColor(Color.WHITE)
      .showSaveCard(true)
      .saveCardByDefault(false)
      .build()

    sdk.start()
  }

  override fun onSuccess()  { /* Update UI */ }
  override fun onFailure()  { /* Update UI */ }
  override fun onPending()  { /* Update UI */ }
}
```

---

## Flutter SDK

**Supported**: Cards, Wallets, Apple Pay (iOS), vaLU, Souhoola, Forsa, Premium6

Min Android SDK: 23, Compile SDK: 33

### Dart Usage

```dart
import 'package:flutter/services.dart';

static const methodChannel = MethodChannel('paymob_sdk_flutter');

Future _payWithPaymob(String pk, String csk, {
  String? appName,
  Color? buttonBackgroundColor,
  Color? buttonTextColor,
  bool? saveCardDefault,
  bool? showSaveCard,
}) async {
  try {
    final String result = await methodChannel.invokeMethod('payWithPaymob', {
      "publicKey": pk,
      "clientSecret": csk,
      "appName": appName,
      "buttonBackgroundColor": buttonBackgroundColor?.value,
      "buttonTextColor": buttonTextColor?.value,
      "saveCardDefault": saveCardDefault,
      "showSaveCard": showSaveCard,
    });

    switch (result) {
      case 'Successfull': /* Update UI */ break;
      case 'Rejected':   /* Update UI */ break;
      case 'Pending':    /* Update UI */ break;
    }
  } on PlatformException catch (e) {
    print("SDK error: ${e.message}");
  }
}
```

iOS native bridge needed in AppDelegate — see full Flutter SDK docs.

Android native bridge needed in MainActivity — see full Flutter SDK docs.

---

## React Native SDK

**Supported**: Cards, Wallets, Apple Pay (iOS)

### Installation

```bash
yarn add paymob-reactnative@https://github.com/PaymobAccept/paymob-reactnative-sdk.git
cd ios && pod install && cd ..
```

Android `build.gradle`:
```groovy
allprojects {
  repositories {
    maven { url = rootProject.projectDir.toURI().resolve("../node_modules/paymob-reactnative/android/libs") }
    maven { url = uri("https://jitpack.io") }
  }
}
```

### Usage

```javascript
import Paymob, { PaymentResult } from 'paymob-reactnative';

// Customize before presenting
Paymob.setAppName('MyApp');
Paymob.setButtonBackgroundColor('#000000');
Paymob.setButtonTextColor('#FFFFFF');
Paymob.setShowSaveCard(true);
Paymob.setSaveCardDefault(false);

// Listen for results
Paymob.setSdkListener((status) => {
  switch (status) {
    case PaymentResult.SUCCESS:  /* Update UI */ break;
    case PaymentResult.FAIL:     /* Update UI */ break;
    case PaymentResult.PENDING:  /* Update UI */ break;
  }
});

// Invoke SDK (savedBankCards is optional)
const savedBankCards = [
  {
    maskedPan: '2346',
    savedCardToken: 'tok_abc123',
    creditCard: CreditCardType.MASTERCARD,
  },
];
Paymob.presentPayVC('CLIENT_SECRET', 'PUBLIC_KEY', savedBankCards);
```

### iOS note
If you hit bridging issues, create a new blank Swift file in Xcode and select "Create Bridging Header" when prompted.

---

## SDK Customization Options (All SDKs)

| Option | Description | Default |
|--------|-------------|---------|
| `appName` / `appIcon` | Header branding | — |
| `buttonBackgroundColor` | Pay button color | Black |
| `buttonTextColor` | Pay button text | White |
| `showSaveCard` | Show save card checkbox | true |
| `saveCardDefault` | Save card pre-checked | false |
