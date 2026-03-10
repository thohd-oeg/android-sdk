
# Android SDK Installation

Follow these steps to import the OEG SDK into your Android Studio project.

## 1. Add Dependencies

Add the JitPack repository to your root `settings.gradle.kts` (or `settings.gradle`):

```kotlin title="settings.gradle.kts"
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        maven { url = uri("https://jitpack.io") }
        // Needed for TikTok SDK
        maven { url = uri("https://artifact.bytedance.com/repository/AwemeOpenSDK") }
    }
}
```

Next, add the SDK to your app-level `build.gradle.kts` (or `build.gradle`):

```kotlin title="app/build.gradle.kts"
dependencies {
    // Core OEG SDK
    implementation("com.github.thohd-oeg:android-sdk:1.0.41")
    
    // (Optional) Adjust analytics if required by your marketing team
    implementation("com.adjust.sdk:adjust-android:5.0.0")
    // (Optional) Firebase BoM for Push Notifications
    implementation(platform("com.google.firebase:firebase-bom:32.7.0"))
    implementation("com.google.firebase:firebase-messaging-ktx")
}
```

## 2. Configure AndroidManifest.xml

The SDK requires standard permissions to communicate with the network and make payments.

```xml title="AndroidManifest.xml"
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="com.android.vending.BILLING" />
</manifest>
```

> **Note**: In V2, the SDK no longer reads `OEG_GAME_ID` or `OEG_CLIENT_KEY` from metadata. Use `oeg_config.json` instead.

## 3. Configuration (`oeg_config.json`)

The OEG SDK V2 uses a single JSON configuration file. Create a file named `oeg_config.json` in your Android `src/main/assets/` folder:

```json title="src/main/assets/oeg_config.json"
{
    "core": {
        "game_id": {YOUR_GAME_ID},
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

## 4. Initialization

There are two ways to initialize the SDK: using the drop-in UI Facade (Option A), or manually (Option B). Call this in your main `Activity`'s `onCreate`.

### Option A: Built-in Facade (Recommended)

If you plan to use `OegSdk.showLogin()` and `OegSdk.showDashboard()`, initialize via the `OegSdk` legacy facade. This will automatically read your `oeg_config.json`.

```kotlin title="MainActivity.kt"
import vn.oeg.sdk.v2.legacy.OegSdk

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        val initResult = OegSdk.init(this)
        if (initResult.isSuccess()) {
            Log.d("OEG", "SDK Initialized from oeg_config.json")
        }
    }
}
```

### Option B: Manual API

If you are building your own UI entirely and only want the core headless logic:

```kotlin title="MainActivity.kt"
import vn.oeg.sdk.v2.auth.OEGAuth

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        OEGAuth.Builder(applicationContext)
            // You can optionally pass manual values or rely on oeg_config.json
            .build()
            
        Log.d("OEG", "SDK Core Initialized")
    }
}
```

# Android Authentication

The OEG SDK provides built-in UI and logical flows for multiple authentication methods to maximize player conversion.

## 1. Play Now (Guest Login)

This is a frictionless login flow that creates a persistent Guest account bounded to the device.

```kotlin title="Calling Play Now"
import vn.oeg.sdk.v2.auth.OEGAuth
import vn.oeg.sdk.v2.core.auth.domain.AuthResult

// Triggers the Play Now API and returns the user token
OEGAuth.playNow { result ->
    when (result) {
        is AuthResult.Success -> {
            Log.d("OEG", "Guest Logged In: ${result.user.userId}")
        }
        is AuthResult.Error -> {
            Log.e("OEG", "Error: ${result.message}")
        }
    }
}
```

> **Note:** The `playNow()` API now automatically creates and caches a persistent `sdk_id` in SharedPreferences. This ensures the guest account is not lost simply by logging out.

## 2. Standard Login & Social Login

For users who want to link data across devices, trigger the unified OEG System Dialog.

```kotlin title="Open Login Dialog"
// Instead of a custom login dialog, you can trigger standard logins via API
// or present the Unified Dashboard if you want a built-in UI:
// OegSdk.showDashboard(activity) (from Option A facade)

