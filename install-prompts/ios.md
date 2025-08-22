# DevCycle iOS SDK Installation Prompt

You are helping to install and configure the DevCycle iOS SDK in an iOS application. Follow this complete guide to successfully integrate DevCycle feature flags. Do not install any Variables as part of this process, the user can ask for you to do that later.

**IMPORTANT: First detect which language and package manager the project uses:**

- Check if the project uses Swift or Objective-C
- Identify the package manager: Swift Package Manager (Package.swift or Xcode packages), CocoaPods (Podfile), or Carthage (Cartfile)
- Use the appropriate code examples based on the detected language

**Do not use the SDK for:**

- React Native apps (use `@devcycle/react-native-client-sdk` instead)
- Flutter apps (use the Flutter SDK instead)
- Web applications (use JavaScript/React/Angular SDKs instead)
- Server-side Swift applications (use server SDK instead)

If you detect that the user is trying to have you install the iOS SDK in an application where it will not work, please stop what you are doing and advise the user which SDK they should be using.

## Required Information

Before proceeding, use your own analysis, the DevCycle MCP or web search to ensure you have:

- [ ] A DevCycle account and project set up
- [ ] A Development environment **Mobile SDK Key** (starts with `dvc_mobile_`)
- [ ] iOS 12.0+ / macOS 10.13+ / tvOS 12.0+ / watchOS 7.0+
- [ ] Xcode project with Swift or Objective-C
- [ ] The most recent DevCycle iOS SDK version to install

**Security Note:** Use a MOBILE SDK key for iOS apps, not a client or server SDK key. Store it securely and avoid committing it to version control.

## SDK Key Configuration

**IMPORTANT:** After obtaining the SDK key, you must set it up properly:

1. **First, attempt to set up the SDK key securely** (e.g., in a configuration file or using xcconfig):

   ```swift
   // In a Config.xcconfig file or environment configuration
   DEVCYCLE_MOBILE_SDK_KEY = your_mobile_sdk_key_here
   ```

   Then reference it in your code using build settings.

2. **If you cannot create or modify configuration files** (due to system restrictions or security policies), ask the user:

   - "I'm unable to create/modify configuration files. Would you like me to:
     a) Temporarily hardcode the SDK key for testing purposes (you'll need to update it later for production)
     b) Provide you with the SDK key and instructions so you can set it up yourself?"

3. **Based on the user's response:**
   - If they choose hardcoding: Add a clear comment indicating this is temporary and should be replaced with secure configuration
   - If they choose manual setup: Provide them with the SDK key and clear instructions on how to set it up securely

**Note:** Always prefer configuration files or environment variables over hardcoding for security reasons.

## Installation Steps

### 1. Install the DevCycle iOS SDK

Choose the installation method based on the detected package manager:

#### Swift Package Manager (Recommended)

**In Package.swift:**

```swift
dependencies: [
    .package(url: "https://github.com/DevCycleHQ/ios-client-sdk.git", .upToNextMajor(from: "1.19.0"))
],
targets: [
    .target(
        name: "YOUR_TARGET",
        dependencies: ["DevCycle"]
    )
]
```

**Or via Xcode:**

1. File > Add Package Dependencies
2. Enter: `https://github.com/DevCycleHQ/ios-client-sdk.git`
3. Choose version rule: Up to Next Major Version
4. Add to your target

#### CocoaPods

Add to your `Podfile`:

```ruby
pod 'DevCycle'
```

Then run:

```bash
pod install
```

#### Carthage

Add to your `Cartfile`:

```text
github "DevCycleHQ/ios-client-sdk"
```

Then run:

```bash
carthage update --platform iOS --use-xcframeworks
```

Drag the built `.xcframework` from `Carthage/Build` into your Xcode project's "Frameworks and Libraries" section.

### 2. Initialize DevCycle in Your App

Add the initialization code to your AppDelegate:

#### For Swift Projects

