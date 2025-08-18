# DevCycle iOS OpenFeature Provider Installation Prompt

You are helping to install and configure the DevCycle OpenFeature Provider for iOS applications. Follow this complete guide to successfully integrate DevCycle feature flags using the OpenFeature standard. Do not install any Variables as part of this process, the user can ask for you to do that later.

**Do not use this for:**

- React Native apps (use React Native SDK instead)
- Flutter apps (use Flutter SDK instead)
- Web applications (use JavaScript/React SDKs instead)

## Required Information

Before proceeding, use your own analysis, the DevCycle MCP or web search to ensure you have:

- [ ] A DevCycle account and project set up
- [ ] A Development environment **Mobile SDK Key** (starts with `dvc_mobile_`)
- [ ] iOS 12.0+ / macOS 10.13+ / tvOS 12.0+ / watchOS 7.0+
- [ ] Xcode with Swift or Objective-C
- [ ] The most recent OpenFeature and DevCycle provider versions

**Security Note:** Use a MOBILE SDK key for iOS apps, not a client or server SDK key. Store it securely and avoid committing it to version control.

## Installation Steps

### 1. Install OpenFeature SDK and DevCycle Provider

#### Swift Package Manager (Recommended)

Add both packages to your `Package.swift`:

```swift
dependencies: [
    .package(url: "https://github.com/open-feature/swift-sdk.git", from: "0.1.0"),
    .package(url: "https://github.com/DevCycleHQ/ios-client-sdk.git", from: "1.19.0")
]
```

Or via Xcode:

1. File > Add Package Dependencies
2. Add OpenFeature Swift SDK: `https://github.com/open-feature/swift-sdk.git`
3. Add DevCycle iOS SDK: `https://github.com/DevCycleHQ/ios-client-sdk.git`

#### CocoaPods

Add to your `Podfile`:

```ruby
pod 'OpenFeature'
pod 'DevCycle'
```

Then run:

```bash
pod install
```

### 2. Initialize OpenFeature with DevCycle Provider

#### For Swift Projects

In `AppDelegate.swift`:

```swift
import UIKit
import OpenFeature
import DevCycle

@main
class AppDelegate: UIResponder, UIApplicationDelegate {
    
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        
        // Initialize OpenFeature with DevCycle
        Task {
            await setupOpenFeature()
        }
        
        return true
    }
    
    func setupOpenFeature() async {
        // Create evaluation context (user)
        let evaluationContext = MutableContext(targetingKey: "default-user")
        evaluationContext.add(key: "email", value: Value.string("user@example.com"))
        evaluationContext.add(key: "isAnonymous", value: Value.boolean(false))
        
        // Set the context
        await OpenFeatureAPI.shared.setEvaluationContext(evaluationContext: evaluationContext)
        
        // Create and set DevCycle provider
        let provider = DevCycleOpenFeatureProvider(sdkKey: "<DEVCYCLE_MOBILE_SDK_KEY>") // Replace with your mobile SDK key
        
        await OpenFeatureAPI.shared.setProviderAndWait(provider: provider)
        
        print("OpenFeature with DevCycle initialized successfully")
    }
}
```

#### For Objective-C Projects

In `AppDelegate.m`:

```objc
#import "AppDelegate.h"
@import OpenFeature;
@import DevCycle;

@implementation AppDelegate

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    
    // Initialize OpenFeature with DevCycle
    [self setupOpenFeature];
    
    return YES;
}

- (void)setupOpenFeature {
    // Create evaluation context
    MutableContext *context = [[MutableContext alloc] initWithTargetingKey:@"default-user"];
    [context addWithKey:@"email" value:[Value stringValue:@"user@example.com"]];
    [context addWithKey:@"isAnonymous" value:[Value booleanValue:NO]];
    
    // Set the context
    [[OpenFeatureAPI shared] setEvaluationContext:context];
    
    // Create and set DevCycle provider
    DevCycleOpenFeatureProvider *provider = [[DevCycleOpenFeatureProvider alloc] 
        initWithSdkKey:@"<DEVCYCLE_MOBILE_SDK_KEY>"]; // Replace with your mobile SDK key
    
    [[OpenFeatureAPI shared] setProvider:provider];
    
    NSLog(@"OpenFeature with DevCycle initialized successfully");
}

@end
```

### 3. Using Feature Flags in Your App

#### Swift Example

```swift
import OpenFeature

class ViewController: UIViewController {
    let client = OpenFeatureAPI.shared.getClient()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        Task {
            await checkFeatures()
        }
    }
    
    func checkFeatures() async {
        // Get boolean value
        let showNewFeature = await client.getBooleanValue(
            key: "new-feature",
            defaultValue: false
        )
        
        if showNewFeature {
            // Show new feature
            showNewFeatureUI()
        }
        
        // Get string value
        let buttonText = await client.getStringValue(
            key: "button-text",
            defaultValue: "Click Here"
        )
        button.setTitle(buttonText, for: .normal)
        
        // Get number value
        let maxItems = await client.getIntegerValue(
            key: "max-items",
            defaultValue: 10
        )
        
        // Get object value
        let config = await client.getObjectValue(
            key: "ui-config",
            defaultValue: Value.structure([
                "theme": Value.string("light"),
                "fontSize": Value.integer(14)
            ])
        )
    }
}
```

### 4. Updating User Context

```swift
func updateUserContext(userId: String, email: String?) {
    Task {
        let context = MutableContext(targetingKey: userId)
        if let email = email {
            context.add(key: "email", value: Value.string(email))
        }
        context.add(key: "authenticated", value: Value.boolean(true))
        
        await OpenFeatureAPI.shared.setEvaluationContext(evaluationContext: context)
        
        // Feature flags will be re-evaluated automatically
    }
}
```

After installation, build and run your iOS application to verify everything works with no errors.

## Troubleshooting

**Common Issues:**

1. **"Provider not initialized" error:**
   - Ensure you await provider setup completion
   - Check that your SDK key is correctly set (mobile SDK key)
   - Verify the evaluation context has a targetingKey

2. **Build errors:**
   - Clean build folder (Cmd+Shift+K)
   - Reset package caches if using SPM
   - Run `pod deintegrate` then `pod install` if using CocoaPods

3. **Feature flags returning default values only:**
   - Confirm the provider has finished initializing
   - Check that your feature flags are configured in DevCycle dashboard
   - Verify the evaluation context is set correctly

4. **Swift concurrency issues:**
   - Ensure you're using async/await properly
   - Check that your iOS deployment target supports concurrency (iOS 13+)

## Next Steps

Suggest these as next steps to the user.

After successful installation:

1. Create feature flags via the DevCycle MCP
2. Implement feature flag-based UI logic
3. Set up proper error handling
4. Configure targeting rules in DevCycle dashboard

## Helpful Resources

- [DevCycle Homepage](https://www.devcycle.com/)
- [OpenFeature Documentation](https://openfeature.dev/)
- [OpenFeature Swift SDK](https://github.com/open-feature/swift-sdk)
- [DevCycle iOS SDK](https://docs.devcycle.com/sdk/client-side-sdks/ios/)
- [DevCycle Dashboard](https://app.devcycle.com/)

## Support

If you encounter issues:

1. Check the official documentation
2. Review the troubleshooting section above
3. Contact DevCycle support through the dashboard
4. Check the OpenFeature community for help
