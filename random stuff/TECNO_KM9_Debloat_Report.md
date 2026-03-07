# TECNO KM9 Debloat Safety Report

**Android 15 | HiOS | MediaTek MT6789 | Build: TP1A.220624.014/155005**  
**Source:** live `adb shell pm list packages -f` audit

---

## Package Count Summary

| Tier | Package Count | Action |
|------|--------------|--------|
| TIER 0: Never Touch | 72 packages | Do not touch under any circumstances |
| TIER 1: Disable Only | 45 packages | `pm disable-user --user 0` |
| TIER 2: Safe to Remove | 93 packages | `pm uninstall --user 0` |

---

## Golden Rules — Read Before Touching Anything

> **WARNING: DISABLE first. Test for 48 hours. REMOVE only if device is stable. Every removal is a one-way door.**

- Use `pm disable-user --user 0 <package>` first — not `pm uninstall` — for all system packages.
- **APEX partition packages** (Mainline modules) — **NEVER touch these.** They are updated by Google.
- **VENDOR partition packages** — hardware resource overlays. Never touch.
- Uninstalling `--user 0` removes from user space but leaves on partition. Recovery: `adb shell cmd package install-existing <package>`
- **NEVER remove `com.android.systemui` or `com.transsion.hilauncher` while on live system.** That is exactly how you bricked it.
- Remove **ONE package at a time.** Reboot. Test. Then proceed.

---

## ADB Command Reference

```bash
# Disable a package (reversible)
adb shell pm disable-user --user 0 <package.name>

# Re-enable a package
adb shell pm enable --user 0 <package.name>

# Uninstall for user (preferred over hard uninstall)
adb shell pm uninstall --user 0 <package.name>

# Restore a user-uninstalled package
adb shell cmd package install-existing <package.name>
```

---

## TIER 0 — NEVER TOUCH

> **WARNING: Removing ANY package from this tier will soft brick or render the device unusable.**

### Android Core, Providers & Google Framework

> NOTE: Android OS internals + Google Play Services stack. Untouchable.

| Package Name | App Name | Partition | Notes |
|---|---|---|---|
| `android` | Android Framework | SYSTEM | Core OS. Remove = instant brick. |
| `com.android.systemui` | System UI | SYSTEM_EXT_PRIV | Status bar, notifications. Likely what crashed previously. |
| `com.android.phone` | Phone App | SYSTEM_PRIV | All phone calls. Remove = no calls. |
| `com.android.server.telecom` | Telecom Server | SYSTEM_PRIV | Telephony framework. Non-negotiable. |
| `com.android.shell` | ADB Shell | SYSTEM_PRIV | Remove = lose ADB permanently. |
| `com.android.keychain` | Key Chain | SYSTEM | Security key storage. |
| `com.android.inputdevices` | Input Devices | SYSTEM_PRIV | Touch/keyboard input. Remove = unresponsive screen. |
| `com.android.credentialmanager` | Credential Manager | SYSTEM_PRIV | Auth framework. |
| `com.android.localtransport` | Local Transport | SYSTEM_PRIV | Backup transport layer. |
| `com.android.location.fused` | Fused Location | SYSTEM_PRIV | GPS/location. Remove = no maps, weather, etc. |
| `com.android.intentresolver` | Intent Resolver | SYSTEM_PRIV | Share sheet. Remove = app-to-app sharing fails. |
| `com.android.settings` | Settings | SYSTEM_EXT_PRIV | Settings app. |
| `com.android.settings.intelligence` | Settings Search | SYSTEM_EXT | Settings search. |
| `com.android.bluetooth` | Bluetooth | APEX | Bluetooth stack. |
| `com.android.nfc` | NFC | SYSTEM_EXT | NFC stack. Remove = no NFC/payments. |
| `com.android.se` | Secure Element | SYSTEM | NFC payment chip. |
| `com.android.mms.service` | MMS Service | SYSTEM_PRIV | SMS/MMS. |
| `com.android.mtp` | MTP/USB | SYSTEM_PRIV | USB file transfer. |
| `com.android.certinstaller` | Cert Installer | SYSTEM | SSL certs. VPN dependency. |
| `com.android.vpndialogs` | VPN Dialogs | SYSTEM_PRIV | VPN framework. |
| `com.android.ons` | Opportunistic Network | SYSTEM_PRIV | Dual SIM management. |
| `com.android.providers.contacts` | Contacts DB | SYSTEM_PRIV | ALL contacts. Remove = data loss. |
| `com.android.providers.telephony` | Telephony DB | SYSTEM_PRIV | SMS/call history database. |
| `com.android.providers.settings` | Settings DB | SYSTEM_EXT_PRIV | All system settings. Remove = settings reset loop. |
| `com.android.providers.media` | Media Provider | SYSTEM_PRIV | Media scanner. Remove = no media in apps. |
| `com.android.providers.calendar` | Calendar DB | SYSTEM_PRIV | Calendar data provider. |
| `com.android.providers.downloads` | Download Manager | SYSTEM_PRIV | Download infra. Remove = installs fail. |
| `com.android.providers.blockednumber` | Blocked Numbers | SYSTEM_PRIV | Blocked caller DB. |
| `com.android.providers.contactkeys` | Contact Keys | SYSTEM_PRIV | Encryption keys for contacts. |
| `com.android.providers.userdictionary` | User Dictionary | SYSTEM_PRIV | Custom autocorrect words. |
| `com.android.externalstorage` | External Storage | SYSTEM_PRIV | SD card access layer. |
| `com.google.android.gms` | Google Play Services | APEX | CRITICAL. Removing kills ~70% of installed apps. |
| `com.google.android.gsf` | Google Services Framework | SYSTEM_EXT_PRIV | GMS backbone. |
| `com.google.android.networkstack` | Network Stack | SYSTEM_PRIV | Android network layer. Remove = no connectivity. |
| `com.google.android.packageinstaller` | Package Installer | SYSTEM_EXT_PRIV | App installation. Remove = can't install anything. |
| `com.google.android.permissioncontroller` | Permission Controller | APEX | Runtime permissions. |
| `com.google.android.webview` | Android WebView | PRODUCT | Web renderer. Many apps' embedded browsers depend on this. |
| `com.android.vending` | Google Play Store | DATA | App store. |

