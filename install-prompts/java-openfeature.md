# DevCycle Java OpenFeature Provider Installation Prompt

You are helping to install and configure the DevCycle OpenFeature Provider for Java server applications. Follow this complete guide to successfully integrate DevCycle feature flags using the OpenFeature standard. Do not install any Variables as part of this process, the user can ask for you to do that later.

**Do not use this for:**

- Android applications (use Android SDK instead)
- Client-side applications (use appropriate client SDKs instead)

## Required Information

Before proceeding, use your own analysis, the DevCycle MCP or web search to ensure you have:

- [ ] A DevCycle account and project set up
- [ ] A Development environment **Server SDK Key** (starts with `dvc_server_`)
- [ ] Java 8+ installed
- [ ] Maven or Gradle build system
- [ ] The most recent OpenFeature and DevCycle provider versions

**Security Note:** Use a SERVER SDK key for Java backend applications. Never expose server keys to client-side code. Store keys in environment variables or configuration files.

## Installation Steps

### 1. Add OpenFeature SDK and DevCycle Provider Dependencies

#### For Maven (pom.xml)

```xml
<dependencies>
    <!-- OpenFeature SDK -->
    <dependency>
        <groupId>dev.openfeature</groupId>
        <artifactId>sdk</artifactId>
        <version>1.7.0</version>
    </dependency>
    
    <!-- DevCycle SDK -->
    <dependency>
        <groupId>com.devcycle</groupId>
        <artifactId>java-server-sdk</artifactId>
        <version>2.0.0</version>
    </dependency>
    
    <!-- DevCycle OpenFeature Provider -->
    <dependency>
        <groupId>com.devcycle</groupId>
        <artifactId>java-server-sdk-openfeature</artifactId>
        <version>2.0.0</version>
    </dependency>
</dependencies>
```

#### For Gradle (build.gradle)

```groovy
dependencies {
    implementation 'dev.openfeature:sdk:1.7.0'
    implementation 'com.devcycle:java-server-sdk:2.0.0'
    implementation 'com.devcycle:java-server-sdk-openfeature:2.0.0'
}
```

### 2. Initialize OpenFeature with DevCycle Provider

Create a configuration class:

```java
package com.yourcompany.config;

import dev.openfeature.sdk.OpenFeatureAPI;
import dev.openfeature.sdk.Client;
import com.devcycle.sdk.server.local.api.DevCycleLocalClient;
import com.devcycle.sdk.server.openfeature.DevCycleProvider;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class OpenFeatureConfig {
    private static final Logger logger = LoggerFactory.getLogger(OpenFeatureConfig.class);
    private static Client openFeatureClient;
    private static DevCycleLocalClient devCycleClient;
    private static final Object lock = new Object();
    
    public static Client getInstance() {
        if (openFeatureClient == null) {
            synchronized (lock) {
                if (openFeatureClient == null) {
                    initialize();
                }
            }
        }
        return openFeatureClient;
    }
    
    private static void initialize() {
        String sdkKey = System.getenv("DEVCYCLE_SERVER_SDK_KEY");
        if (sdkKey == null || sdkKey.isEmpty()) {
            throw new RuntimeException("DEVCYCLE_SERVER_SDK_KEY is not configured");
        }
        
        try {
            // Initialize DevCycle client
            devCycleClient = new DevCycleLocalClient(sdkKey);
            
            // Create DevCycle provider
            DevCycleProvider provider = new DevCycleProvider(devCycleClient);
            
            // Set the provider for OpenFeature
            OpenFeatureAPI api = OpenFeatureAPI.getInstance();
            api.setProviderAndWait(provider);
            
            // Get OpenFeature client
            openFeatureClient = api.getClient();
            
            logger.info("OpenFeature with DevCycle initialized successfully");
        } catch (Exception e) {
            logger.error("Failed to initialize OpenFeature with DevCycle", e);
            throw new RuntimeException("OpenFeature initialization failed", e);
        }
    }
    
    public static void shutdown() {
        if (devCycleClient != null) {
            try {
                devCycleClient.close();
                logger.info("DevCycle client closed successfully");
            } catch (Exception e) {
                logger.error("Error closing DevCycle client", e);
            }
        }
    }
}
```

### 3. Spring Boot Integration

For Spring Boot applications:

