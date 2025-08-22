# DevCycle Java SDK Installation Prompt

You are helping to install and configure the DevCycle Java SDK in a Java server application. Follow this complete guide to successfully integrate DevCycle feature flags. Do not install any Variables as part of this process, the user can ask for you to do that later.

**IMPORTANT: First detect the project configuration:**

- Check the build system: Maven (pom.xml) or Gradle (build.gradle)
- Identify the framework: Spring Boot, Micronaut, Quarkus, or plain Java
- Check Java version (requires Java 8+)

**Do not use the SDK for:**

- Android applications (use Android SDK instead)
- Client-side applications (use appropriate client SDKs instead)
- Kotlin-only projects (while compatible, consider Kotlin-specific patterns)

If you detect that the user is trying to have you install the Java SDK in an application where it will not work, please stop what you are doing and advise the user which SDK they should be using.

## Required Information

Before proceeding, use your own analysis, the DevCycle MCP or web search to ensure you have:

- [ ] A DevCycle account and project set up
- [ ] A Development environment **Server SDK Key** (starts with `dvc_server_`)
- [ ] Java 8+ installed
- [ ] Maven or Gradle build system
- [ ] The most recent DevCycle Java SDK version to install

**Security Note:** Use a SERVER SDK key for Java backend applications. Never expose server keys to client-side code. Store keys in environment variables or configuration files.

## SDK Key Configuration

**IMPORTANT:** After obtaining the SDK key, you must set it up properly:

1. **First, attempt to create a configuration file** (application.properties or application.yml):

   ```properties
   # application.properties
   devcycle.server.sdk.key=your_server_sdk_key_here
   ```

   Or use environment variables:

   ```bash
   export DEVCYCLE_SERVER_SDK_KEY=your_server_sdk_key_here
   ```

2. **If you cannot create or modify configuration files** (due to system restrictions or security policies), ask the user:

   - "I'm unable to create/modify configuration files. Would you like me to:
     a) Temporarily hardcode the SDK key for testing purposes (you'll need to update it later for production)
     b) Provide you with the SDK key and instructions so you can set it up yourself?"

3. **Based on the user's response:**
   - If they choose hardcoding: Add a clear comment indicating this is temporary and should be replaced with configuration files or environment variables
   - If they choose manual setup: Provide them with the SDK key and clear instructions on how to set it up

**Note:** Always prefer configuration files or environment variables over hardcoding for security reasons.

## Installation Steps

### 1. Add DevCycle Java SDK Dependency

#### For Maven (pom.xml)

```xml
<dependency>
    <groupId>com.devcycle</groupId>
    <artifactId>java-server-sdk</artifactId>
    <version>2.0.0</version>
</dependency>
```

#### For Gradle (build.gradle)

```groovy
dependencies {
    implementation 'com.devcycle:java-server-sdk:2.0.0'
}
```

#### For Gradle Kotlin DSL (build.gradle.kts)

```kotlin
dependencies {
    implementation("com.devcycle:java-server-sdk:2.0.0")
}
```

### 2. Create DevCycle Configuration Class

Create a configuration class to manage the DevCycle client:

```java
package com.yourcompany.config;

import com.devcycle.sdk.server.local.api.DevCycleLocalClient;
import com.devcycle.sdk.server.common.model.DevCycleUser;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class DevCycleConfig {
    private static final Logger logger = LoggerFactory.getLogger(DevCycleConfig.class);
    private static DevCycleLocalClient client;
    private static final Object lock = new Object();

    public static DevCycleLocalClient getInstance() {
        if (client == null) {
            synchronized (lock) {
                if (client == null) {
                    initializeClient();
                }
            }
        }
        return client;
    }

    private static void initializeClient() {
        String sdkKey = System.getenv("DEVCYCLE_SERVER_SDK_KEY");
        if (sdkKey == null || sdkKey.isEmpty()) {
            sdkKey = "<DEVCYCLE_SERVER_SDK_KEY>"; // Replace with your server SDK key
        }

        try {
            client = new DevCycleLocalClient(sdkKey);
            logger.info("DevCycle initialized successfully");
        } catch (Exception e) {
            logger.error("Failed to initialize DevCycle", e);
            throw new RuntimeException("DevCycle initialization failed", e);
        }
    }

    public static void shutdown() {
        if (client != null) {
            try {
                client.close();
                logger.info("DevCycle client closed successfully");
            } catch (Exception e) {
                logger.error("Error closing DevCycle client", e);
            }
        }
    }
}
```