### MediaTek Hardware Layer

> NOTE: MTK HAL services. Removing IMS will break ALL voice calls on this 5G device.

| Package Name | App Name | Partition | Notes |
|---|---|---|---|
| `com.mediatek.ims` | MediaTek IMS | SYSTEM_EXT_PRIV | VoLTE/VoWiFi. CRITICAL for 5G calls. Remove = calls fail on 4G/5G. |
| `com.mediatek.telephony` | MTK Telephony | SYSTEM_EXT_PRIV | MTK radio layer. Remove = no cellular. |
| `com.mediatek.simprocessor` | SIM Processor | SYSTEM_EXT_PRIV | SIM card handling. |
| `com.mediatek.gbaservice` | GBA Service | SYSTEM_EXT_PRIV | Network/carrier auth service. |
| `com.mediatek.location.lppe.main` | MTK Location Engine | SYSTEM_EXT_PRIV | GPS precision engine. |
| `com.mediatek.location.mtkgeofence` | MTK Geofence | SYSTEM_EXT_PRIV | Geofencing service. |

### Tecno HiOS Core

> NOTE: HiOS services with deep interdependencies. `cloudserver` is what triggered the previous brick.

| Package Name | App Name | Partition | Notes |
|---|---|---|---|
| `com.transsion.hilauncher` | HiLauncher | SYSTEM_EXT_PRIV | THE LAUNCHER. Remove = black screen, no UI. Only disable if replacing with Nova/etc. |
| `com.transsion.resolver` | Transsion Resolver | SYSTEM_EXT_PRIV | HiOS intent resolver. Remove = share menus break. |
| `com.transsion.tabe` | TABE | SYSTEM_EXT_PRIV | Likely core framework component. Unknown internals. |
| `com.transsion.trancare` | TranCare | SYSTEM_EXT_PRIV | Core Tecno care/health service. |
| `com.transsion.tranengine` | TranEngine | SYSTEM_EXT | Core HiOS runtime engine. Touch = undefined behavior. |
| `com.transsion.necessity` | Necessity | SYSTEM_EXT | Name literally means essential. Core dependency layer. |
| `com.transsion.spl` | SPL | PRODUCT | Likely system policy/permission layer. High risk. |
| `com.transsion.spld` | SPLD | SYSTEM_EXT | Companion to SPL. Same risk. |
| `com.transsion.succ` | SUCC | SYSTEM_EXT | Possibly system usage/core component. Disable only. |
| `com.transsion.aisupportercore` | AI Supporter Core | SYSTEM_PRIV | Core AI service provider. Other AI apps depend on this. |
| `com.transsion.cloudserver` | Cloud Server | SYSTEM_EXT_PRIV | Tecno account backend. **THIS triggered the previous brick. Never uninstall.** |
| `com.transsion.settings.wifi` | WiFi Settings | SYSTEM_EXT_PRIV | HiOS WiFi settings panel. |
| `com.transsion.settings.bluetooth` | BT Settings | SYSTEM_EXT_PRIV | HiOS Bluetooth settings panel. |
| `com.transsion.settings.nfc` | NFC Settings | SYSTEM_EXT_PRIV | HiOS NFC settings panel. |
| `com.transsion.tele.settings` | Telecom Settings | SYSTEM_EXT_PRIV | HiOS telecom/call settings. |
| `com.transsion.plat.appupdate` | App Update Platform | SYSTEM_EXT_PRIV | System OTA/app update service. |
| `com.transsion.overlaysuw` | Setup Wizard Overlay | SYSTEM_EXT | Factory reset/setup wizard. |
| `com.transsion.keyguardclock` | Keyguard Clock | SYSTEM_EXT | Lock screen clock component. |
| `com.transsion.keyguardtheme` | Keyguard Theme | SYSTEM_EXT | Lock screen theming. HiOS depends on this. |
| `com.transsion.os.typeface` | OS Typeface | SYSTEM_EXT_PRIV | System fonts. Remove = corrupted UI rendering. |
| `com.transsion.dynamicbar` | Dynamic Bar | SYSTEM_EXT | Notification/status bar component. |
| `com.transsion.ossettingsext` | OS Settings Extension | SYSTEM_EXT | HiOS settings pages. Remove = settings pages disappear. |
| `com.transsion.phonemanager` | Phone Manager Service | SYSTEM_EXT_PRIV | Core phone management backend (not just the app). |
| `com.transsion.applock` | App Lock | SYSTEM_EXT_PRIV | System-level app locking framework. |
| `com.hilauncherconfig` | HiLauncher Config | TR_PRODUCT | Config data for HiLauncher. Remove = launcher loses settings. |
| `com.transsion` | Transsion Base | SYSTEM | Base Transsion package. Core dependency. |
| `com.transsion.thub.res` | T-Hub Resources | SYSTEM | Resource package. |
| `com.transsion.trproductframeworkresoverlay` | TR Framework Overlay | TR_PRODUCT | Framework resource overlay. |