OEGAuth.login("username", "password") { result ->
    when (result) {
        is AuthResult.Success -> println("Token: ${result.user.token}")
        is AuthResult.Error -> println("Error: ${result.message}")
    }
}
```

### Social Login Requirements

If you want to support Facebook, Google, or TikTok in the dialog, ensure:
1. Valid credentials are provided in the `sns` block of your `oeg_config.json`.
2. For Google: `google-services.json` must be present in your Android `app/` folder, and you must add `implementation("com.google.android.libraries.identity.googleid:googleid:1.1.0")` to your app's dependencies.
3. For Facebook: Even though the App ID and Token are passed dynamically via `oeg_config.json`, the Meta SDK **strictly requires** the following entries in your `AndroidManifest.xml` for fallback mechanics like Chrome Custom Tabs to work properly:

```xml title="AndroidManifest.xml"
<application>
    <meta-data android:name="com.facebook.sdk.ApplicationId" android:value="@string/facebook_app_id"/>
    <meta-data android:name="com.facebook.sdk.ClientToken" android:value="@string/facebook_client_token"/>
    
    <provider
        android:name="com.facebook.FacebookContentProvider"
        android:authorities="com.facebook.app.FacebookContentProvider{YOUR_FB_APP_ID}"
        android:exported="true" />
        
    <!-- Facebook Login Activity (required for OAuth dialog) -->
    <activity
        android:name="com.facebook.FacebookActivity"
        android:configChanges="keyboard|keyboardHidden|screenLayout|screenSize|orientation"
        android:label="@string/app_name" />
        
    <!-- Facebook Custom Tab redirect (handles fb{YOUR_FB_APP_ID}:// scheme) -->
    <activity
        android:name="com.facebook.CustomTabActivity"
        android:exported="true">
        <intent-filter>
            <action android:name="android.intent.action.VIEW" />
            <category android:name="android.intent.category.DEFAULT" />
            <category android:name="android.intent.category.BROWSABLE" />
            <data android:scheme="fb{YOUR_FB_APP_ID}" />
        </intent-filter>
    </activity>
</application>
```
*(Make sure to replace `{YOUR_FB_APP_ID}` securely in the `authorities` attribute, leaving the prefix intact)*

4. For TikTok: Ensure the Aweme repository is in your `settings.gradle` and the provider is set up.

## 3. Persistent Session Check

Always check if the user is already logged in when the game starts to bypass the login screen.

```kotlin title="MainActivity.kt"
if (OEGAuth.isLoggedIn) {
    // User already logged in, enter game
    val token = OEGAuth.token
} else {
    // Show login UI
}
```

## 4. Other Auth APIs

The `OEGAuth` facade provides many other APIs for building your own UI:
- `OEGAuth.fetchUserInfo { result -> ... }`: Retrieves the freshest user info from the server.
- `OEGAuth.forgotPassword(email) { result -> ... }`: Sends a password reset email.
- `OEGAuth.requestEmailOtp() { result -> ... }`: Requests an OTP for verifying the current email.

## 5. Logout

Call `OEGAuth.logout()` when the user intends to sign out. This clears the active session token and social login state, but preserves the guest `sdk_id` locally.

```kotlin
OEGAuth.logout()
// Return UI to login screen
```

# Unified Dashboard

The Android SDK includes a pre-built Unified Dashboard via `OEGDashboardFragment`. This dashboard provides a centralized hub for all account management activities, reducing the amount of custom UI you need to build.

## Features

The Android Unified Dashboard offers the following features:

- **User Profile Info**: Displays the current user's Username, User ID, and Email. Includes a Guest badge for `playNow` (Guest) accounts.
- **Update Information**: Allows users to update their Email, Phone Number, and Fullname via the `OEGUserProfileActivity`. *(Hidden for Guest accounts)*
- **Change Password**: Dedicated screen for updating passwords. *(Hidden for Guest accounts)*
- **Change Language**: Built-in dialog to toggle the SDK's language (English, Tiếng Việt, 简体中文).
- **Developer Test Services**: Built-in stubs for testing IAP configuration and Push Notifications directly from the SDK.
- **Danger Zone**: Delete Account and Logout functionality.

## Displaying the Dashboard

You can surface the Unified Dashboard to your users using the Legacy Facade (Option A). If your app relies on the `OegSdk` wrapper, you can call:

```kotlin
// Assuming 'activity' is your current Activity context
OegSdk.showDashboard(activity)
```

Alternatively, if you are integrating views manually, you can launch the `OEGUserProfileActivity` or host the `OEGDashboardFragment` within your own activity stack.

## Customization and Callbacks

The Unified Dashboard relies on the `ProfileViewModel` and `OEGAuthCore`. Changes made within the dashboard (like updating the profile or logging out) update the central session. Ensure that you have registered your global Authentication listeners to react when a user logs out from this dashboard.

# Android IAP & Analytics

The OEG SDK provides robust wrappers around Google Play Billing and industry-standard marketing tools (Adjust/AppsFlyer).

## 1. In-App Purchase (IAP)

The `IabViewModel` and `BillingManager` handle connecting to Google Play, fetching product details, launching the purchase flow, and communicating with the OEG Backend for Server-to-Server receipt verification.

```kotlin title="Triggering a Purchase"
import vn.oeg.sdk.v2.payment.OEGPayment
import vn.oeg.sdk.v2.core.payment.domain.pojo.GameData

// Construct GameData with details needed for server-to-server delivery
val gameData = GameData(
    userId = currentUser.id, // OEG User ID
    serverId = 101,          // Game Server
    roleId = 5001,           // Character ID
    roleIdString = "mage_5001",
    level = 25,
    extInfo = "payload_data" // Optional game-specific context
)

val productId = "com.oeg.game.pack1"

OEGPayment.purchaseAndVerify(
    activity = this, 
    productId = productId, 
    gameData = gameData
) { success ->
    if (success) {
        // Validation with OEG Server successful.
        // Google Play Receipt consumed.
        Log.d("OEG", "Item Delivered!")
    } else {
        // Purchase failed or user cancelled
        Log.e("OEG", "Purchase Failed")
    }
}
```

### Server Verification (Mandatory)
For security, the OEG SDK handles verifying the `purchaseToken` with the OEG Server *before* returning `onSuccess()`. You do not need to verify the receipt yourself. Your game server should listen to Webhook callbacks from the OEG Server to deliver in-game items (KNB).

### Level Gating APIs
You can gate purchases by level requirements configured remotely via the `OEGPayment.isIAPEnabled()` API.

```kotlin title="Checking if IAP is Enabled"
OEGPayment.isIAPEnabled(level = currentLevel) { isEnabled ->
    if (isEnabled) {
        // Show payment button
    } else {
        // Hide payment button or show "Reach level 10 to unlock shop"
    }
}
```

## 2. Event Tracking (Adjust & AppsFlyer)

Tracking installs, retention, and custom events is essential for your Marketing User Acquisition (UA) teams.

### Standard Events

Basic events (Install, App Open, Login, Purchase) are tracked **automatically** by the SDK. You do not need to write extra code for these.

### Custom Events Log

To track custom events (Tutorial Complete, Level Up, First Boss Defeated), use the dedicated API:

```kotlin
import vn.oeg.sdk.v2.core.analytics.OEGAnalytics

// Example: Tracking an event using the Adjust Event Token from the dashboard
val levelUpEventToken = "g3m9bk" 

// You can pass an optional map of multiple parameters to enrich your data
val eventData = mapOf(
    "current_level" to "5",
    "score" to "12050",
    "weapon_equipped" to "sword"
)

OEGAnalytics.logEvent(levelUpEventToken, eventData)
```

## 3. Firebase Push Notifications (FCM)

The SDK utilizes Firebase Cloud Messaging to send localized push notifications and marketing payloads.

To enable, place your `google-services.json` file in the Android `app/` folder. The SDK will automatically retrieve the FCM Device Token.

### Receiving Marketing Data (Active Game)
Currently, automatic backend token sync is **disabled** to allow games to manage their own push notification delivery logic directly with Firebase. However, if you are utilizing the SDK's generic `OEGNotificationService` for pushes sent to the app, you can intercept the JSON payload payload provided by marketing directly in your game code via the listener.

```kotlin title="Listening for Notification Data"
import vn.oeg.sdk.v2.push.OEGPush

OEGPush.setPushListener(object : OEGPush.PushListener {
    override fun onMessageReceived(data: Map<String, String>) {
        Log.d("OEG", "Received push payload: ${data["custom_key"]}")
        // e.g. open special event UI or grant rewards
    }
})
```
