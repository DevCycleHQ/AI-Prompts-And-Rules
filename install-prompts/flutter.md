# DevCycle Flutter SDK Installation Prompt

You are helping to install and configure the DevCycle Flutter SDK in a Flutter application. Follow this complete guide to successfully integrate DevCycle feature flags. Do not install any Variables as part of this process, the user can ask for you to do that later.

**Do not use the SDK for:**

- React Native apps (use `@devcycle/react-native-client-sdk` instead)
- Native iOS apps (use iOS SDK instead)
- Native Android apps (use Android SDK instead)
- Web-only applications (use JavaScript/React SDKs instead)

If you detect that the user is trying to have you install the Flutter SDK in an application where it will not work, please stop what you are doing and advise the user which SDK they should be using.

## Required Information

Before proceeding, use your own analysis, the DevCycle MCP or web search to ensure you have:

- [ ] A DevCycle account and project set up
- [ ] A Development environment **Mobile SDK Key** (starts with `dvc_mobile_`)
- [ ] Flutter 2.0+ and Dart 2.12+ installed
- [ ] iOS 11+ and/or Android API 21+ target platforms
- [ ] The most recent DevCycle Flutter SDK version to install

**Security Note:** Use a MOBILE SDK key for Flutter apps, not a client or server SDK key. Store it securely and avoid committing it to version control.

## SDK Key Configuration

**IMPORTANT:** After obtaining the SDK key, you must set it up properly:

1. **First, attempt to create an environment configuration file** (e.g., .env or config file):

   ```bash
   # .env
   DEVCYCLE_MOBILE_SDK_KEY=your_mobile_sdk_key_here
   ```

   And ensure flutter_dotenv or similar package is configured if using environment files.

2. **If you cannot create or modify configuration files** (due to system restrictions or security policies), ask the user:

   - "I'm unable to create/modify configuration files. Would you like me to:
     a) Temporarily hardcode the SDK key for testing purposes (you'll need to update it later for production)
     b) Provide you with the SDK key and instructions so you can set it up yourself?"

3. **Based on the user's response:**
   - If they choose hardcoding: Add a clear comment indicating this is temporary and should be replaced with secure configuration
   - If they choose manual setup: Provide them with the SDK key and clear instructions on how to set it up securely

**Note:** Always prefer environment files or secure storage over hardcoding for security reasons.

## Installation Steps

### 1. Add DevCycle Flutter SDK Dependency

Add the DevCycle Flutter SDK to your `pubspec.yaml`:

```yaml
dependencies:
  devcycle_flutter_client_sdk: ^1.0.0
```

Then run:

```bash
flutter pub get
```

### 2. Platform-Specific Configuration

#### iOS Configuration

No additional configuration required. The SDK automatically configures itself during the build process.

**Note:** Ensure your iOS deployment target is set to iOS 11.0 or higher in `ios/Podfile`:

```ruby
platform :ios, '11.0'
```

#### Android Configuration

No additional configuration required. The SDK automatically configures itself during the build process.

**Note:** Ensure your minimum SDK version is API 21 or higher in `android/app/build.gradle`:

```groovy
minSdkVersion 21
```

### 3. Initialize DevCycle in Your App

In your main Dart file (typically `lib/main.dart`):

```dart
import 'package:flutter/material.dart';
import 'package:devcycle_flutter_client_sdk/devcycle_flutter_client_sdk.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  // Create user object
  final user = DVCUser(
    userId: 'default-user',  // Replace with actual user ID
    email: 'user@example.com',  // Optional: for better targeting
    isAnonymous: false,  // Set to true for anonymous users
  );

  // Initialize DevCycle client
  final dvcClient = DVCClient.builder()
    .sdkKey('<DEVCYCLE_MOBILE_SDK_KEY>')  // Replace with your mobile SDK key
    .user(user)
    .build();

  // Wait for initialization
  await dvcClient.onInitialized;

  runApp(MyApp(dvcClient: dvcClient));
}

class MyApp extends StatelessWidget {
  final DVCClient dvcClient;

  const MyApp({Key? key, required this.dvcClient}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: MyHomePage(dvcClient: dvcClient),
    );
  }
}

class MyHomePage extends StatelessWidget {
  final DVCClient dvcClient;

  const MyHomePage({Key? key, required this.dvcClient}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('DevCycle Flutter Example'),
      ),
      body: Center(
        child: Text('DevCycle is initialized!'),
      ),
    );
  }
}
```

### 4. Using a Provider Pattern (Recommended)

For better state management, consider using Provider or similar state management solution:

```dart
import 'package:provider/provider.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  final user = DVCUser(userId: 'default-user');
  final dvcClient = DVCClient.builder()
    .sdkKey('<DEVCYCLE_MOBILE_SDK_KEY>')
    .user(user)
    .build();

  await dvcClient.onInitialized;

  runApp(
    Provider<DVCClient>.value(
      value: dvcClient,
      child: MyApp(),
    ),
  );
}
```

### 5. Environment Configuration

For better security, use different configurations for different environments:

1. Create environment configuration files:

   ```dart
   // lib/config/dev_config.dart
   class DevConfig {
     static const String devcycleSdkKey = 'your_dev_mobile_sdk_key';
   }

   // lib/config/prod_config.dart
   class ProdConfig {
     static const String devcycleSdkKey = 'your_prod_mobile_sdk_key';
   }
   ```

2. Use compile-time environment variables or build flavors to select the appropriate configuration.

After installation, run your application on both iOS and Android to verify everything works with no errors.

## Troubleshooting

**Common Issues:**

1. **"DVCClient initialization failed" error:**

   - Ensure you're using a valid mobile SDK key
   - Check that the user object has a valid `userId` or `isAnonymous: true`
   - Verify network connectivity

2. **iOS build errors:**

   - Run `cd ios && pod install`
   - Clean build: `flutter clean`
   - Check that iOS deployment target is 11.0+

3. **Android build errors:**

   - Clean and rebuild: `flutter clean && flutter pub get`
   - Check that minSdkVersion is 21+
   - Ensure Gradle sync is successful

4. **Hot reload issues:**
   - DevCycle client initialization should be done outside of hot reload scope
   - Consider storing the client instance in a singleton or state management solution

## Next Steps

Suggest these as next steps to the user.

After successful installation:

1. Update the user object with real user data when available
2. **Wait for user guidance** before creating any feature flags or DevCycle Variables - do not create them proactively
3. When requested, help implement variables in Flutter widgets
4. Help set up targeting rules for different user segments when asked

## Helpful Resources

- [DevCycle Homepage](https://www.devcycle.com/)
- [DevCycle Documentation](https://docs.devcycle.com/)
- [Flutter SDK Documentation](https://docs.devcycle.com/sdk/client-side-sdks/flutter/)
- [DevCycle Dashboard](https://app.devcycle.com/)
- [Flutter SDK GitHub Repository](https://github.com/DevCycleHQ/flutter-client-sdk)
- [Feature Flag Best Practices](https://docs.devcycle.com/best-practices/)

## Support

If you encounter issues:

1. Check the official documentation
2. Review the troubleshooting section above
3. Contact DevCycle support through the dashboard
4. Check the GitHub repository for known issues
