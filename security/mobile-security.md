# Mobile Security (iOS and Android)

A mobile app runs on a device the user controls: the binary can be decompiled, the traffic can be intercepted with a proxy, and on a rooted/jailbroken device even the app's memory can be inspected. The rule that follows is the same as for a browser: **the client is never trusted** — anything that matters (prices, permissions, validation) is decided on the backend, and nothing secret ships inside the app.

| Area | iOS | Android |
|---|---|---|
| Permissions | `Info.plist` usage strings + runtime prompt | `AndroidManifest.xml` + runtime prompt |
| Secure storage | Keychain | Android Keystore |
| Enforced HTTPS | ATS (App Transport Security) | Network Security Config |
| Certificate pinning | `NSPinnedDomains` (iOS 14+), TrustKit | `<pin-set>` in Network Security Config, OkHttp `CertificatePinner` |
| Device integrity | App Attest / DeviceCheck | Play Integrity API |
| Biometrics | `LocalAuthentication` (Face ID / Touch ID) | `BiometricPrompt` |
| Code protection | Compiled binary (still dumpable) | R8 / ProGuard obfuscation |
| Verified deep links | Universal Links | App Links |

## Sandbox and permissions

Each app runs isolated: it can't read another app's files or memory. Access to anything sensitive (camera, location, contacts) needs a permission, which the user grants at runtime. Ask only for what the feature actually needs, and as late as possible — an app that requests everything at launch gets denied, and is a bigger target if compromised.

```xml
<!-- iOS: Info.plist — the text shown in the permission prompt is mandatory -->
<key>NSCameraUsageDescription</key>
<string>The camera is used to scan the payment QR code.</string>
```

```xml
<!-- Android: AndroidManifest.xml -->
<uses-permission android:name="android.permission.CAMERA" />
```

## Secure storage

Tokens, passwords, and keys never go in `UserDefaults`, `SharedPreferences`, `AsyncStorage`, or plain files: they are unencrypted files inside the app's folder, readable on a rooted device or from a device backup.

- **iOS Keychain**: encrypted store managed by the OS, survives app reinstalls and can be restricted to the unlocked device.
- **Android Keystore**: holds encryption keys in secure hardware (TEE or StrongBox) — the key material never leaves it. The app encrypts its data with that key and stores only the ciphertext.

```swift
// iOS: save a token in the Keychain
let query: [String: Any] = [
    kSecClass as String: kSecClassGenericPassword,
    kSecAttrAccount as String: "access_token",
    kSecValueData as String: Data(token.utf8),
    kSecAttrAccessible as String: kSecAttrAccessibleWhenUnlockedThisDeviceOnly,  // not in backups, only while unlocked
]
SecItemDelete(query as CFDictionary)  // replace any previous value
SecItemAdd(query as CFDictionary, nil)
```

```kotlin
// Android: key generated inside the Keystore, used to encrypt the token
val keyGenerator = KeyGenerator.getInstance(KeyProperties.KEY_ALGORITHM_AES, "AndroidKeyStore")
keyGenerator.init(
    KeyGenParameterSpec.Builder("token_key", KeyProperties.PURPOSE_ENCRYPT or KeyProperties.PURPOSE_DECRYPT)
        .setBlockModes(KeyProperties.BLOCK_MODE_GCM)
        .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_NONE)
        .build()
)
val key = keyGenerator.generateKey()
val cipher = Cipher.getInstance("AES/GCM/NoPadding").apply { init(Cipher.ENCRYPT_MODE, key) }
val encrypted = cipher.doFinal(token.toByteArray())  // store `encrypted` + `cipher.iv`
```

In React Native, `react-native-keychain` and `expo-secure-store` wrap these two APIs — `AsyncStorage` does not.

## Network: HTTPS and certificate pinning

Both platforms block plain HTTP by default: **ATS** on iOS and, since Android 9, cleartext traffic disabled. Turning that off "to make it work in dev" and shipping it is a common mistake.