In `AppDelegate.swift`:

```swift
import UIKit
import DevCycle

@main
class AppDelegate: UIResponder, UIApplicationDelegate {

    var devcycleClient: DevCycleClient?

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {

        // Create user object
        let user = try? DevCycleUser.builder()
            .userId("default-user")  // Replace with actual user ID
            .build()

        // Initialize DevCycle client
        guard let user = user else { return true }

        self.devcycleClient = try? DevCycleClient.builder()
            .sdkKey("<DEVCYCLE_MOBILE_SDK_KEY>")  // Replace with your mobile SDK key
            .user(user)
            .build(onInitialized: { error in
                if let error = error {
                    print("DevCycle initialization error: \(error)")
                } else {
                    print("DevCycle initialized successfully")
                }
            })

        return true
    }
}
```

#### For Objective-C Projects

In `AppDelegate.m`:

```objc
#import "AppDelegate.h"
@import DevCycle;

@interface AppDelegate ()
@property (nonatomic, strong) DevCycleClient *devcycleClient;
@end

@implementation AppDelegate

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

    // Create user object
    DevCycleUser *user = [DevCycleUser initializeWithUserId:@"default-user"];

    // Initialize DevCycle client
    self.devcycleClient = [DevCycleClient initialize:@"<DEVCYCLE_MOBILE_SDK_KEY>"
                                                 user:user
                                              options:nil
                                        onInitialized:^(NSError * _Nullable error) {
        if (error) {
            NSLog(@"DevCycle initialization error: %@", error);
        } else {
            NSLog(@"DevCycle initialized successfully");
        }
    }];

    return YES;
}

@end
```

### 3. Make DevCycle Client Accessible

For Swift, you can pass the client instance through dependency injection or make it accessible via a singleton pattern. For Objective-C, you can access it through the AppDelegate.

### 4. Store Your SDK Key Securely

Instead of hardcoding the SDK key, consider using:

- Xcode configuration files (.xcconfig)
- Environment variables
- Secure key management solutions

After installation, build and run your application to verify everything works with no errors.

## Troubleshooting

**Common Issues:**

1. **"DevCycle client is nil" error:**

   - Ensure the SDK key is correctly set (mobile SDK key, not client/server)
   - Check that the user object has a valid `userId` or `isAnonymous: true`
   - Verify initialization completes before using the client

2. **Build errors after installation:**

   - Clean build folder (Cmd+Shift+K)
   - For SPM: Reset package caches in Xcode
   - For CocoaPods: Run `pod deintegrate` then `pod install`
   - For Carthage: Delete Carthage folder and rebuild

3. **Package conflicts:**

   - If you have naming conflicts, use the alternative URL: `https://github.com/DevCycleHQ/devcycle-ios-client-sdk.git`

4. **Platform compatibility issues:**
   - Verify minimum iOS version is 12.0+
   - Check that deployment target matches requirements

## Next Steps

Suggest these as next steps to the user.

After successful installation:

1. Update the user object with real user data when available
2. **Wait for user guidance** before creating any feature flags or DevCycle Variables - do not create them proactively
3. When requested, help implement variables throughout the iOS application
4. Help set up targeting rules for different user segments when asked

## Helpful Resources

- [DevCycle Homepage](https://www.devcycle.com/)
- [DevCycle Documentation](https://docs.devcycle.com/)
- [iOS SDK Documentation](https://docs.devcycle.com/sdk/client-side-sdks/ios/)
- [DevCycle Dashboard](https://app.devcycle.com/)
- [iOS SDK Example App](https://github.com/DevCycleHQ/ios-client-sdk/tree/main/Example)
- [Feature Flag Best Practices](https://docs.devcycle.com/best-practices/)

## Support

If you encounter issues:

1. Check the official documentation
2. Review the troubleshooting section above
3. Contact DevCycle support through the dashboard
4. Check the GitHub repository for known issues
