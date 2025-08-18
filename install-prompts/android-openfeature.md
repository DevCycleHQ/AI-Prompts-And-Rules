# DevCycle Android OpenFeature Provider Installation Prompt

You are helping to install and configure the DevCycle OpenFeature Provider for Android applications. Follow this complete guide to successfully integrate DevCycle feature flags using the OpenFeature standard. Do not install any Variables as part of this process, the user can ask for you to do that later.

**Do not use this for:**

- React Native apps (use React Native SDK instead)
- Flutter apps (use Flutter SDK instead)
- Web applications (use JavaScript/React SDKs instead)
- iOS applications (use iOS SDK instead)

## Required Information

Before proceeding, use your own analysis, the DevCycle MCP or web search to ensure you have:

- [ ] A DevCycle account and project set up
- [ ] A Development environment **Mobile SDK Key** (starts with `dvc_mobile_`)
- [ ] Android project with minimum API level 21 (Android 5.0)
- [ ] Kotlin or Java project
- [ ] The most recent OpenFeature and DevCycle provider versions

**Security Note:** Use a MOBILE SDK key for Android apps, not a client or server SDK key. Store it securely using Android best practices.

## Installation Steps

### 1. Add OpenFeature SDK and DevCycle Provider Dependencies

#### For Gradle (build.gradle or build.gradle.kts)

**In your module-level build.gradle file:**

```groovy
dependencies {
    implementation 'dev.openfeature:android-sdk:0.2.0'
    implementation 'com.devcycle:android-client-sdk:2.+'
    implementation 'com.devcycle:android-client-sdk-openfeature:2.+'
}
```

**For Kotlin DSL (build.gradle.kts):**

```kotlin
dependencies {
    implementation("dev.openfeature:android-sdk:0.2.0")
    implementation("com.devcycle:android-client-sdk:2.+")
    implementation("com.devcycle:android-client-sdk-openfeature:2.+")
}
```

### 2. Add Required Permissions

In your `AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

### 3. Initialize OpenFeature with DevCycle Provider

Create or update your Application class:

#### For Kotlin Projects

**Application.kt:**

```kotlin
import android.app.Application
import dev.openfeature.sdk.OpenFeatureAPI
import dev.openfeature.sdk.ImmutableContext
import dev.openfeature.sdk.Value
import com.devcycle.sdk.android.openfeature.DevCycleOpenFeatureProvider

class MyApplication : Application() {
    
    override fun onCreate() {
        super.onCreate()
        
        // Initialize OpenFeature with DevCycle
        initializeOpenFeature()
    }
    
    private fun initializeOpenFeature() {
        // Create evaluation context
        val evaluationContext = ImmutableContext(
            targetingKey = "default-user", // Required: unique user identifier
            attributes = mapOf(
                "email" to Value.String("user@example.com"),
                "isAnonymous" to Value.Boolean(false),
                "plan" to Value.String("premium"),
                "role" to Value.String("admin")
            )
        )
        
        // Set the evaluation context
        OpenFeatureAPI.setEvaluationContext(evaluationContext)
        
        // Create DevCycle provider
        val provider = DevCycleOpenFeatureProvider(
            context = this,
            sdkKey = "<DEVCYCLE_MOBILE_SDK_KEY>" // Replace with your mobile SDK key
        )
        
        // Set the provider
        OpenFeatureAPI.setProviderAndWait(provider) { result ->
            if (result.isSuccess) {
                println("OpenFeature with DevCycle initialized successfully")
            } else {
                println("Failed to initialize: ${result.exceptionOrNull()}")
            }
        }
    }
}
```

#### For Java Projects

**MyApplication.java:**

```java
import android.app.Application;
import dev.openfeature.sdk.OpenFeatureAPI;
import dev.openfeature.sdk.ImmutableContext;
import dev.openfeature.sdk.Value;
import com.devcycle.sdk.android.openfeature.DevCycleOpenFeatureProvider;
import java.util.HashMap;
import java.util.Map;

public class MyApplication extends Application {
    
    @Override
    public void onCreate() {
        super.onCreate();
        
        // Initialize OpenFeature with DevCycle
        initializeOpenFeature();
    }
    
