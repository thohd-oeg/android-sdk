# OEG SDK V2 — Cross-Platform Game SDK

> **Authentication, User Management, Payments, Push Notifications, and Analytics**  
> Android (Kotlin) · iOS (Swift) · JS/TS Wrappers (Cocos Creator, Egret)

---

## Architecture

Two-Layer Clean Architecture (SOLID) — see [architecture.md](.agent/context/architecture.md) for details.

```
┌─────────────────────────────────────────────┐
│  UI SDK (Layer 2 - Pre-built Screens)       │
│  - Views (Activities, SwiftUI, Fragments)   │
│  - ViewModels (StateFlow / @Published)      │
│  - ViewModels inject UseCases, NEVER repos  │
├─────────────────────────────────────────────┤
│  Core SDK (Layer 1 - Headless Logic)        │
│  - Domain (UseCases, Interfaces, Models)    │
│  - Data (Repositories, API, Storage)        │
│  - Public Facades (OEGAuth, OEGPayment)     │
└─────────────────────────────────────────────┘
```

---

## Quick Start

### Android

```kotlin
// 1. Add JitPack repository
// settings.gradle.kts
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        maven { url = uri("https://jitpack.io") }
    }
}

// 2. Add dependency
// build.gradle.kts
dependencies {
    implementation("com.github.thohd-oeg:android-sdk:TAG")
}
```

### iOS

```swift
// Swift Package Manager
// Package URL: [Git Repo URL]
// Library: OegSdkV2
```

---

## Configuration (`oeg_config.json`)

Place in `src/main/assets/` (Android) or App Bundle (iOS).

```json
{
    "core": {
        "game_id": 44,
        "api_base_url": "https://dev-api-sdk-game.oeg.vn/",
        "user_info_fullname_required": false,
        "user_info_birthday_required": false,
        "user_info_email_required": true,
        "user_info_phone_required": false,
        "notification_channel_id": "default",
        "notification_title": "Notification",
        "debug_env": true
    },
    "sns": {
        "facebook_app_id": "YOUR_FB_APP_ID",
        "facebook_client_token": "YOUR_FB_CLIENT_TOKEN",
        "google_web_client_id": "YOUR_GOOGLE_WEB_CLIENT_ID",
        "tiktok_client_key": "YOUR_TIKTOK_CLIENT_KEY"
    },
    "tracking": {
        "adjust_app_token": "YOUR_ADJUST_TOKEN",
        "adjust_environment": "sandbox"
    }
}
```

---

## Integration

### Option A: Built-in UI (Unified Dashboard & Auth)

The SDK provides a complete, out-of-the-box Unified Dashboard and Authentication flow.

```kotlin
// Android
OegSdk.init(context)
OegSdk.showLogin(activity) { /* login complete */ }
// Opens the Unified Dashboard (Profile, Language, Change Password, IAP/Push Testing)
OegSdk.showDashboard(activity)
```

```swift
// iOS
OegSdkV2.shared.initialize()
OEGManager.shared.showLogin(from: viewController)
// Opens the Unified Dashboard
OEGManager.shared.showDashboard(from: viewController)
```

The Unified Dashboard features:
- **User Profile:** View and update Email, Phone, and Fullname.
- **Account Management:** Change Password.
- **Localization:** Multi-language Selection (English, Tiếng Việt, 简体中文).
- **Service Testing:** Stubs for testing In-App Purchases (IAP) and Push Notifications.
- **Danger Zone:** Account Deletion & Logout.

### Option B: V2 API (Custom UI)

```kotlin
// Android — callback-based
OEGAuth.Builder(context).build()

OEGAuth.login("username", "password") { result ->
    when (result) {
        is AuthResult.Success -> println("Token: ${result.user.token}")
        is AuthResult.Error -> println("Error: ${result.message}")
    }
}

OEGAuth.updateProfile(email = "new@email.com") { result -> /* ... */ }
OEGAuth.fetchUserInfo { result -> /* ... */ }
OEGAuth.changePassword("old", "new") { result -> /* ... */ }
OEGAuth.logout()
```

```swift
// iOS — callback-based
OEGAuth.shared.initialize(baseURL: url, gameId: 44, packageName: bundle)

OEGAuth.shared.login(username: "user", password: "pass") { user, error in
    if let user = user { print("Logged in: \(user.username ?? "")") }
}
```

---

## Public API Surface

### `OEGAuth` — Authentication & Profile

| Method | Description |
|--------|-------------|
| `login(username, password, callback)` | Username/password login |
| `register(fullname, username, password, callback)` | Create new account |
| `playNow(callback)` | Guest login (Deprecated) |
| `playNow2(callback)` | Guest login with persistent SDK ID |
| `socialLogin(provider, accessToken, authorizationCode?, callback)` | Google/Facebook/Apple/TikTok |
| `merge(uuid, username, password, ..., callback)` | Link guest → full account |
| `mergeSocial(uuid, provider, accessToken, callback)` | Link guest → social |
| `forgotPassword(email, callback)` | Send password reset email |
| `changePassword(currentPassword, newPassword, confirmPassword, callback)` | Change password |
| `updateProfile(..., callback)` | Update user profile fields (PUT) |
| `fetchUserInfo(callback)` | Get fresh user data from API |
| `checkEmailVerification(callback)` | Check email verification status |
| `requestEmailOtp(callback)` | Send email verification OTP |
| `requestPhoneOtp(callback)` | Send phone verification OTP |
| `verifyOtp(otp, callback)` | Verify OTP code |
| `logout()` | Clear session + social state |

**Properties:** `currentUser`, `token`, `isLoggedIn`, `getConfig()`

### `OEGPayment` — In-App Purchases

| Method | Description |
|--------|-------------|
| `purchaseAndVerify(productId, gameData, callback)` | Purchase + server verify |
| `isIAPEnabled(level, callback)` | Level gate check |

### `OEGPush` — Push Notifications

| Method | Description |
|--------|-------------|
| `getDeviceToken()` | Get FCM/APNs token |
| `subscribe(topic)` / `unsubscribe(topic)` | Topic messaging |

### `OEGAnalytics` — Adjust Wrapper

| Method | Description |
|--------|-------------|
| `logEvent(eventToken)` | Track custom event |
| `logPurchase(revenue, currency, eventToken)` | Track IAP |

---

## Project Structure

```
Workspace/
├── android/
│   ├── sdk/          # Android SDK (vn.oeg.sdk.v2)
│   └── sample/       # Demo app
├── ios/
│   ├── Sources/OegSdkV2/   # iOS SDK (SPM)
│   ├── TestApp/             # Demo app
│   └── fastlane/            # CI/CD lanes
├── api-docs/         # Postman API collection
├── .agent/           # Agent context, workflows, skills
└── .github/workflows/ # CI/CD pipelines
```

## Build Commands

| Task | Command |
|------|---------|
| Build Android | `cd android && .\gradlew.bat :sdk:assembleDebug` |
| Test Android | `cd android && .\gradlew.bat :sdk:testDebugUnitTest` |
| Build iOS | `cd ios && swift build` |
| Test iOS | `cd ios && bundle exec fastlane test` |

## Troubleshooting

- **Missing config:** App crashes if `oeg_config.json` is missing — check Logcat/Console
- **404 on profile update:** Ensure latest SDK version (HTTP method fix applied)
- **400 on change password:** `confirm_password` must match `new_password`
- **Social login fails:** Verify `facebook_app_id` and `google_web_client_id` in config

---

*Last updated: 2026-02-26*
