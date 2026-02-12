# OEG Frameworks SDK Developer Guide

Cross-platform mobile SDK supporting **Android**, **iOS**, and **Unity**, focused on **Authentication, User Management, and Payments**.

This SDK provides two integration paths:
1.  **Legacy Integration (Recommended for speed)**: Uses a built-in UI for Login, Profile, and Payments.
2.  **V2 API Integration (Advanced)**: Provides direct API access for building custom UIs.

---

## 1. Installation

### Android

#### Option 1: JitPack (Recommended)

1.  Add JitPack to your project's `settings.gradle` (or root `build.gradle`):
    ```kotlin
    dependencyResolutionManagement {
        repositories {
            google()
            mavenCentral()
            maven { url = uri("https://jitpack.io") }
        }
    }
    ```

2.  Add the dependency in your module's `build.gradle`:
    ```kotlin
    dependencies {
        implementation("com.github.thohd-oeg:android-sdk:1.0.19") // Check JitPack for latest version
    }
    ```

#### Option 2: Manual AAR

1.  Copy the `oeg-sdk-android.aar` to your project's `libs` folder.
2.  Add dependencies in `build.gradle`:

```kotlin
dependencies {
    implementation(files("libs/oeg-sdk-android.aar"))
    // ... (You must manually add all transitive dependencies listed in the POM)
}
```

### iOS

1.  Add the package via Swift Package Manager (SPM).
2.  Repository URL: `[Git Repo URL]` (or local path).
3.  Select `OEGFrameworks` library.

---

## 2. Configuration (`oeg_config.json`)

The SDK requires a configuration file to initialize.

**Location:**
*   **Android:** `src/main/assets/oeg_config.json`
*   **iOS:** Add to your App Bundle resources.

**Format:**

```json
{
    "core": {
        "game_id": 44,
        "api_base_url": "https://dev-api-sdk-game.oeg.vn/",
        "user_info_email_required": true,
        "notification_channel_id": "default",
        "debug_env": true
    },
    "sns": {
        "facebook_app_id": "YOUR_FB_APP_ID",
        "facebook_client_token": "YOUR_FB_CLIENT_TOKEN",
        "google_web_client_id": "YOUR_GOOGLE_WEB_CLIENT_ID"
    },
    "tracking": {
        "adjust_app_token": "YOUR_ADJUST_TOKEN",
        "adjust_environment": "sandbox"
    }
}
```

---

## 3. Integration Options

### Option A: Legacy Integration (Built-in UI)

Use this if you want a ready-made UI for login, profile management, and payments.

#### Android
```kotlin
// Initialize in Application or MainActivity
OegSdk.init(context)

// Show Login Screen
OegSdk.showLogin(this) { 
    // Callback when login screen closes
    // Check OEGAuth.currentUser for status
}

// Show User Profile / Update Info
OegSdk.showUpdateUserInfo(this)
```

#### iOS
```swift
// Initialize
OegSdkV2.shared.initialize()

// Show Login (SwiftUI / UIKit adapter required)
// ... (Refer to iOS Sample App)
```

### Option B: V2 API Integration (Custom UI)

Use this if you want to build your own UI and just use the SDK for logic.

#### Android
```kotlin
// 1. Initialize
val auth = OEGAuth.Builder(context).build()

// 2. Login
OEGAuth.login("username", "password") { result ->
    when (result) {
        is AuthResult.Success -> {
            println("Logged in: ${result.user.displayName}")
        }
        is AuthResult.Error -> {
            println("Login failed: ${result.message}")
        }
    }
}

// 3. Update Profile
OEGAuth.updateProfile(
    email = "new@email.com",
    displayName = "New Name"
) { result ->
    // Handle result
}

// 4. Change Password
OEGAuth.changePassword(
    oldPass = "current123",
    newPass = "new123"
) { result -> 
    // Handle result
}
```

---

## 4. API Reference

### Authentication (`OEGAuth`)

*   `login(username, password, callback)`
*   `register(username, password, email, ..., callback)`
*   `socialLogin(type, token, callback)`
*   `logout()`
*   `currentUser`: Returns the currently logged-in `AuthUser` object (or null).

### User Management

*   `updateProfile(data, callback)`: Updates user info (Email, Phone, DOB, etc.).
*   `changePassword(old, new, callback)`: Changes account password.
*   `verifyEmail(...)` / `verifyPhone(...)`: OTP verification flows.

### Payment (`OEGPayment`)

*   `purchase(productId, callback)`: Initiates IAP flow (Google Play / App Store).

---

## 5. Troubleshooting

**Common Issues:**

*   **404 on Update Profile:** Ensure you are using the latest SDK version. (Fixed issue with HTTP method).
*   **400 on Change Password:** Ensure `confirm_password` matches `new_password`.
*   **Missing Config:** The app will crash if `oeg_config.json` is missing or invalid. Check Logcat/Console for "Config not found".

## 6. Sample Apps

Check the `android/sample` and `ios/SampleProject` directories for full working examples of both Legacy and V2 integrations.