---

## TIER 1 — DISABLE ONLY (Do Not Uninstall)

> NOTE: These can be safely DISABLED but must NOT be uninstalled. Disable one at a time. Reboot. Verify.

| Package Name | App Name | Partition | Notes |
|---|---|---|---|
| `com.transsion.zahooc` | Za-Hooc (Privacy Guard) | SYSTEM_EXT_PRIV | Intercepts permissions — feeds empty data to greedy apps. Useful privacy tool. Disable only, do NOT uninstall. |
| `com.transsion.teop` | TEOP Push Service | PRODUCT | Tecno push notification/ad delivery. Disable for privacy. |
| `com.transsion.statisticalsales` | Statistical Sales | SYSTEM_EXT | Sales analytics telemetry. Zero user value. Disable. |
| `com.transsion.personalizedService.hios` | Personalized Service | SYSTEM_EXT | Ad personalization engine. Disable for privacy. |
| `com.transsion.magazineservice.hios` | Magazine Service | SYSTEM_EXT | Feeds content to launcher lockscreen. Disable if not using. |
| `com.transsion.microintelligence` | Micro Intelligence | SYSTEM_EXT | Background AI scene analysis. Battery drain. Disable. |
| `com.transsion.uxdetector` | UX Detector | SYSTEM_EXT | UI interaction telemetry. Disable. |
| `com.transsion.tecnospot` | TecnoSpot | TR_PRODUCT | Tecno news/content feed. Disable for privacy. |
| `com.transsion.aivoiceassistant` | AI Voice Assistant | SYSTEM_EXT_PRIV | Tecno voice assistant. Disable if not using. |
| `com.transsion.overlay.aivoiceassistant` | AI Voice Overlay | PRODUCT | Resource overlay for voice assistant. |
| `com.transsion.kolun.aiservice` | Kolun AI Service | SYSTEM_EXT_PRIV | Background AI inference. Disable if not using Kolun. |
| `com.transsion.kolun.assistant` | Kolun Assistant | SYSTEM_EXT | Kolun AI assistant frontend. |
| `com.transsion.aiassistantlifestyle` | AI Lifestyle Assistant | SYSTEM_EXT | AI lifestyle suggestions. |
| `com.transsion.smartrecognition` | Smart Recognition | SYSTEM_EXT | AI camera scene recognition. |
| `com.transsion.faceid` | Face ID | SYSTEM_EXT | Face unlock. Disable if not using. |
| `com.transsion.tranfacmode` | Face Mode | SYSTEM_EXT | Face beautification backend. |
| `com.transsion.letswitch` | LetSwitch | SYSTEM_EXT | Phone switching/transition feature. |
| `com.transsion.mol` | MOL | SYSTEM_EXT | Unknown service. Disable only, monitor for issues. |
| `com.transsion.sru` | SRU | SYSTEM_EXT | Unknown. Possibly System Recovery Utility. Disable only. |
| `com.transsion.sk` | SK | TR_PRODUCT | Unknown. Disable only. Do not remove without identifying. |
| `com.xui.xhide` | XHide (Notch) | SYSTEM_EXT | Notch/status bar hiding. Safe to disable. |
| `com.scorpio.securitycom` | SecurityCom | PRODUCT | Security component of unknown depth. Disable only. |
| `com.talpa.hiservice` | HiService (Talpa) | SYSTEM_EXT | Browser backend. Disable if not using HiBrowser. |
| `com.geniex.vsimhelper` | vSIM Helper | SYSTEM_EXT_PRIV | Virtual/eSIM helper. Only relevant for eSIM users. |
| `com.debug.loggerui` | Logger UI | SYSTEM_EXT | Debug log capture. Disable. |
| `com.mediatek.engineermode` | Engineer Mode | SYSTEM_EXT | MTK engineering diagnostics. Disable. |
| `com.mediatek.mdmconfig` | MDM Config | SYSTEM_EXT | Mobile Device Management config. Disable. |
| `com.mediatek.mdmlsample` | MDM Sample | SYSTEM_EXT | MDM sample app. Disable. |
| `com.mediatek.ygps` | MTK YGPS | SYSTEM_EXT | GPS diagnostic utility. Disable. |
| `com.mediatek.atmwifimeta` | ATM WiFi Meta | SYSTEM_EXT | WiFi factory test tool. Disable. |
| `com.mediatek.voiceunlock` | Voice Unlock | SYSTEM_EXT_PRIV | MTK voice unlock. Disable if not using. |
| `com.mediatek.schpwronoff` | Scheduled Power | SYSTEM_EXT | Scheduled power on/off. Disable if not using. |
| `com.google.android.adservices.api` | Google AdServices API | APEX | Google Ads SDK. Disable for privacy. |
| `com.google.android.ondevicepersonalization.services` | On-Device Personalization | APEX | On-device ad targeting. Disable. |
| `com.google.android.federatedcompute` | Federated Compute | APEX | Federated learning for ads. Disable. |
| `com.google.mainline.adservices` | Mainline AdServices | PRODUCT | Google ads mainline module. Disable. |
| `com.google.mainline.telemetry` | Google Telemetry | PRODUCT | Google usage telemetry. Disable. |
| `com.google.android.gms.location.history` | Location History | PRODUCT | Stores location history. Disable for privacy. |
| `com.google.android.gms.supervision` | Family Supervision | PRODUCT | Already disabled. Leave it. |
| `com.transsion.aod` | Always-On Display | SYSTEM_EXT_PRIV | AOD service. Disable if not using — battery drain. |
| `com.transsion.magicshow` | Magic Show | TR_PRELOAD | Lock screen animation. Safe to disable. |
| `com.funbase.xradio` | XRadio | PRODUCT | Streaming radio. Disable if not using. |
| `com.transsion.iotservice` | IoT Service | SYSTEM_EXT | Smart device integration. Disable if not using smart home. |
| `com.transsion.iotcard` | IoT Card | SYSTEM_EXT | IoT quick access card. Disable if not using. |
| `com.transsion.tranradionet` | Tran Radio Net | SYSTEM_EXT | Network radio service. Disable only. |

