# DevCycle React Native SDK Installation Prompt

You are helping to install and configure the DevCycle React Native SDK in a React Native application. Follow this complete guide to successfully integrate DevCycle feature flags. Do not install any Variables as part of this process, the user can ask for you to do that later.

**Do not use the SDK for:**

- React web applications (use `@devcycle/react-client-sdk` instead)
- Native iOS apps (use iOS SDK instead)
- Native Android apps (use Android SDK instead)
- Flutter apps (use Flutter SDK instead)
- Expo managed workflow (requires ejecting to bare workflow)

If you detect that the user is trying to have you install the React Native SDK in an application where it will not work, please stop what you are doing and advise the user which SDK they should be using.

## Required Information

Before proceeding, use your own analysis, the DevCycle MCP or web search to ensure you have:

- [ ] A DevCycle account and project set up
- [ ] A Development environment **Mobile SDK Key** (starts with `dvc_mobile_`)
- [ ] React Native 0.64+ installed
- [ ] iOS 12+ and/or Android API 21+ target platforms
- [ ] The most recent DevCycle React Native SDK version to install

**Security Note:** Use a MOBILE SDK key for React Native apps, not a client or server SDK key. Store it securely using environment variables or secure storage solutions.

## SDK Key Configuration

**IMPORTANT:** After obtaining the SDK key, you must set it up properly:

1. **First, attempt to create an environment file** (.env) in the project root:

   ```bash
   # .env
   DEVCYCLE_MOBILE_SDK_KEY=your_mobile_sdk_key_here
   ```

   And ensure react-native-dotenv or similar is configured if not already present.

2. **If you cannot create or modify environment files** (due to system restrictions or security policies), ask the user:

   - "I'm unable to create/modify environment files. Would you like me to:
     a) Temporarily hardcode the SDK key for testing purposes (you'll need to update it later for production)
     b) Provide you with the SDK key and instructions so you can set it up yourself?"

3. **Based on the user's response:**
   - If they choose hardcoding: Add a clear comment indicating this is temporary and should be replaced with environment variables
   - If they choose manual setup: Provide them with the SDK key and clear instructions on how to set up the environment variable

**Note:** Always prefer environment variables or secure storage over hardcoding for security reasons.

## Installation Steps

### 1. Install the DevCycle React Native SDK

```bash
# Using npm
npm install --save @devcycle/react-native-client-sdk

# Using yarn
yarn add @devcycle/react-native-client-sdk
```

### 2. Platform-Specific Setup

#### iOS Setup

Navigate to the iOS folder and install pods:

```bash
cd ios && pod install
```

**Note:** Ensure your iOS deployment target is set to iOS 12.0 or higher in your `Podfile`:

```ruby
platform :ios, '12.0'
```

#### Android Setup

No additional setup required for Android. The SDK automatically configures itself during the build process.

**Note:** Ensure your minimum SDK version is API 21 or higher in `android/build.gradle`:

```groovy
minSdkVersion = 21
```

### 3. Initialize DevCycle in Your App

In your main App component or entry point:

```javascript
import React, { useEffect } from 'react'
import { withDevCycleProvider } from '@devcycle/react-native-client-sdk'

function App() {
  return (
    // Your app components
  )
}

// Wrap your app with the DevCycle provider
export default withDevCycleProvider({
  sdkKey: '<DEVCYCLE_MOBILE_SDK_KEY>',  // Replace with your mobile SDK key
  user: {
    user_id: 'default-user',  // Replace with actual user ID
    email: 'user@example.com',  // Optional: for better targeting
    isAnonymous: false  // Set to true for anonymous users
  }
})(App)
```

### 4. Alternative: Using DevCycleProvider Component

If you prefer using a provider component pattern:

```javascript
import React from "react";
import { DevCycleProvider } from "@devcycle/react-native-client-sdk";

export default function App() {
  const user = {
    user_id: "default-user", // Replace with actual user ID
    email: "user@example.com", // Optional
    isAnonymous: false,
  };

  return (
    <DevCycleProvider
      sdkKey="<DEVCYCLE_MOBILE_SDK_KEY>" // Replace with your mobile SDK key
      user={user}
    >
      {/* Your app components */}
    </DevCycleProvider>
  );
}
```

### 5. Environment-Specific Configuration

For better security, use environment variables:

1. Install react-native-config:

   ```bash
   npm install react-native-config
   ```

2. Create `.env` files for different environments:

   ```bash
   # .env.development
   DEVCYCLE_MOBILE_SDK_KEY=your_dev_mobile_sdk_key

   # .env.production
   DEVCYCLE_MOBILE_SDK_KEY=your_prod_mobile_sdk_key
   ```

3. Use in your code:

   ```javascript
   import Config from "react-native-config";

   export default withDevCycleProvider({
     sdkKey: Config.DEVCYCLE_MOBILE_SDK_KEY,
     user: { user_id: "default-user" },
   })(App);
   ```

After installation, rebuild your application for both platforms and verify everything runs with no errors.

## Troubleshooting

**Common Issues:**

1. **"DevCycle is not initialized" error:**

   - Ensure the provider wraps your entire app
   - Check that your SDK key is correctly set (mobile SDK key)
   - Verify the user object has a valid `user_id` or `isAnonymous: true`

2. **iOS build errors:**

   - Run `cd ios && pod install` again
   - Clean build: `cd ios && xcodebuild clean`
   - Check that iOS deployment target is 12.0+

3. **Android build errors:**

   - Clean and rebuild: `cd android && ./gradlew clean`
   - Check that minSdkVersion is 21+
   - Sync project with Gradle files

4. **Metro bundler issues:**

   - Clear cache: `npx react-native start --reset-cache`
   - Delete node_modules and reinstall
   - Restart Metro bundler

5. **TypeScript errors:**
   - The SDK includes TypeScript definitions
   - Ensure @types/react-native is installed and up to date

## Next Steps

Suggest these as next steps to the user.

After successful installation:

1. Update the user object with real user data when users log in
2. **Wait for user guidance** before creating any feature flags or DevCycle Variables - do not create them proactively
3. When requested, help implement feature flags with React Native hooks
4. Help set up targeting rules for different user segments when asked

## Helpful Resources

- [DevCycle Homepage](https://www.devcycle.com/)
- [DevCycle Documentation](https://docs.devcycle.com/)
- [React Native SDK Documentation](https://docs.devcycle.com/sdk/client-side-sdks/react-native/)
- [DevCycle Dashboard](https://app.devcycle.com/)
- [React Native SDK GitHub Repository](https://github.com/DevCycleHQ/react-native-client-sdk)
- [Feature Flag Best Practices](https://docs.devcycle.com/best-practices/)

## Support

If you encounter issues:

1. Check the official documentation
2. Review the troubleshooting section above
3. Contact DevCycle support through the dashboard
4. Check the GitHub repository for known issues