    private void initializeOpenFeature() {
        // Create evaluation context
        Map<String, Value> attributes = new HashMap<>();
        attributes.put("email", new Value.String("user@example.com"));
        attributes.put("isAnonymous", new Value.Boolean(false));
        attributes.put("plan", new Value.String("premium"));
        attributes.put("role", new Value.String("admin"));
        
        ImmutableContext evaluationContext = new ImmutableContext(
            "default-user", // targetingKey
            attributes
        );
        
        // Set the evaluation context
        OpenFeatureAPI.setEvaluationContext(evaluationContext);
        
        // Create DevCycle provider
        DevCycleOpenFeatureProvider provider = new DevCycleOpenFeatureProvider(
            this,
            "<DEVCYCLE_MOBILE_SDK_KEY>" // Replace with your mobile SDK key
        );
        
        // Set the provider
        OpenFeatureAPI.setProviderAndWait(provider, result -> {
            if (result.isSuccess()) {
                System.out.println("OpenFeature with DevCycle initialized successfully");
            } else {
                System.out.println("Failed to initialize: " + result.getException());
            }
        });
    }
}
```

### 4. Register Your Application Class

In your `AndroidManifest.xml`:

```xml
<application
    android:name=".MyApplication"
    android:icon="@mipmap/ic_launcher"
    android:label="@string/app_name"
    android:theme="@style/AppTheme">
    <!-- Your activities and other components -->
</application>
```

### 5. Using Feature Flags in Your App

#### Kotlin Example

```kotlin
import dev.openfeature.sdk.OpenFeatureAPI
import dev.openfeature.sdk.Client

class MainActivity : AppCompatActivity() {
    private val client: Client = OpenFeatureAPI.getClient()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        checkFeatures()
    }
    
    private fun checkFeatures() {
        // Get boolean value
        val showNewFeature = client.getBooleanValue(
            "new-feature",
            false
        )
        
        if (showNewFeature) {
            // Show new feature UI
            showNewFeatureView()
        }
        
        // Get string value
        val buttonText = client.getStringValue(
            "button-text",
            "Click Here"
        )
        button.text = buttonText
        
        // Get number value
        val maxItems = client.getIntegerValue(
            "max-items",
            10
        )
        
        // Get object value
        val config = client.getObjectValue(
            "ui-config",
            Value.Structure(mapOf(
                "theme" to Value.String("light"),
                "fontSize" to Value.Integer(14)
            ))
        )
    }
}
```

### 6. Updating User Context

```kotlin
fun updateUserContext(userId: String, email: String?) {
    val attributes = mutableMapOf<String, Value>(
        "authenticated" to Value.Boolean(true)
    )
    
    email?.let {
        attributes["email"] = Value.String(it)
    }
    
    val newContext = ImmutableContext(
        targetingKey = userId,
        attributes = attributes
    )
    
    OpenFeatureAPI.setEvaluationContext(newContext)
    
    // Feature flags will be re-evaluated automatically
}
```

After installation, build and run your Android application to verify everything works with no errors.

## Troubleshooting

**Common Issues:**

1. **"Provider not initialized" error:**
   - Ensure the Application class is properly registered in AndroidManifest.xml
   - Check that initialization completes before using the client
   - Verify you're using a valid mobile SDK key

2. **Build errors:**
   - Sync your project with Gradle files
   - Check for version conflicts with other dependencies
   - Ensure minimum SDK version is API 21+

3. **Feature flags returning default values only:**
   - Confirm the provider has finished initializing
   - Check that your feature flags are configured in DevCycle dashboard
   - Verify the evaluation context has a valid targetingKey

4. **Network/Internet issues:**
   - Verify Internet permission is added to AndroidManifest.xml
   - Check network connectivity on device/emulator
   - Ensure SDK key is correct and active

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
- [OpenFeature Android SDK](https://github.com/open-feature/android-sdk)
- [DevCycle Android SDK](https://docs.devcycle.com/sdk/client-side-sdks/android/)
- [DevCycle Dashboard](https://app.devcycle.com/)

## Support

If you encounter issues:

1. Check the official documentation
2. Review the troubleshooting section above
3. Contact DevCycle support through the dashboard
4. Check the OpenFeature community for help