```java
package com.yourcompany.config;

import dev.openfeature.sdk.Client;
import dev.openfeature.sdk.OpenFeatureAPI;
import com.devcycle.sdk.server.local.api.DevCycleLocalClient;
import com.devcycle.sdk.server.openfeature.DevCycleProvider;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import javax.annotation.PreDestroy;

@Configuration
public class OpenFeatureSpringConfig {
    
    @Value("${devcycle.server.sdk.key:#{environment.DEVCYCLE_SERVER_SDK_KEY}}")
    private String sdkKey;
    
    private DevCycleLocalClient devCycleClient;
    
    @Bean
    public Client openFeatureClient() throws Exception {
        if (sdkKey == null || sdkKey.isEmpty()) {
            throw new IllegalStateException("DevCycle SDK key is not configured");
        }
        
        // Initialize DevCycle client
        devCycleClient = new DevCycleLocalClient(sdkKey);
        
        // Create provider and set it
        DevCycleProvider provider = new DevCycleProvider(devCycleClient);
        OpenFeatureAPI api = OpenFeatureAPI.getInstance();
        api.setProviderAndWait(provider);
        
        return api.getClient();
    }
    
    @PreDestroy
    public void cleanup() {
        if (devCycleClient != null) {
            devCycleClient.close();
        }
    }
}
```

### 4. Using Feature Flags with OpenFeature

```java
package com.yourcompany.service;

import dev.openfeature.sdk.Client;
import dev.openfeature.sdk.EvaluationContext;
import dev.openfeature.sdk.MutableContext;
import dev.openfeature.sdk.Value;
import dev.openfeature.sdk.Structure;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import java.util.Map;
import java.util.HashMap;

@Service
public class FeatureService {
    
    @Autowired
    private Client openFeatureClient;
    
    public boolean isFeatureEnabled(String userId, String featureKey) {
        // Create evaluation context
        MutableContext context = new MutableContext(userId);
        context.add("plan", "premium");
        context.add("role", "admin");
        
        // Get boolean value
        return openFeatureClient.getBooleanValue(featureKey, false, context);
    }
    
    public String getStringConfig(String userId, String configKey) {
        MutableContext context = new MutableContext(userId);
        return openFeatureClient.getStringValue(configKey, "default", context);
    }
    
    public Integer getNumberConfig(String userId, String configKey) {
        MutableContext context = new MutableContext(userId);
        return openFeatureClient.getIntegerValue(configKey, 100, context);
    }
    
    public Map<String, Object> getObjectConfig(String userId, String configKey) {
        MutableContext context = new MutableContext(userId);
        
        // Default value
        Map<String, Value> defaultMap = new HashMap<>();
        defaultMap.put("theme", new Value("light"));
        defaultMap.put("fontSize", new Value(14));
        Structure defaultValue = new Structure(defaultMap);
        
        Value result = openFeatureClient.getObjectValue(configKey, defaultValue, context);
        return result.asStructure().asMap();
    }
}
```

### 5. REST Controller Example

```java
@RestController
@RequestMapping("/api/features")
public class FeatureController {
    
    @Autowired
    private Client openFeatureClient;
    
    @GetMapping("/check")
    public ResponseEntity<?> checkFeature(
        @RequestHeader(value = "X-User-Id", defaultValue = "anonymous") String userId,
        @RequestParam String featureKey
    ) {
        MutableContext context = new MutableContext(userId);
        context.add("timestamp", System.currentTimeMillis());
        
        boolean enabled = openFeatureClient.getBooleanValue(featureKey, false, context);
        
        Map<String, Object> response = new HashMap<>();
        response.put("featureKey", featureKey);
        response.put("enabled", enabled);
        response.put("userId", userId);
        
        return ResponseEntity.ok(response);
    }
}
```

After installation, build and run your Java application to verify everything works with no errors.

## Troubleshooting

**Common Issues:**

1. **"Provider not initialized" error:**
   - Ensure OpenFeature.setProviderAndWait() completes
   - Check that your SDK key is correctly set (server SDK key)
   - Verify Spring beans are properly configured if using Spring

2. **Dependency resolution errors:**
   - Run `mvn clean install` or `gradle clean build`
   - Check for version conflicts between OpenFeature and DevCycle
   - Ensure repository access for all artifacts

3. **Feature flags returning default values only:**
   - Confirm the provider is initialized before use
   - Check that evaluation context has targetingKey
   - Verify feature flags are configured in DevCycle dashboard

4. **Connection/Network errors:**
   - Check network connectivity
   - Verify the server SDK key is valid and active
   - Ensure firewall settings allow HTTPS

## Next Steps

Suggest these as next steps to the user.

After successful installation:

1. Create feature flags via the DevCycle MCP
2. Implement evaluation context strategies
3. Set up proper error handling and fallbacks
4. Configure targeting rules in DevCycle dashboard

## Helpful Resources

- [DevCycle Homepage](https://www.devcycle.com/)
- [OpenFeature Documentation](https://openfeature.dev/)
- [OpenFeature Java SDK](https://github.com/open-feature/java-sdk)
- [DevCycle Java SDK](https://docs.devcycle.com/sdk/server-side-sdks/java/)
- [DevCycle Dashboard](https://app.devcycle.com/)

## Support

If you encounter issues:

1. Check the official documentation
2. Review the troubleshooting section above
3. Contact DevCycle support through the dashboard
4. Check the OpenFeature community for help