---

## TIER 2 — SAFE TO REMOVE

> NOTE: Use `adb shell pm uninstall --user 0 <package>` | All recoverable via `install-existing`.

### Facebook Suite

> **WARNING: Facebook App Manager is the most dangerous — it silently phones home and auto-updates Facebook.**

| Package Name | App Name | Partition | Notes |
|---|---|---|---|
| `com.facebook.katana` | Facebook | TR_PRELOAD | Facebook main app. |
| `com.facebook.appmanager` | Facebook App Manager | TR_PRELOAD | Silently updates Facebook, high telemetry risk. Remove immediately. |
| `com.facebook.services` | Facebook Services | TR_PRELOAD | System-level Facebook service hooks. |
| `com.facebook.system` | Facebook System | TR_PRELOAD | Facebook system integration. |
| `com.instagram.android` | Instagram | TR_PRELOAD | Reinstallable from Play Store. |

### Third-Party Bloat

> NOTE: Pre-installed third-party apps with zero system role.

| Package Name | App Name | Partition | Notes |
|---|---|---|---|
| `com.zhiliaoapp.musically` | TikTok | TR_PRELOAD | Reinstallable from Play Store. |
| `cn.wps.moffice.lite.abroad.transsion` | WPS Office | TR_PRELOAD | Remove if not using. |
| `net.bat.store` | BAT Store | TR_PRELOAD | Third-party app store. |
| `com.transsnet.store` | Transsnet Store | TR_PRELOAD | Tecno app store. Remove if using Play Store. |
| `com.driftsand.game.blockfunny` | CrushBlock (Game) | TR_PRELOAD | Pre-installed game. Obvious bloat. |
| `com.zaz.translate` | Zaz Translate | TR_PRELOAD | Third-party translator. |
| `com.smartlife.nebula` | SmartLife | TR_PRELOAD | Tuya IoT app. Remove unless using smart home. |
| `com.idea.questionnaire` | Questionnaire | SYSTEM_EXT | Survey/feedback app. |
| `com.hoffnung` | Hoffnung | SYSTEM_EXT | Unknown third-party app. No user value. |
| `com.rlk.weathers` | RLK Weather Widget | PRODUCT | Third-party weather widget. |
| `com.talpa.hibrowser` | HiBrowser | TR_PRELOAD | Tecno browser. Chrome is already installed. |

