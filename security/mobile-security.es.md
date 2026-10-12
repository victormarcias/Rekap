# Seguridad Mobile (iOS y Android)

Una app mobile corre en un dispositivo que controla el usuario: el binario se puede decompilar, el tráfico se puede interceptar con un proxy, y en un dispositivo con root/jailbreak hasta la memoria de la app se puede inspeccionar. La regla que se desprende es la misma que en un browser: **el cliente nunca es de confianza** — todo lo que importa (precios, permisos, validaciones) se decide en el backend, y nada secreto viaja dentro de la app.

| Área | iOS | Android |
|---|---|---|
| Permisos | Textos de uso en `Info.plist` + prompt en runtime | `AndroidManifest.xml` + prompt en runtime |
| Almacenamiento seguro | Keychain | Android Keystore |
| HTTPS obligatorio | ATS (App Transport Security) | Network Security Config |
| Certificate pinning | `NSPinnedDomains` (iOS 14+), TrustKit | `<pin-set>` en Network Security Config, `CertificatePinner` de OkHttp |
| Integridad del dispositivo | App Attest / DeviceCheck | Play Integrity API |
| Biometría | `LocalAuthentication` (Face ID / Touch ID) | `BiometricPrompt` |
| Protección del código | Binario compilado (igual se puede volcar) | Ofuscación con R8 / ProGuard |
| Deep links verificados | Universal Links | App Links |

## Sandbox y permisos

Cada app corre aislada: no puede leer los archivos ni la memoria de otra. El acceso a algo sensible (cámara, ubicación, contactos) requiere un permiso, que el usuario concede en runtime. Conviene pedir solo lo que la feature realmente necesita, y lo más tarde posible — una app que pide todo al arrancar recibe rechazos, y es un objetivo más grande si la comprometen.

```xml
<!-- iOS: Info.plist — el texto que se muestra en el prompt de permiso es obligatorio -->
<key>NSCameraUsageDescription</key>
<string>La cámara se usa para escanear el QR de pago.</string>
```

```xml
<!-- Android: AndroidManifest.xml -->
<uses-permission android:name="android.permission.CAMERA" />
```

## Almacenamiento seguro

Tokens, passwords y claves no van en `UserDefaults`, `SharedPreferences`, `AsyncStorage` ni en archivos planos: son archivos sin cifrar dentro de la carpeta de la app, legibles en un dispositivo con root o desde un backup del dispositivo.

- **Keychain (iOS)**: almacén cifrado administrado por el SO, sobrevive a la reinstalación de la app y se puede restringir al dispositivo desbloqueado.
- **Android Keystore**: guarda claves de cifrado en hardware seguro (TEE o StrongBox) — el material de la clave nunca sale de ahí. La app cifra sus datos con esa clave y guarda solo el texto cifrado.

```swift
// iOS: guardar un token en el Keychain
let query: [String: Any] = [
    kSecClass as String: kSecClassGenericPassword,
    kSecAttrAccount as String: "access_token",
    kSecValueData as String: Data(token.utf8),
    kSecAttrAccessible as String: kSecAttrAccessibleWhenUnlockedThisDeviceOnly,  // no va a backups, solo con el equipo desbloqueado
]
SecItemDelete(query as CFDictionary)  // reemplaza cualquier valor anterior
SecItemAdd(query as CFDictionary, nil)
```

```kotlin
// Android: clave generada dentro del Keystore, usada para cifrar el token
val keyGenerator = KeyGenerator.getInstance(KeyProperties.KEY_ALGORITHM_AES, "AndroidKeyStore")
keyGenerator.init(
    KeyGenParameterSpec.Builder("token_key", KeyProperties.PURPOSE_ENCRYPT or KeyProperties.PURPOSE_DECRYPT)
        .setBlockModes(KeyProperties.BLOCK_MODE_GCM)
        .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_NONE)
        .build()
)
val key = keyGenerator.generateKey()
val cipher = Cipher.getInstance("AES/GCM/NoPadding").apply { init(Cipher.ENCRYPT_MODE, key) }
val encrypted = cipher.doFinal(token.toByteArray())  // guardar `encrypted` + `cipher.iv`
```

En React Native, `react-native-keychain` y `expo-secure-store` envuelven estas dos APIs — `AsyncStorage` no.

