# SUNTEK SDK Selection & App Signing Architecture

## 1. Choosing the Right SDK Repository

SUNTEK organizes public hardware SDKs under canonical `suntek-sdk-*` repositories grouped by hardware/board compatibility rather than generic product category:

- **SC1 / SK1 / SK1-15**: Use [`suntek-sdk-sc1-sk1`](https://github.com/SUNTEK-Official/suntek-sdk-sc1-sk1) (`SdkApiJar` GMS & Non-GMS)
- **SC5 / SK6**: Use [`suntek-sdk-sc5-sk6`](https://github.com/SUNTEK-Official/suntek-sdk-sc5-sk6) (`SdkApiJar` + WS2812B `/dev/ttyS1` LED protocol)
- **ST5**: Use [`suntek-sdk-st5`](https://github.com/SUNTEK-Official/suntek-sdk-st5) (`SdkApiJar` + CAN/Display control)
- **SK7**: Use [`suntek-sdk-sk7`](https://github.com/SUNTEK-Official/suntek-sdk-sk7)
- **SK5 Fingerprint**: Use [`suntek-sdk-sk5-fingerprint`](https://github.com/SUNTEK-Official/suntek-sdk-sk5-fingerprint)
- **SM1 / SM6 Smart POS**: Use [`suntek-sdk-sm1`](https://github.com/SUNTEK-Official/suntek-sdk-sm1) / [`suntek-sdk-sm6`](https://github.com/SUNTEK-Official/suntek-sdk-sm6)
- **FA5 / FA8 / FA10**: Use [`suntek-sdk-fa5`](https://github.com/SUNTEK-Official/suntek-sdk-fa5) / [`suntek-sdk-fa8-fa10`](https://github.com/SUNTEK-Official/suntek-sdk-fa8-fa10)
- **AI-Box / SCP80 / PV2 / NFC**: Use `suntek-sdk-aibox`, `suntek-sdk-printer-scp80`, `suntek-sdk-speaker-pv2`, `suntek-sdk-nfc`

## 2. Application Signing Architecture

Customer Android applications should be signed with your own **Google Play App Signing** certificate or **enterprise release key**. Terminals support three seamless integration levels without requiring custom ROM modifications:

| Integration Mode | How It Works | App Signing Certificate | Ideal Use Cases |
| :--- | :--- | :--- | :--- |
| **Standard System Service IPC** | SUNTEK SDKs communicate via Binder IPC with preinstalled terminal system services (`com.sdkapi.sdkapiservice`, `sdk.service`, `AppServer`) | Customer Play / Enterprise Key | Hardware I/O, serial/GPIO, printer, card reader, display control |
| **Companion Bridge APK (`SuntekBridgeService.apk`)** | Install the signed `SuntekBridgeService.apk` companion from GitHub Releases via `adb` or MDM and invoke allowlisted AIDL endpoints | Customer Play / Enterprise Key | Silent install, system properties, Ethernet/Wi-Fi policy on managed deployments |
| **Enterprise Signing Support** | Contact SUNTEK enterprise support (`support@suntekai.com`) for managed application signing workflows | SUNTEK Enterprise Release Signing | Specialized system apps explicitly declaring `android:sharedUserId="android.uid.system"` |