### 3. Framework-Specific Integration

#### For Spring Boot Applications

Create a Spring configuration class:

```java
package com.yourcompany.config;

import com.devcycle.sdk.server.local.api.DevCycleLocalClient;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import javax.annotation.PreDestroy;

@Configuration
public class DevCycleSpringConfig {

    @Value("${devcycle.server.sdk.key:#{environment.DEVCYCLE_SERVER_SDK_KEY}}")
    private String sdkKey;

    private DevCycleLocalClient client;

    @Bean
    public DevCycleLocalClient devCycleClient() {
        if (sdkKey == null || sdkKey.isEmpty()) {
            throw new IllegalStateException("DevCycle SDK key is not configured");
        }

        client = new DevCycleLocalClient(sdkKey);
        return client;
    }

    @PreDestroy
    public void cleanup() {
        if (client != null) {
            client.close();
        }
    }
}
```

Add to `application.properties` or `application.yml`:

**application.properties:**

```properties
# application.properties
devcycle.server.sdk.key=${DEVCYCLE_SERVER_SDK_KEY:your_default_key_here}
```

**application.yml:**

```yaml
# application.yml
devcycle:
  server:
    sdk:
      key: ${DEVCYCLE_SERVER_SDK_KEY:your_default_key_here}
```

### 4. Using DevCycle in Your Application

Example service using DevCycle:

```java
package com.yourcompany.service;

import com.devcycle.sdk.server.local.api.DevCycleLocalClient;
import com.devcycle.sdk.server.common.model.DevCycleUser;
import com.devcycle.sdk.server.common.model.Variable;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class FeatureService {

    @Autowired
    private DevCycleLocalClient devCycleClient;

    public boolean isFeatureEnabled(String userId, String userEmail) {
        // Create user object
        DevCycleUser user = DevCycleUser.builder()
            .userId(userId)
            .email(userEmail)
            .customData(Map.of(
                "plan", "premium",
                "role", "admin"
            ))
            .build();

        // Check feature flag value
        Variable<Boolean> variable = devCycleClient.variable(user, "feature-key", false);
        return variable.getValue();
    }
}
```

Example REST controller:

```java
@RestController
@RequestMapping("/api")
public class FeatureController {

    @Autowired
    private FeatureService featureService;

    @GetMapping("/feature")
    public ResponseEntity<?> checkFeature(@RequestHeader(value = "X-User-Id", defaultValue = "anonymous") String userId) {
        boolean enabled = featureService.isFeatureEnabled(userId, null);

        Map<String, Object> response = new HashMap<>();
        response.put("featureEnabled", enabled);

        return ResponseEntity.ok(response);
    }
}
```

After installation, build and run your Java application to verify everything works with no errors.

## Troubleshooting

**Common Issues:**

1. **"DevCycle client not initialized" error:**

   - Ensure the client is properly initialized on application startup
   - Check that your SDK key is correctly set (server SDK key)
   - Verify Spring beans are properly configured if using Spring

2. **Dependency resolution errors:**

   - Run `mvn clean install` or `gradle clean build`
   - Check for version conflicts with other dependencies
   - Ensure repository access for DevCycle artifacts

3. **Environment variable not found:**

   - Ensure environment variables are set before starting the application
   - Check Spring property resolution if using Spring Boot
   - Verify the SDK key environment variable name matches

4. **Connection/Network errors:**
   - Check network connectivity
   - Verify the server SDK key is valid and active
   - Ensure firewall/proxy settings allow outbound HTTPS

## Next Steps

Suggest these as next steps to the user.

After successful installation:

1. Set up user identification logic for your application
2. **Wait for user guidance** before creating any feature flags or DevCycle Variables - do not create them proactively
3. Implement proper error handling for feature flag evaluations if needed
4. Help set up targeting rules for different user segments when requested

## Helpful Resources

- [DevCycle Homepage](https://www.devcycle.com/)
- [DevCycle Documentation](https://docs.devcycle.com/)
- [Java SDK Documentation](https://docs.devcycle.com/sdk/server-side-sdks/java/)
- [DevCycle Dashboard](https://app.devcycle.com/)
- [Java SDK GitHub Repository](https://github.com/DevCycleHQ/java-server-sdk)
- [Feature Flag Best Practices](https://docs.devcycle.com/best-practices/)

## Support

If you encounter issues:

1. Check the official documentation
2. Review the troubleshooting section above
3. Contact DevCycle support through the dashboard
4. Check the GitHub repository for known issues