### Google Apps (Reinstallable from Play Store)

| Package Name | App Name | Partition | Notes |
|---|---|---|---|
| `com.google.android.youtube` | YouTube | PRODUCT | Reinstallable from Play Store. |
| `com.google.android.gm` | Gmail | PRODUCT | Reinstallable from Play Store. |
| `com.google.android.apps.maps` | Google Maps | PRODUCT | Reinstallable from Play Store. |
| `com.google.android.apps.photos` | Google Photos | PRODUCT | Reinstallable from Play Store. |
| `com.google.android.calendar` | Google Calendar | TR_PRODUCT | Reinstallable from Play Store. |
| `com.google.android.apps.messaging` | Google Messages | TR_PRODUCT | Reinstallable. Tecno has its own SMS app. |
| `com.google.android.apps.docs` | Google Drive | PRODUCT | Reinstallable from Play Store. |
| `com.google.android.apps.wellbeing` | Digital Wellbeing | PRODUCT | Usage tracker. |
| `com.google.android.apps.tachyon` | Google Meet | PRODUCT | Reinstallable from Play Store. |
| `com.google.android.apps.googleassistant` | Google Assistant | TR_PRODUCT | Reinstallable. |
| `com.google.android.apps.restore` | Backup & Restore | PRODUCT | Only needed for Google backups. |
| `com.google.android.projection.gearhead` | Android Auto | PRODUCT | Only if using Android Auto. |
| `com.google.ar.core` | ARCore | PRODUCT | Only for AR apps. Reinstallable. |
| `com.google.android.apps.youtube.music` | YouTube Music | PRODUCT | Reinstallable. |
| `com.google.android.apps.safetyhub` | Safety Hub | PRODUCT | Redundant with Tecno safety features. |
| `com.google.android.apps.adm` | Find My Device | TR_PRODUCT | Reinstallable if needed. |
| `com.google.android.videos` | Google TV | PRODUCT | Already disabled. |

### Tecno Optional Apps & Features

