# OEG Frameworks SDK Developer Guide

Cross-platform mobile SDK for **Android**, **iOS**, and **Unity**, providing **Authentication**, **User Management**, **Payments**, and **Push Notifications**.

## 🚀 Features

*   **Authentication**: Login, Register, Social Login (Google, Facebook), Guest Play.
*   **User Management**: Profile Update, Change Password, Account Merging.
*   **Payments**: In-App Purchases (Google Play / App Store) with server-side verification.
*   **Push Notifications**: Firebase Cloud Messaging (FCM) integration.
*   **Configuration**: Remote Config support with local caching.
*   **Security**: Encrypted storage for sensitive data.

---

## 1. Installation

### Android

1.  Copy the `sdk-release.aar` to your `libs/` folder.
2.  Add dependencies in your module's `build.gradle.kts`:

```kotlin
dependencies {
    implementation(files("libs/sdk-release.aar"))
    
    // Core Dependencies
    implementation("com.google.code.gson:gson:2.10.1")
    implementation("com.squareup.retrofit2:retrofit:2.9.0")
    implementation("com.squareup.retrofit2:converter-gson:2.9.0")
    implementation("com.squareup.okhttp3:logging-interceptor:4.12.0")
    implementation("androidx.security:security-crypto:1.1.0-alpha06")
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.7.3")

    // Social Login (Google & Facebook)
    implementation("androidx.credentials:credentials:1.3.0")
    implementation("androidx.credentials:credentials-play-services-auth:1.3.0")
    implementation("com.google.android.libraries.identity.googleid:googleid:1.1.0")
    implementation("com.facebook.android:facebook-login:16.0.0")

    // Payments & Push
    implementation("com.android.billingclient:billing-ktx:6.0.0")
    implementation(platform("com.google.firebase:firebase-bom:32.7.0"))
    implementation("com.google.firebase:firebase-messaging-ktx")
}
```

---

## 2. Configuration (`assets/oeg_config.json`)

Create `src/main/assets/oeg_config.json` with your game's configuration:

```json
{
    "core": {
        "game_id": 14,
        "api_base_url": "https://api-sdk-game.oeg.vn/",
        "notification_channel_id": "default_channel"
    },
    "sns": {
        "google_web_client_id": "YOUR_WEB_CLIENT_ID.apps.googleusercontent.com",
        "facebook_app_id": "YOUR_FB_APP_ID",
        "facebook_client_token": "YOUR_FB_CLIENT_TOKEN"
    }
}
```

---

## 3. Usage (Android V2)

### Initialization

Initialize the SDK in your `Application` class or `MainActivity`:

```kotlin
OEGAuth.Builder(context)
    .gameId(14)
    .packageName("com.your.game.package")
    .build()

// Initialize Payment System
OEGPayment.initialize(context)
```

### Authentication

**Login:**
```kotlin
OEGAuth.login("username", "password") { result ->
    when (result) {
        is AuthResult.Success -> {
            val token = result.user.token
            println("Logged in: ${result.user.displayName}")
        }
        is AuthResult.Error -> println("Login failed: ${result.message}")
    }
}
```

**Register:**
```kotlin
OEGAuth.register("Full Name", "username", "password") { result ->
    // Handle result
}
```

**Play Now (Guest):**
```kotlin
OEGAuth.playNow { result ->
    // Handle guest login
}
```

**Social Login (Google/Facebook):**
```kotlin
// Get token/accessToken from Google/Facebook SDKs, then:
OEGAuth.socialLogin(SocialProvider.GOOGLE, idToken) { result ->
    // Handle social login
}
```

**Logout:**
```kotlin
OEGAuth.logout()
```

### User Management

**Update Profile:**
```kotlin
OEGAuth.updateProfile(
    email = "new@email.com",
    birthday = "1990-01-01",
    displayName = "New Name"
) { result ->
    // Handle result
}
```

**Change Password:**
```kotlin
OEGAuth.changePassword(oldPass, newPass) { result ->
    // Handle result
}
```

**Check Verification Status:**
```kotlin
if (OEGAuth.needsVerification()) {
    // Prompt user to verify email/phone
}
```

### Payments (IAP)

**Purchase a Product:**
```kotlin
OEGPayment.purchase(activity, productDetails) { result ->
    when (result) {
        is PurchaseResult.Success -> {
            println("Purchase successful: ${result.purchase.orderId}")
            // Optional: Consume if it's a consumable item
            OEGPayment.consumePurchase(result.purchase)
        }
        is PurchaseResult.Cancelled -> println("User cancelled")
        is PurchaseResult.Error -> println("Error: ${result.message}")
    }
}
```

### Push Notifications

**Get Device Token:**
```kotlin
lifecycleScope.launch {
    val token = OEGPush.getDeviceToken()
    // Send to your backend if needed
}
```

---

## 4. API Reference

### `OEGAuth`
| Method | Description |
|--------|-------------|
| `login`, `register`, `playNow` | Basic authentication methods. |
| `socialLogin` | Authenticate using Google or Facebook tokens. |
| `logout` | Clears local session and cancels pending tasks. |
| `updateProfile` | Updates user attributes (email, phone, etc.). |
| `changePassword` | Updates account password. |
| `merge`, `mergeSocial` | Merges a guest account into a registered/social account. |
| `isEmailVerified`, `isPhoneVerified` | Checks verification status. |
| `getCurrentUser`, `getToken`, `isLoggedIn` | Session state helpers. |

### `OEGPayment`
| Method | Description |
|--------|-------------|
| `initialize` | Sets up Google Billing Client. |
| `purchase` | Starts the purchase flow. |
| `queryProducts` | Fetches product details from Google Play. |
| `consumePurchase` | Consumes a purchase (makes it available to buy again). |

### `OEGPush`
| Method | Description |
|--------|-------------|
| `getDeviceToken` | Returns the FCM device token. |
| `subscribeToTopic` | Subscribes the device to an FCM topic. |
