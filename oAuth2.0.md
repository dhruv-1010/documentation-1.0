# Authentication Module - React Native Google & Apple Sign-In

[![React Native](https://img.shields.io/badge/React%20Native-0.75.4-blue.svg)](https://reactnative.dev/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0-blue.svg)](https://www.typescriptlang.org/)
[![Platform](https://img.shields.io/badge/Platform-iOS%20%7C%20Android-green.svg)](https://reactnative.dev/)

A comprehensive authentication module for React Native applications that provides Google Sign-In and Apple Sign-In functionality with proper OAuth 2.0 handling for both iOS and Android platforms.

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Platform Setup](#platform-setup)
- [Usage](#usage)
- [Troubleshooting](#troubleshooting)
- [Common Issues](#common-issues)
- [Best Practices](#best-practices)
- [Security Considerations](#security-considerations)

## 🎯 Overview

This module provides a unified interface for implementing Google Sign-In and Apple Sign-In in React Native applications. It handles OAuth 2.0 authentication flows with proper error handling and configuration management. The module is designed to work seamlessly with Firebase, as our app primarily handles Google Cloud services through Firebase.

## ✨ Features

- **Google Sign-In**: Complete Google authentication with proper token handling
- **Apple Sign-In**: Native Apple authentication for iOS devices
- **Cross-Platform**: Works on both iOS and Android
- **TypeScript Support**: Full TypeScript support with proper type definitions
- **Error Handling**: Comprehensive error handling and logging
- **Firebase Integration**: Designed to work with Firebase for Google Cloud services
- **Localization**: Support for multiple languages
- **Security**: Secure token management and validation

## 🔧 Prerequisites

Before starting, ensure you have:

- React Native project set up (version 0.75.4+)
- Google Cloud Console access
- Firebase project configured
- Apple Developer account (for iOS)
- Physical device for testing (Apple Sign-In doesn't work on simulator)

## 📦 Installation

### 1. Install Dependencies

```bash
# Install authentication libraries
yarn add @invertase/react-native-apple-authentication @react-native-google-signin/google-signin jwt-decode react-native-app-auth

# Install iOS pods
cd ios && pod install && cd ..
```

### 2. Required Dependencies

```json
{
  "@invertase/react-native-apple-authentication": "^2.4.1",
  "@react-native-google-signin/google-signin": "^15.0.0",
  "jwt-decode": "^4.0.0",
  "react-native-app-auth": "^8.0.3"
}
```

## ⚙️ Configuration

### 1. Google Cloud Console Setup

**Important**: Our app primarily handles Google Cloud services through Firebase. Follow these steps:

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select existing one
3. Enable Google Sign-In API
4. Create OAuth 2.0 credentials:
   - **Web application** (for Android) - This will be your Web OAuth Client ID
   - **iOS application** (for iOS) - This will be your iOS Client ID
5. **Note down the client IDs** - You'll need these for configuration

### 2. Firebase Project Setup

Since our app primarily handles Google Cloud through Firebase:

1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Create a new project or select existing one
3. Add your Android and iOS apps to the Firebase project
4. Download configuration files:
   - `google-services.json` for Android
   - `GoogleService-Info.plist` for iOS

### 3. SHA-1 Key Management

**Critical**: You must generate SHA-1 keys and add them to Firebase project settings:

```bash
# Generate debug SHA-1 (for development)
keytool -list -v -keystore ~/.android/debug.keystore -alias androiddebugkey -storepass android -keypass android

# Generate release SHA-1 (for production)
keytool -list -v -keystore your-release-key.keystore -alias your-key-alias

# Another method Generate local build SHA-1 for local development (useful for multiple version)
cd android 
./gradlew signingreport

```

**Steps to add SHA-1 keys to Firebase**:
1. Copy the SHA-1 fingerprint from the keytool output
2. Go to Firebase Console → Project Settings → Your Apps
3. Find your Android app
4. Add the SHA-1 fingerprint to the "SHA certificate fingerprints" section
5. Repeat for all environments (debug, release, internal testing)

### 4. Update OAuth Configuration

Update `src-v2/modules/auth/oauthConfig.ts`:

```typescript
export const oauthConfigs: OAuthConfigs = {
    google: {
        webClientId: 'YOUR_WEB_CLIENT_ID_HERE', // From Google Cloud Console
        iosClientId: 'YOUR_IOS_CLIENT_ID_HERE',  // From Google Cloud Console
        offlineAccess: true,
        forceCodeForRefreshToken: true,
    },
    apple: {
        scopes: ['EMAIL', 'FULL_NAME'],
    },
};
```

## 🔧 Platform Setup

### Android Configuration

#### 1. Build.gradle Configuration

Add to `android/app/build.gradle`:

```gradle
android {
    defaultConfig {
        // ... existing config
        manifestPlaceholders["appAuthRedirectScheme"] = applicationId
    }
}
```

#### 2. Google Services File

- Place `google-services.json` in `android/app/`
- Ensure your package name matches the one in Firebase Console

#### 3. OAuth Client ID Requirements

**Important**: For Android, you must use the **Web OAuth Client ID** from Google Cloud Console, not the Android client ID.

### iOS Configuration

#### 1. Info.plist Configuration

Add to `ios/YourApp/Info.plist`:

```xml
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleURLName</key>
        <string>GoogleSignIn</string>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>com.googleusercontent.apps.YOUR_CLIENT_ID</string>
        </array>
    </dict>
</array>
```

#### 2. Entitlements Configuration

Add to `ios/YourApp/YourApp.entitlements`:

```xml
<key>com.apple.developer.applesignin</key>
<array>
    <string>Default</string>
</array>
```

#### 3. Google Services File

- Add `GoogleService-Info.plist` to your iOS project in Xcode
- Ensure your bundle ID matches the one in Firebase Console

## 💻 Usage

### Basic Implementation

```typescript
import { oauthService } from './src-v2/modules/auth/oauth';

// Google Sign-In
const handleGoogleSignIn = async () => {
    try {
        const result = await oauthService.signInWithGoogle();
        console.log('Google Sign-In successful:', result);
        
        // Handle the result
        const { email, name, idToken } = result;
        
        // Send to your backend
        const response = await fetch('/api/auth/google', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ idToken, email, name })
        });
        
    } catch (error) {
        console.error('Google Sign-In failed:', error);
    }
};

// Apple Sign-In
const handleAppleSignIn = async () => {
    try {
        const result = await oauthService.signInWithApple();
        console.log('Apple Sign-In successful:', result);
        
        // Handle the result
        const { email, name, idToken } = result;
        
        // Send to your backend
        const response = await fetch('/api/auth/apple', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ idToken, email, name })
        });
        
    } catch (error) {
        console.error('Apple Sign-In failed:', error);
    }
};

// Sign Out
const handleSignOut = async () => {
    try {
        await oauthService.signOut();
        console.log('Sign out successful');
    } catch (error) {
        console.error('Sign out failed:', error);
    }
};
```

### Advanced Configuration

```typescript
// Custom Google Sign-In with force consent
const handleGoogleSignInWithConsent = async () => {
    try {
        const result = await oauthService.signInWithGoogle({ forceConsent: true });
        // Handle result
    } catch (error) {
        // Handle error
    }
};
```

## 🚨 Troubleshooting

### Most Common Error: DEVELOPER_ERROR

The `DEVELOPER_ERROR` is the most common authentication error. It occurs when Google Sign-In cannot verify your app's identity.

#### Root Causes & Solutions

**1. Missing SHA-1 Keys**
- **Symptoms**: `DEVELOPER_ERROR` on Android, authentication fails immediately
- **Solution**: Add SHA-1 keys to Firebase project settings (not Google Cloud Console)

**2. Wrong OAuth Client ID**
- **Symptoms**: `DEVELOPER_ERROR` persists even after adding SHA-1 keys
- **Solution**: Use Web OAuth Client ID for Android, iOS Client ID for iOS

**3. Package Name Mismatch**
- **Symptoms**: Authentication works in debug but fails in release
- **Solution**: Ensure package name matches Firebase project settings

**4. Firebase Configuration**
- **Symptoms**: Authentication fails across all environments
- **Solution**: Verify `google-services.json` and `GoogleService-Info.plist` are correctly placed

### Other Common Errors

**Network Error**
- Check internet connectivity
- Verify Google Play Services are up to date (Android)
- Try on different network (WiFi vs mobile data)

**Apple Sign-In Not Available**
- Test on real device (not simulator)
- Ensure iOS 13+ is installed
- Check Apple Sign-In capability is enabled in entitlements

**Configuration Errors**
- Verify `oauthConfigs` has correct client IDs
- Check configuration files are properly placed
- Ensure build.gradle has correct `appAuthRedirectScheme`

## 🔍 Debugging Steps

### Step 1: Enable Verbose Logging

```typescript
const handleGoogleSignIn = async () => {
    try {
        console.log('[DEBUG] Starting Google Sign-In...');
        const result = await oauthService.signInWithGoogle();
        console.log('[DEBUG] Success:', result);
    } catch (error) {
        console.error('[DEBUG] Error details:', {
            message: error.message,
            code: error.code,
            stack: error.stack
        });
    }
};
```

### Step 2: Check Configuration

```typescript
console.log('[DEBUG] OAuth Config:', {
    webClientId: oauthConfigs.google?.webClientId,
    iosClientId: oauthConfigs.google?.iosClientId,
    hasWebClientId: !!oauthConfigs.google?.webClientId,
    hasIosClientId: !!oauthConfigs.google?.iosClientId
});
```

### Step 3: Verify Firebase Configuration

Check these in Firebase Console:
- [ ] SHA-1 keys are added for your package name
- [ ] `google-services.json` and `GoogleService-Info.plist` are correctly placed
- [ ] Package name/bundle ID matches Firebase project settings

## ✅ Quick Fix Checklist

When you encounter `DEVELOPER_ERROR`:

- [ ] Generate and add SHA-1 keys to Firebase project settings
- [ ] Verify you're using Web OAuth Client ID for Android
- [ ] Check package name matches Firebase project settings
- [ ] Ensure `google-services.json` is in correct location
- [ ] Verify `appAuthRedirectScheme` in build.gradle
- [ ] Test on real device (not simulator)
- [ ] Check console logs for detailed error messages
- [ ] Try with different Google account

## 🛡️ Best Practices

1. **Always handle errors gracefully** with proper user feedback
2. **Test on real devices** for both platforms
3. **Use different Google accounts** for testing
4. **Keep SHA-1 keys secure** and don't commit them to version control
5. **Regularly update dependencies** to get security patches
6. **Implement proper error logging** for production debugging
7. **Test in all environments** (debug, release, production)

## 🔒 Security Considerations

1. **Never expose client IDs** in client-side code (they're public anyway)
2. **Always verify tokens** on your backend
3. **Use HTTPS** for all API calls
4. **Implement proper session management**
5. **Handle token refresh** appropriately
6. **Log security events** for monitoring
7. **Keep Firebase configuration files secure**

## 📁 Project Structure

```
src-v2/modules/auth/
├── oauth.ts              # Main authentication service
├── oauthConfig.ts        # Configuration management
└── GITHUB_README.md      # This documentation
```

## 🔧 Key Configuration Files

### Android
- `android/app/build.gradle` - Add `appAuthRedirectScheme`
- `android/app/google-services.json` - Firebase configuration

### iOS
- `ios/YourApp/Info.plist` - URL schemes for Google Sign-In
- `ios/YourApp/YourApp.entitlements` - Apple Sign-In capability

## 🚀 Testing

### Local Testing

```bash
# Android
yarn android

# iOS
yarn ios
```

### Environment Testing

Test in different environments to ensure SHA-1 keys work correctly:

1. **Debug Build**: Test with debug SHA-1 key
2. **Release Build**: Test with release SHA-1 key
3. **Production**: Test with production SHA-1 key

## 📞 Getting Help

If you encounter issues:

1. **Check the logs**: Look for detailed error messages in console
2. **Verify Firebase configuration**: Double-check all configuration steps
3. **Test isolation**: Try with minimal configuration to isolate the issue
4. **Community support**: Check React Native and library GitHub issues

## 🎯 Success Indicators

You'll know you've set up correctly when:
- ✅ Google Sign-In works on both debug and release builds
- ✅ Apple Sign-In works on real iOS devices
- ✅ No DEVELOPER_ERROR messages
- ✅ Authentication tokens are received properly
- ✅ Sign-out functionality works

## 📝 Contributing

If you find issues or want to improve the documentation:
1. Test your changes thoroughly
2. Update all relevant documentation
3. Include examples and troubleshooting steps
4. Keep the setup guide simple and actionable

---

## 🔑 Key Points to Remember

### Most Common Issue: DEVELOPER_ERROR
- **Cause**: Missing SHA-1 keys in Firebase project settings or wrong OAuth client ID
- **Solution**: Add SHA-1 keys to Firebase (not Google Cloud Console), use Web OAuth Client ID for Android

### Platform Differences
- **Android**: Use Web OAuth Client ID, add SHA-1 keys to Firebase
- **iOS**: Use iOS Client ID, enable Apple Sign-In capability

### Firebase Integration
- **SHA-1 keys**: Add to Firebase project settings (not Google Cloud Console)
- **Configuration files**: Use `google-services.json` and `GoogleService-Info.plist`
- **Package name**: Must match Firebase project settings

**Remember**: The DEVELOPER_ERROR is almost always a configuration issue, not a code problem. Start with SHA-1 keys in Firebase and OAuth client IDs!