| Package Name | App Name | Partition | Notes |
|---|---|---|---|
| `com.transsion.carlcare` | Carlcare Support | TR_PRELOAD | Tecno customer service. Remove after warranty. |
| `com.transsion.manualguide` | Manual Guide | TR_PRODUCT | Phone manual app. |
| `com.transsion.compass` | Compass | TR_PRODUCT | Remove if using Google Maps compass. |
| `com.transsion.fmradio` | FM Radio | SYSTEM_EXT | Needs wired headset as antenna. |
| `com.transsion.soundrecorder` | Sound Recorder | PRODUCT | Remove if using another recorder. |
| `com.transsion.mobilecloner` | Mobile Cloner | SYSTEM_EXT | Phone-to-phone transfer. Remove after setup. |
| `com.transsion.airtransfer` | Air Transfer | PRODUCT | Wireless file transfer. |
| `com.transsion.pcconnect` | PC Connect | SYSTEM_EXT | Tecno PC-link app. |
| `com.transsion.connectx.mirror.source` | ConnectX Mirror | PRODUCT | Screen mirroring. |
| `com.transsion.dualapp` | Dual App | PRODUCT | Run two accounts for same app. |
| `com.transsion.gamespace.app` | Game Space | PRODUCT | Gaming mode launcher. |
| `com.transsion.intercom` | Intercom | PRODUCT | Intercom feature. |
| `com.transsion.livecaption` | Live Caption | PRODUCT | Real-time subtitles. |
| `com.transsion.tranvoicecommand` | Voice Command | PRODUCT | Voice commands. |
| `com.transsion.notebook` | Notebook | TR_PRELOAD | Notes app. |
| `com.transsion.inearmonitor` | In-Ear Monitor | SYSTEM_EXT | For musicians. Remove if irrelevant. |
| `com.transsion.batterylab` | Battery Lab | SYSTEM_EXT | Battery diagnostic. |
| `com.transsion.repaircard` | Repair Card | SYSTEM_EXT | Hardware repair diagnostics. |
| `com.transsion.scanningrecharger` | Scanning Recharger | SYSTEM_EXT | Remove. |
| `com.transsion.smartpanel` | Smart Panel | SYSTEM_EXT_PRIV | Side edge panel launcher. |
| `com.transsion.chromecustomization` | Chrome Customization | TR_PRELOAD | Tecno Chrome skin. Use Chrome directly. |
| `com.transsion.ella` | Ella AI | SYSTEM_EXT | Tecno AI product. Remove if not using. |
| `com.transsion.magicfont` | Magic Font | SYSTEM_EXT | Download custom fonts. |
| `com.transsion.easypic` | Easy Pic | SYSTEM_EXT | Quick photo editing shortcuts. |
| `com.transsion.screenrecorder` | Screen Recorder | SYSTEM_EXT | Remove if using another. |
| `com.transsion.screencapture` | Screen Capture | SYSTEM_EXT | Android 15 has native screenshots. |
| `com.transsion.aiwriting` | AI Writing | SYSTEM_EXT | AI text suggestions. |
| `com.transsion.aiwallpaper` | AI Wallpaper | PRODUCT | AI-generated wallpapers. |
| `com.transsion.aichargeprovider` | AI Charge Provider | PRODUCT | AI charging optimization. |
| `com.transsion.multiwindow` | Multi Window | SYSTEM_EXT | Android 15 has native multi-window. |
| `com.transsion.spacesaversdk` | Space Saver SDK | PRODUCT | Storage optimization SDK. |
| `com.transsion.audiosmartconnect` | Audio Smart Connect | SYSTEM_EXT | Smart audio routing. |
| `com.sh.smart.caller` | Smart Caller | TR_PRODUCT | Caller ID lookup. Remove for privacy. |
| `com.transsion.livewallpaper.colorart` | LWP: ColorArt | PRODUCT | Live wallpaper pack. |
| `com.transsion.livewallpaper.fantasy` | LWP: Fantasy | PRODUCT | Live wallpaper pack. |
| `com.transsion.livewallpaper.magictouch` | LWP: MagicTouch | PRODUCT | Live wallpaper pack. |
| `com.transsion.livewallpaper.microsparkslim` | LWP: MicroSpark | PRODUCT | Live wallpaper pack. |
| `com.transsion.livewallpaper.mondrian` | LWP: Mondrian | PRODUCT | Live wallpaper pack. |
| `com.transsion.livewallpaper.pictorial` | LWP: Pictorial | PRODUCT | Live wallpaper pack. |
| `com.transsion.livewallpaper.page` | LWP: Page | PRODUCT | Live wallpaper pack. |
| `com.transsion.livewallpaper.theme` | LWP: Theme | PRODUCT | Live wallpaper pack. |
| `com.transsion.aicore.llm` | AI Core: LLM | PRODUCT | LLM engine. Large package. Remove if not using AI. |
| `com.transsion.aicore.cv` | AI Core: Computer Vision | PRODUCT | CV inference. Remove if not using AI camera. |
| `com.transsion.aicore.cv.matting` | AI Core: CV Matting | PRODUCT | Background removal AI. |
| `com.transsion.aicore.matting` | AI Core: Matting | PRODUCT | Duplicate matting engine. |
| `com.transsion.aicore.ocr` | AI Core: OCR | PRODUCT | Scan-to-text OCR engine. |
| `com.transsion.aicore.speech` | AI Core: Speech | PRODUCT | Speech AI engine. |
| `com.imaging.aiengine` | Imaging AI Engine | SYSTEM_EXT | Camera AI backend. |
| `com.android.egg` | Android Easter Egg | SYSTEM | Useless easter egg. |
| `com.android.dreams.basic` | Screensaver | SYSTEM | Screensaver. |
| `com.android.traceur` | System Tracer | SYSTEM | Developer performance tracing. |
| `com.android.DeviceAsWebcam` | Device as Webcam | SYSTEM_PRIV | USB webcam mode. |
| `com.android.bookmarkprovider` | Bookmark Provider | SYSTEM | Browser bookmark sync. |
| `com.android.providers.partnerbookmarks` | Partner Bookmarks | SYSTEM | OEM partner bookmarks. |
| `com.android.apps.tag` | NFC Tag Writer | SYSTEM_EXT_PRIV | Write NFC tags. |
| `com.android.bips` | BIPS Printing | SYSTEM_PRIV | Mopria print service. |
| `com.android.printspooler` | Print Spooler | SYSTEM | Print infrastructure. |
| `com.google.android.printservice.recommendation` | Print Service Rec. | SYSTEM_EXT | Suggests print apps. |
| `com.mediatek.batterywarning` | Battery Warning | SYSTEM_EXT | Android 15 has native battery warnings. |