```xml
<!-- Android: res/xml/network_security_config.xml — HTTP forbidden everywhere -->
<network-security-config>
    <base-config cleartextTrafficPermitted="false" />
</network-security-config>
```

### Certificate pinning

HTTPS trusts any certificate signed by any of the hundreds of CAs the device trusts. That's not enough when an attacker can get a trusted certificate issued for the domain (a compromised CA), or convince the user to install their own CA (a corporate proxy, malware, or a test tool like Charles/mitmproxy). **Pinning** makes the app accept only the certificates it expects for its own backend, so a man-in-the-middle with a valid-but-unexpected certificate gets rejected.

What to pin:

| Pin | Survives certificate renewal | Notes |
|---|---|---|
| The certificate itself | ❌ | Breaks the app every time it renews (every ~90 days with Let's Encrypt) |
| The public key (SPKI hash) | ✅ if the key is reused | The usual choice |
| The intermediate CA | ✅ | Less strict, much safer to operate |

Getting the SPKI hash of the server's key:

```bash
openssl s_client -connect api.myapp.com:443 </dev/null 2>/dev/null \
  | openssl x509 -pubkey -noout \
  | openssl pkey -pubin -outform der \
  | openssl dgst -sha256 -binary | base64
```

```xml
<!-- Android: res/xml/network_security_config.xml -->
<network-security-config>
    <domain-config>
        <domain includeSubdomains="true">api.myapp.com</domain>
        <pin-set expiration="2027-06-01">
            <pin digest="SHA-256">Base64HashOfCurrentKey=</pin>
            <pin digest="SHA-256">Base64HashOfBackupKey=</pin>  <!-- backup, see below -->
        </pin-set>
    </domain-config>
</network-security-config>
```

```xml
<!-- iOS: Info.plist (iOS 14+) -->
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSPinnedDomains</key>
    <dict>
        <key>api.myapp.com</key>
        <dict>
            <key>NSIncludesSubdomains</key><true/>
            <key>NSPinnedLeafIdentities</key>
            <array>
                <dict><key>SPKI-SHA256-BASE64</key><string>Base64HashOfCurrentKey=</string></dict>
                <dict><key>SPKI-SHA256-BASE64</key><string>Base64HashOfBackupKey=</string></dict>
            </array>
        </dict>
    </dict>
</dict>
```

The same thing in code, for Android apps using OkHttp (the HTTP client behind Retrofit):

```kotlin
val client = OkHttpClient.Builder()
    .certificatePinner(
        CertificatePinner.Builder()
            .add("api.myapp.com", "sha256/Base64HashOfCurrentKey=")
            .add("api.myapp.com", "sha256/Base64HashOfBackupKey=")
            .build()
    )
    .build()
```

**The risk of pinning**: if the server's key changes and the app only knew the old pin, *every installed copy stops connecting* until the user updates — a self-inflicted outage that can last days. That's why it's always done with:

- A **backup pin** (the hash of a key generated in advance and kept offline), so rotating doesn't break the app.
- An **expiration** or a way to ship a new pin (a forced app update).
- Pinning only to your own backend, never to third-party domains whose certificates you don't control.

Pinning is also not unbreakable: on a rooted/jailbroken device, tools like Frida or objection can disable the check at runtime. It raises the cost of an attack, it doesn't make it impossible.

## Device integrity

A **jailbroken** (iOS) or **rooted** (Android) device has the OS protections removed: the sandbox doesn't hold, Keychain/Keystore contents can be reached, and code can be hooked at runtime. Detecting it on the client (looking for `su`, Cydia, writable system paths) is easy to bypass — the check itself lives in code the attacker controls.

The robust version is **attestation**: the OS vendor signs a statement that "this is the genuine app, on a real unmodified device", and the *backend* verifies that signature before trusting the request.

- **iOS**: App Attest (`DCAppAttestService`) and DeviceCheck.
- **Android**: Play Integrity API (the replacement for SafetyNet Attestation).

Typical use: sensitive operations (a payment, an account creation, a promo claim) require a fresh attestation token, which the server validates.

## Biometrics

Face ID / Touch ID (iOS, `LocalAuthentication`) and `BiometricPrompt` (Android) confirm that the person holding the device is its owner.

```swift
let context = LAContext()
context.evaluatePolicy(.deviceOwnerAuthenticationWithBiometrics,
                       localizedReason: "Confirm the payment") { success, error in
    if success { /* continue */ }
}
```

On its own that is a `Bool` returned by code on the device — on a compromised device it can be forced to `true`. The robust version **ties a key to the biometric**, so the secret is physically unusable without it:

```kotlin
// Android: the Keystore key can only be used after a successful biometric check
KeyGenParameterSpec.Builder("token_key", KeyProperties.PURPOSE_ENCRYPT or KeyProperties.PURPOSE_DECRYPT)
    .setUserAuthenticationRequired(true)
    .build()
```

On iOS the equivalent is storing the Keychain item with `SecAccessControlCreateWithFlags(..., .biometryCurrentSet, ...)`.

## Secrets and reverse engineering

An Android APK is a zip that tools like `jadx` turn back into near-readable code in minutes; an iOS binary can be dumped from a jailbroken device; a React Native app ships its JavaScript bundle readable (even compiled to Hermes bytecode it can be decompiled). So:

```kotlin
// ❌ anyone who downloads the app can extract this
const val PAYMENTS_SECRET_KEY = "sk_live_..."

// ✅ the app only holds a short-lived user token; the secret key lives on the backend
val response = api.createPayment(userToken, amount)  // the backend calls the payment provider
```

- Keys that are "public by design" (a Google Maps key, Firebase config) are fine in the app — they should be restricted by app ID/signature in the provider's console.
- **Obfuscation** (R8/ProGuard on Android) renames classes and removes dead code, which only slows down someone reading the code — it's not protection for a secret.

```groovy
// Android: build.gradle — enable R8 in release builds
buildTypes {
    release {
        minifyEnabled true
        proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
    }
}
```

## Deep links

A custom scheme like `myapp://reset?token=...` can be registered by **any** other app on the device, which would then receive that link (and the token in it). The verified alternative is a normal `https://` link that the OS only opens in the app if the domain publicly vouches for it:

- **iOS Universal Links**: `https://myapp.com/.well-known/apple-app-site-association`
- **Android App Links**: `https://myapp.com/.well-known/assetlinks.json`, with `android:autoVerify="true"` in the manifest.

`assetlinks.json` is the domain saying "only the app with this package and signing certificate may open my links":

```json
[{
  "relation": ["delegate_permission/common.handle_all_urls"],
  "target": {
    "namespace": "android_app",
    "package_name": "com.myapp",
    "sha256_cert_fingerprints": ["AA:BB:CC:..."]
  }
}]
```

Whatever arrives through a link is user input: it gets validated like any other request parameter, never trusted for being "internal to the app".

## Quick checklist

- [ ] No secrets or private keys in the binary.
- [ ] Tokens in Keychain / Keystore, not in `UserDefaults` / `SharedPreferences` / `AsyncStorage`.
- [ ] HTTPS only (ATS / `cleartextTrafficPermitted="false"`).
- [ ] Certificate pinning (when the risk justifies it) with a backup pin and an expiration.
- [ ] Sensitive operations verified on the backend, with attestation if the risk requires it.
- [ ] No sensitive data in logs (`print`, `Log.d`) or in screenshots of the recent-apps view.
- [ ] Verified deep links (Universal / App Links) instead of custom schemes for sensitive flows.
- [ ] Least-privilege permissions, requested when needed.

The reference catalog of risks and tests is the **OWASP MASVS** (requirements) and **MASTG** (how to test them), with the **OWASP Mobile Top 10** as the shorter list.

---
Related: [Authentication and Security](../backend/authentication.md), [Security Headers](security-headers.md), [Zero Trust](zero-trust.md) (mTLS), [React Native](../stacks/react-native/fundamentals.md), [Access token vs refresh token](../backend/authentication.md#6-access-token-vs-refresh-token).