## Red: HTTPS y certificate pinning

Las dos plataformas bloquean HTTP plano por defecto: **ATS** en iOS y, desde Android 9, el tráfico cleartext deshabilitado. Apagar eso "para que ande en dev" y publicarlo así es un error común.

```xml
<!-- Android: res/xml/network_security_config.xml — HTTP prohibido en todos lados -->
<network-security-config>
    <base-config cleartextTrafficPermitted="false" />
</network-security-config>
```

### Certificate pinning

HTTPS confía en cualquier certificado firmado por cualquiera de las cientos de CAs que el dispositivo considera de confianza. Eso no alcanza cuando un atacante puede conseguir un certificado válido para el dominio (una CA comprometida), o convencer al usuario de instalar su propia CA (un proxy corporativo, malware, o una herramienta de testing como Charles/mitmproxy). El **pinning** hace que la app acepte solo los certificados que espera para su propio backend, así un man-in-the-middle con un certificado válido pero inesperado es rechazado.

Qué pinear:

| Pin | Sobrevive a la renovación del certificado | Notas |
|---|---|---|
| El certificado en sí | ❌ | Rompe la app cada vez que se renueva (cada ~90 días con Let's Encrypt) |
| La clave pública (hash SPKI) | ✅ si la clave se reutiliza | La opción habitual |
| La CA intermedia | ✅ | Menos estricto, mucho más seguro de operar |

Obtener el hash SPKI de la clave del servidor:

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
            <pin digest="SHA-256">HashBase64DeLaClaveActual=</pin>
            <pin digest="SHA-256">HashBase64DeLaClaveDeBackup=</pin>  <!-- backup, ver abajo -->
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
                <dict><key>SPKI-SHA256-BASE64</key><string>HashBase64DeLaClaveActual=</string></dict>
                <dict><key>SPKI-SHA256-BASE64</key><string>HashBase64DeLaClaveDeBackup=</string></dict>
            </array>
        </dict>
    </dict>
</dict>
```

Lo mismo por código, para apps Android que usan OkHttp (el cliente HTTP detrás de Retrofit):

```kotlin
val client = OkHttpClient.Builder()
    .certificatePinner(
        CertificatePinner.Builder()
            .add("api.myapp.com", "sha256/HashBase64DeLaClaveActual=")
            .add("api.myapp.com", "sha256/HashBase64DeLaClaveDeBackup=")
            .build()
    )
    .build()
```

**El riesgo del pinning**: si la clave del servidor cambia y la app solo conocía el pin viejo, *todas las copias instaladas dejan de conectarse* hasta que el usuario actualice — una caída autoinfligida que puede durar días. Por eso se hace siempre con:

- Un **pin de backup** (el hash de una clave generada de antemano y guardada offline), para que rotar no rompa la app.
- Una **expiración** o una forma de enviar un pin nuevo (un update forzado de la app).
- Pinear solo contra el backend propio, nunca contra dominios de terceros cuyos certificados no controlás.

El pinning tampoco es infranqueable: en un dispositivo con root/jailbreak, herramientas como Frida u objection pueden desactivar la verificación en runtime. Sube el costo de un ataque, no lo hace imposible.

## Integridad del dispositivo

Un dispositivo con **jailbreak** (iOS) o **root** (Android) tiene las protecciones del SO removidas: el sandbox no se sostiene, se puede llegar al contenido del Keychain/Keystore, y el código se puede hookear en runtime. Detectarlo desde el cliente (buscar `su`, Cydia, rutas del sistema escribibles) es fácil de esquivar — el chequeo vive en código que controla el atacante.

La versión robusta es la **atestación**: el proveedor del SO firma una declaración de que "esta es la app genuina, en un dispositivo real sin modificar", y el *backend* verifica esa firma antes de confiar en el request.

- **iOS**: App Attest (`DCAppAttestService`) y DeviceCheck.
- **Android**: Play Integrity API (el reemplazo de SafetyNet Attestation).

Uso típico: las operaciones sensibles (un pago, crear una cuenta, canjear una promo) exigen un token de atestación fresco, que el servidor valida.

## Biometría

Face ID / Touch ID (iOS, `LocalAuthentication`) y `BiometricPrompt` (Android) confirman que quien tiene el dispositivo en la mano es su dueño.

```swift
let context = LAContext()
context.evaluatePolicy(.deviceOwnerAuthenticationWithBiometrics,
                       localizedReason: "Confirmá el pago") { success, error in
    if success { /* continuar */ }
}
```

Por sí solo eso es un `Bool` devuelto por código que corre en el dispositivo — en uno comprometido se puede forzar a `true`. La versión robusta **ata una clave a la biometría**, así el secreto es físicamente inutilizable sin ella:

```kotlin
// Android: la clave del Keystore solo se puede usar tras una verificación biométrica exitosa
KeyGenParameterSpec.Builder("token_key", KeyProperties.PURPOSE_ENCRYPT or KeyProperties.PURPOSE_DECRYPT)
    .setUserAuthenticationRequired(true)
    .build()
```

En iOS el equivalente es guardar el ítem del Keychain con `SecAccessControlCreateWithFlags(..., .biometryCurrentSet, ...)`.

## Secretos e ingeniería inversa

Un APK de Android es un zip que herramientas como `jadx` convierten de vuelta en código casi legible en minutos; un binario de iOS se puede volcar desde un dispositivo con jailbreak; una app React Native distribuye su bundle de JavaScript legible (incluso compilado a bytecode de Hermes se puede decompilar). Entonces:

```kotlin
// ❌ cualquiera que descargue la app puede extraer esto
const val PAYMENTS_SECRET_KEY = "sk_live_..."

// ✅ la app solo tiene un token de usuario de vida corta; la clave secreta vive en el backend
val response = api.createPayment(userToken, amount)  // el backend llama al proveedor de pagos
```

- Las claves "públicas por diseño" (una key de Google Maps, la config de Firebase) están bien en la app — se restringen por app ID/firma en la consola del proveedor.
- La **ofuscación** (R8/ProGuard en Android) renombra clases y elimina código muerto, lo que solo demora a quien lee el código — no es protección para un secreto.

```groovy
// Android: build.gradle — activar R8 en los builds de release
buildTypes {
    release {
        minifyEnabled true
        proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
    }
}
```

## Deep links

Un esquema custom como `myapp://reset?token=...` lo puede registrar **cualquier** otra app del dispositivo, que entonces recibiría ese link (y el token que lleva). La alternativa verificada es un link `https://` normal que el SO solo abre en la app si el dominio lo avala públicamente:

- **iOS Universal Links**: `https://myapp.com/.well-known/apple-app-site-association`
- **Android App Links**: `https://myapp.com/.well-known/assetlinks.json`, con `android:autoVerify="true"` en el manifest.

`assetlinks.json` es el dominio diciendo "solo la app con este package y este certificado de firma puede abrir mis links":

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

Todo lo que llega por un link es input del usuario: se valida como cualquier otro parámetro de un request, nunca se confía en él por ser "interno de la app".

## Checklist rápido

- [ ] Sin secretos ni claves privadas en el binario.
- [ ] Tokens en Keychain / Keystore, no en `UserDefaults` / `SharedPreferences` / `AsyncStorage`.
- [ ] Solo HTTPS (ATS / `cleartextTrafficPermitted="false"`).
- [ ] Certificate pinning (cuando el riesgo lo justifica) con pin de backup y expiración.
- [ ] Operaciones sensibles verificadas en el backend, con atestación si el riesgo lo requiere.
- [ ] Sin datos sensibles en logs (`print`, `Log.d`) ni en la vista de apps recientes.
- [ ] Deep links verificados (Universal / App Links) en vez de esquemas custom para flujos sensibles.
- [ ] Permisos de mínimo privilegio, pedidos cuando se necesitan.

El catálogo de referencia de riesgos y pruebas es **OWASP MASVS** (requisitos) y **MASTG** (cómo probarlos), con el **OWASP Mobile Top 10** como la lista más corta.

---
Relacionado: [Autenticación y Seguridad](../backend/authentication.es.md), [Security Headers](security-headers.es.md), [Zero Trust](zero-trust.es.md) (mTLS), [React Native](../stacks/react-native/fundamentals.es.md), [Access token vs Refresh token](../backend/authentication.es.md#6-access-token-vs-refresh-token).