---

## Ready-to-Run ADB Scripts

### Script 1: Disable Telemetry & Analytics (Run This First)

> NOTE: Zero risk. No reboot needed. Kills all analytics, ads, and data collection.

```bash
adb shell pm disable-user --user 0 com.transsion.statisticalsales
adb shell pm disable-user --user 0 com.transsion.personalizedService.hios
adb shell pm disable-user --user 0 com.transsion.uxdetector
adb shell pm disable-user --user 0 com.transsion.teop
adb shell pm disable-user --user 0 com.transsion.microintelligence
adb shell pm disable-user --user 0 com.transsion.magazineservice.hios
adb shell pm disable-user --user 0 com.transsion.tecnospot
adb shell pm disable-user --user 0 com.google.android.adservices.api
adb shell pm disable-user --user 0 com.google.android.ondevicepersonalization.services
adb shell pm disable-user --user 0 com.google.android.federatedcompute
adb shell pm disable-user --user 0 com.google.mainline.adservices
adb shell pm disable-user --user 0 com.google.mainline.telemetry
adb shell pm disable-user --user 0 com.google.android.gms.location.history
adb shell pm disable-user --user 0 com.mediatek.mdmconfig
adb shell pm disable-user --user 0 com.mediatek.mdmlsample
adb shell pm disable-user --user 0 com.mediatek.engineermode
adb shell pm disable-user --user 0 com.debug.loggerui
```

### Script 2: Remove Facebook Suite

```bash
adb shell pm uninstall --user 0 com.facebook.katana
adb shell pm uninstall --user 0 com.facebook.appmanager
adb shell pm uninstall --user 0 com.facebook.services
adb shell pm uninstall --user 0 com.facebook.system
adb shell pm uninstall --user 0 com.instagram.android
```

### Script 3: Remove Third-Party Bloat

```bash
adb shell pm uninstall --user 0 com.zhiliaoapp.musically
adb shell pm uninstall --user 0 cn.wps.moffice.lite.abroad.transsion
adb shell pm uninstall --user 0 net.bat.store
adb shell pm uninstall --user 0 com.transsnet.store
adb shell pm uninstall --user 0 com.driftsand.game.blockfunny
adb shell pm uninstall --user 0 com.zaz.translate
adb shell pm uninstall --user 0 com.smartlife.nebula
adb shell pm uninstall --user 0 com.idea.questionnaire
adb shell pm uninstall --user 0 com.hoffnung
adb shell pm uninstall --user 0 com.rlk.weathers
adb shell pm uninstall --user 0 com.talpa.hibrowser
adb shell pm uninstall --user 0 com.transsion.carlcare
adb shell pm uninstall --user 0 com.transsion.manualguide
adb shell pm uninstall --user 0 com.transsion.chromecustomization
```

### Script 4: Remove Google Optional Apps

> NOTE: All reinstallable from Play Store.

```bash
adb shell pm uninstall --user 0 com.google.android.youtube
adb shell pm uninstall --user 0 com.google.android.gm
adb shell pm uninstall --user 0 com.google.android.apps.maps
adb shell pm uninstall --user 0 com.google.android.apps.photos
adb shell pm uninstall --user 0 com.google.android.calendar
adb shell pm uninstall --user 0 com.google.android.apps.messaging
adb shell pm uninstall --user 0 com.google.android.apps.docs
adb shell pm uninstall --user 0 com.google.android.apps.wellbeing
adb shell pm uninstall --user 0 com.google.android.apps.tachyon
adb shell pm uninstall --user 0 com.google.android.apps.googleassistant
adb shell pm uninstall --user 0 com.google.android.apps.restore
adb shell pm uninstall --user 0 com.google.android.projection.gearhead
adb shell pm uninstall --user 0 com.google.android.apps.youtube.music
adb shell pm uninstall --user 0 com.google.android.apps.safetyhub
adb shell pm uninstall --user 0 com.google.android.videos
```

### Script 5: Remove AI Engine Bloat

> NOTE: ONLY run this if you are NOT using any HiOS AI camera/writing/voice features.

```bash
adb shell pm uninstall --user 0 com.transsion.aicore.llm
adb shell pm uninstall --user 0 com.transsion.aicore.cv
adb shell pm uninstall --user 0 com.transsion.aicore.cv.matting
adb shell pm uninstall --user 0 com.transsion.aicore.matting
adb shell pm uninstall --user 0 com.transsion.aicore.ocr
adb shell pm uninstall --user 0 com.transsion.aicore.speech
adb shell pm uninstall --user 0 com.imaging.aiengine
adb shell pm uninstall --user 0 com.transsion.aiwriting
adb shell pm uninstall --user 0 com.transsion.aiwallpaper
adb shell pm uninstall --user 0 com.transsion.ella
adb shell pm uninstall --user 0 com.transsion.aiassistantlifestyle
adb shell pm uninstall --user 0 com.transsion.kolun.assistant
```

