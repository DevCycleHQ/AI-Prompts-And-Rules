# DevCycle Android SDK Installation Prompt

You are helping to install and configure the DevCycle Android SDK in an Android application. Follow this complete guide to successfully integrate DevCycle feature flags. Do not install any Variables as part of this process, the user can ask for you to do that later.

**IMPORTANT: First detect the project configuration:**
- Check if the project uses Kotlin or Java
- Identify the build system: Gradle (most common) or Maven
- Check the minimum Android SDK version (requires API 21+)
- Use the appropriate code examples based on the detected language

**Do not use the SDK for:**
- React Native apps (use `@devcycle/react-native-client-sdk` instead)
- Flutter apps (use the Flutter SDK instead)
- Web applications (use JavaScript/React/Angular SDKs instead)
- iOS applications (use iOS SDK instead)

If you detect that the user is trying to have you install the Android SDK in an application where it will not work, please stop what you are doing and advise the user which SDK they should be using.

## Required Information

Before proceeding, use your own analysis, the DevCycle MCP or web search to ensure you have:
- [ ] A DevCycle account and project set up
- [ ] A Development environment **Mobile SDK Key** (starts with `dvc_mobile_`)
- [ ] Android project with minimum API level 21 (Android 5.0)
- [ ] Gradle or Maven build system
- [ ] The most recent DevCycle Android SDK version to install

**Security Note:** Use a MOBILE SDK key for Android apps, not a client or server SDK key. Store it securely using Android best practices.

## Installation Steps

### 1. Add DevCycle SDK Dependency

#### For Gradle (build.gradle or build.gradle.kts)

**In your module-level build.gradle file:**
```groovy
dependencies {
    implementation 'com.devcycle:android-client-sdk:2.+'
}
```

**For Kotlin DSL (build.gradle.kts):**
```kotlin
dependencies {
    implementation("com.devcycle:android-client-sdk:2.+")
}
```

#### For Maven (pom.xml)

```xml
<dependency>
    <groupId>com.devcycle</groupId>
    <artifactId>android-client-sdk</artifactId>
    <version>2.0.0</version>
</dependency>
```

### 2. Add Required Permissions

In your `AndroidManifest.xml`, add the Internet permission:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

### 3. Initialize DevCycle in Your Application

Create or update your Application class:

#### For Kotlin Projects

**Application.kt:**
```kotlin
import android.app.Application
import com.devcycle.sdk.android.api.DVCClient
import com.devcycle.sdk.android.model.DVCUser

class MyApplication : Application() {
    
    companion object {
        lateinit var dvcClient: DVCClient
    }
    
    override fun onCreate() {
        super.onCreate()
        
        // Create user object
        val user = DVCUser.builder()
            .withUserId("default-user")  // Replace with actual user ID
            .build()
        
        // Initialize DevCycle client
        dvcClient = DVCClient.builder()
            .withContext(this)
            .withUser(user)
            .withSDKKey("<DEVCYCLE_MOBILE_SDK_KEY>")  // Replace with your mobile SDK key
            .build { error ->
                if (error != null) {
                    // Handle initialization error
                    println("DevCycle initialization error: $error")
                } else {
                    // Successfully initialized
                    println("DevCycle initialized successfully")
                }
            }
    }
}
```

#### For Java Projects

**MyApplication.java:**
```java
import android.app.Application;
import com.devcycle.sdk.android.api.DVCClient;
import com.devcycle.sdk.android.model.DVCUser;
import com.devcycle.sdk.android.api.DVCCallback;

public class MyApplication extends Application {
    
    public static DVCClient dvcClient;
    
    @Override
    public void onCreate() {
        super.onCreate();
        
        // Create user object
        DVCUser user = DVCUser.builder()
            .withUserId("default-user")  // Replace with actual user ID
            .build();
        
        // Initialize DevCycle client
        dvcClient = DVCClient.builder()
            .withContext(this)
            .withUser(user)
            .withSDKKey("<DEVCYCLE_MOBILE_SDK_KEY>")  // Replace with your mobile SDK key
            .build(new DVCCallback<String>() {
                @Override
                public void onSuccess(String result) {
                    // Successfully initialized
                    System.out.println("DevCycle initialized successfully");
                }
                
                @Override
                public void onError(Throwable error) {
                    // Handle initialization error
                    System.out.println("DevCycle initialization error: " + error);
                }
            });
    }
}
```

### 4. Register Your Application Class

In your `AndroidManifest.xml`, register your custom Application class:

```xml
<application
    android:name=".MyApplication"
    android:icon="@mipmap/ic_launcher"
    android:label="@string/app_name"
    android:theme="@style/AppTheme">
    <!-- Your activities and other components -->
</application>
```

### 5. Store Your SDK Key Securely

Instead of hardcoding the SDK key, consider using:
- BuildConfig fields
- Android resource files
- Secure key management solutions
- Environment-specific build configurations

After installation, build and run your application to verify everything works with no errors.

## Troubleshooting

**Common Issues:**

1. **"DVCClient is null" error:**
   - Ensure the Application class is properly registered in AndroidManifest.xml
   - Check that initialization completes before using the client
   - Verify you're using a valid mobile SDK key

2. **Build errors after adding dependency:**
   - Sync your project with Gradle files
   - Check for version conflicts with other dependencies
   - Ensure minimum SDK version is API 21+

3. **Network/Internet issues:**
   - Verify Internet permission is added to AndroidManifest.xml
   - Check network connectivity on device/emulator
   - Ensure SDK key is correct and active

4. **ProGuard/R8 issues:**
   - If using code obfuscation, add DevCycle rules to your ProGuard configuration
   - The SDK includes consumer ProGuard rules that should be applied automatically

## Next Steps

Suggest these as next steps to the user.

After successful installation:
1. Update the user object with real user data when available
2. Create your first feature flag via the DevCycle MCP and use it in your app
3. Learn how to use variables throughout your Android application
4. Set up targeting rules for different user segments

## Helpful Resources

- [DevCycle Homepage](https://www.devcycle.com/)
- [DevCycle Documentation](https://docs.devcycle.com/)
- [Android SDK Documentation](https://docs.devcycle.com/sdk/client-side-sdks/android/)
- [DevCycle Dashboard](https://app.devcycle.com/)
- [Android SDK GitHub Repository](https://github.com/DevCycleHQ/android-client-sdk)
- [Feature Flag Best Practices](https://docs.devcycle.com/best-practices/)

## Support

If you encounter issues:
1. Check the official documentation
2. Review the troubleshooting section above
3. Contact DevCycle support through the dashboard
4. Check the GitHub repository for known issues