### Script 6: Remove Live Wallpapers

```bash
adb shell pm uninstall --user 0 com.transsion.livewallpaper.colorart
adb shell pm uninstall --user 0 com.transsion.livewallpaper.fantasy
adb shell pm uninstall --user 0 com.transsion.livewallpaper.magictouch
adb shell pm uninstall --user 0 com.transsion.livewallpaper.microsparkslim
adb shell pm uninstall --user 0 com.transsion.livewallpaper.mondrian
adb shell pm uninstall --user 0 com.transsion.livewallpaper.pictorial
adb shell pm uninstall --user 0 com.transsion.livewallpaper.theme
```

### Script 7: Remove Unused Tecno Apps

```bash
adb shell pm uninstall --user 0 com.transsion.compass
adb shell pm uninstall --user 0 com.transsion.fmradio
adb shell pm uninstall --user 0 com.transsion.soundrecorder
adb shell pm uninstall --user 0 com.transsion.mobilecloner
adb shell pm uninstall --user 0 com.transsion.airtransfer
adb shell pm uninstall --user 0 com.transsion.pcconnect
adb shell pm uninstall --user 0 com.transsion.connectx.mirror.source
adb shell pm uninstall --user 0 com.transsion.gamespace.app
adb shell pm uninstall --user 0 com.transsion.intercom
adb shell pm uninstall --user 0 com.transsion.livecaption
adb shell pm uninstall --user 0 com.transsion.tranvoicecommand
adb shell pm uninstall --user 0 com.transsion.notebook
adb shell pm uninstall --user 0 com.transsion.batterylab
adb shell pm uninstall --user 0 com.transsion.repaircard
adb shell pm uninstall --user 0 com.transsion.scanningrecharger
adb shell pm uninstall --user 0 com.transsion.easypic
adb shell pm uninstall --user 0 com.transsion.magicfont
adb shell pm uninstall --user 0 com.transsion.dualapp
adb shell pm uninstall --user 0 com.transsion.smartpanel
adb shell pm uninstall --user 0 com.transsion.multiwindow
adb shell pm uninstall --user 0 com.transsion.spacesaversdk
adb shell pm uninstall --user 0 com.transsion.audiosmartconnect
adb shell pm uninstall --user 0 com.sh.smart.caller
adb shell pm uninstall --user 0 com.transsion.aichargeprovider
adb shell pm uninstall --user 0 com.android.egg
adb shell pm uninstall --user 0 com.android.dreams.basic
adb shell pm uninstall --user 0 com.android.traceur
adb shell pm uninstall --user 0 com.android.bookmarkprovider
adb shell pm uninstall --user 0 com.android.providers.partnerbookmarks
adb shell pm uninstall --user 0 com.android.apps.tag
adb shell pm uninstall --user 0 com.android.bips
adb shell pm uninstall --user 0 com.android.printspooler
adb shell pm uninstall --user 0 com.google.android.printservice.recommendation
adb shell pm uninstall --user 0 com.mediatek.batterywarning
```

---

## Emergency Recovery Commands

> **WARNING: If anything breaks after removal, run these BEFORE rebooting.**

```bash
# Restore any removed package
adb shell cmd package install-existing <package.name>

# Re-enable a disabled package
adb shell pm enable --user 0 <package.name>

# If launcher crashes (home screen gone)
adb shell cmd package install-existing com.transsion.hilauncher

# If SystemUI crashes (status bar gone)
adb shell pm enable --user 0 com.android.systemui

# Restore ALL disabled packages (nuclear undo)
adb shell pm list packages -d | sed 's/package://' | while read p; do adb shell pm enable --user 0 $p; done
```

---

## Partition Risk Reference

| Partition | Risk Level | Notes |
|---|---|---|
| `/apex/` | **NEVER TOUCH** | Google Mainline modules. Immutable. |
| `/vendor/` | **NEVER TOUCH** | Hardware driver overlays. |
| `/system/priv-app/` | EXTREME CAUTION | Core OS. Only known-safe removals. |
| `/system_ext/priv-app/` | HIGH RISK | OEM privileged apps. Research every package. |
| `/system_ext/app/` | MEDIUM RISK | OEM apps. Disable before remove. |
| `/product/` | LOW-MEDIUM | Pre-loaded product apps. Generally safer. |
| `/tr_product/` | LOW RISK | Tecno product-specific apps. |
| `/tr_preload/` | SAFE | Pre-loaded third-party apps. |
| `/data/app/` | SAFE | User-installed apps. |
