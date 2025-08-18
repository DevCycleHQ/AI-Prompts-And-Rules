# DevCycle Go OpenFeature Provider Installation Prompt

You are helping to install and configure the DevCycle OpenFeature Provider for Go server applications. Follow this complete guide to successfully integrate DevCycle feature flags using the OpenFeature standard. Do not install any Variables as part of this process, the user can ask for you to do that later.

**Do not use this for:**

- Client-side applications (use appropriate client SDKs instead)
- Applications already using the DevCycle Go SDK directly

## Required Information

Before proceeding, use your own analysis, the DevCycle MCP or web search to ensure you have:

- [ ] A DevCycle account and project set up
- [ ] A Development environment **Server SDK Key** (starts with `dvc_server_`)
- [ ] Go 1.19+ installed
- [ ] Go modules enabled in your project
- [ ] The most recent OpenFeature and DevCycle provider versions

**Security Note:** Use a SERVER SDK key for Go backend applications. Never expose server keys to client-side code. Store keys in environment variables.

## Installation Steps

### 1. Install OpenFeature SDK and DevCycle Provider

```bash
go get github.com/open-feature/go-sdk
go get github.com/devcyclehq/go-server-sdk/v2
go get github.com/devcyclehq/go-server-sdk/v2/openfeature
```

### 2. Initialize OpenFeature with DevCycle Provider

Create an OpenFeature configuration file (e.g., `openfeature/config.go`):

```go
package openfeature

import (
    "context"
    "fmt"
    "log"
    "os"
    "sync"
    
    "github.com/open-feature/go-sdk/openfeature"
    devcycle "github.com/devcyclehq/go-server-sdk/v2"
    devcycleprovider "github.com/devcyclehq/go-server-sdk/v2/openfeature"
)

var (
    client *openfeature.Client
    once   sync.Once
    mu     sync.RWMutex
)

// Initialize sets up OpenFeature with DevCycle provider
func Initialize(ctx context.Context) error {
    var initErr error
    
    once.Do(func() {
        sdkKey := os.Getenv("DEVCYCLE_SERVER_SDK_KEY")
        if sdkKey == "" {
            initErr = fmt.Errorf("DEVCYCLE_SERVER_SDK_KEY is not set")
            return
        }
        
        // Create DevCycle client
        config := devcycle.ClientConfig{
            SDKKey: sdkKey,
        }
        
        devcycleClient, err := devcycle.NewClient(config)
        if err != nil {
            initErr = fmt.Errorf("failed to create DevCycle client: %w", err)
            return
        }
        
        // Create DevCycle provider for OpenFeature
        provider := devcycleprovider.NewProvider(devcycleClient)
        
        // Set the provider
        openfeature.SetProvider(provider)
        
        // Create OpenFeature client
        client = openfeature.NewClient("my-app")
        
        log.Println("OpenFeature with DevCycle initialized successfully")
    })
    
    return initErr
}

// GetClient returns the OpenFeature client
func GetClient() (*openfeature.Client, error) {
    mu.RLock()
    defer mu.RUnlock()
    
    if client == nil {
        return nil, fmt.Errorf("OpenFeature client not initialized")
    }
    return client, nil
}
```

### 3. Initialize on Application Startup

In your main application:

```go
package main

import (
    "context"
    "log"
    "net/http"
    "os"
    "os/signal"
    "syscall"
    
    "your-module/openfeature"
)

func main() {
    ctx := context.Background()
    
    // Initialize OpenFeature with DevCycle
    if err := openfeature.Initialize(ctx); err != nil {
        log.Fatal("Failed to initialize OpenFeature:", err)
    }
    
    // Set up HTTP server
    http.HandleFunc("/", homeHandler)
    http.HandleFunc("/api/feature", featureHandler)
    
    server := &http.Server{Addr: ":8080"}
    
    // Handle graceful shutdown
    go func() {
        sigChan := make(chan os.Signal, 1)
        signal.Notify(sigChan, syscall.SIGINT, syscall.SIGTERM)
        <-sigChan
        
        log.Println("Shutting down server...")
        server.Close()
    }()
    
    log.Println("Server starting on :8080")
    if err := server.ListenAndServe(); err != http.ErrServerClosed {
        log.Fatal("Server failed:", err)
    }
}

func homeHandler(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("Go server with OpenFeature and DevCycle!"))
}
```

### 4. Using Feature Flags with OpenFeature

```go
package main

import (
    "context"
    "encoding/json"
    "net/http"
    
    "github.com/open-feature/go-sdk/openfeature"
    "your-module/openfeature"
)

func featureHandler(w http.ResponseWriter, r *http.Request) {
    client, err := openfeature.GetClient()
    if err != nil {
        http.Error(w, "OpenFeature not available", http.StatusInternalServerError)
        return
    }
    
    // Set evaluation context
    ctx := openfeature.NewEvaluationContext(
        "user-123", // targetingKey
        map[string]interface{}{
            "email": "user@example.com",
            "plan": "premium",
            "role": "admin",
        },
    )
    
    // Get boolean value
    featureEnabled, err := client.BooleanValue(
        context.Background(),
        "new-feature",
        false,
        ctx,
    )
    if err != nil {
        http.Error(w, "Error evaluating feature", http.StatusInternalServerError)
        return
    }
    
    // Get string value
    buttonText, _ := client.StringValue(
        context.Background(),
        "button-text",
        "Click Here",
        ctx,
    )
    
    // Get number value
    rateLimit, _ := client.FloatValue(
        context.Background(),
        "rate-limit",
        100.0,
        ctx,
    )
    
    // Get object value
    config, _ := client.ObjectValue(
        context.Background(),
        "ui-config",
        map[string]interface{}{"theme": "light"},
        ctx,
    )
    
    response := map[string]interface{}{
        "featureEnabled": featureEnabled,
        "buttonText": buttonText,
        "rateLimit": rateLimit,
        "config": config,
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}
```

### 5. Service Pattern Example

```go
package services

import (
    "context"
    
    "github.com/open-feature/go-sdk/openfeature"
)

type FeatureService struct {
    client *openfeature.Client
}

func NewFeatureService() (*FeatureService, error) {
    client, err := openfeature.GetClient()
    if err != nil {
        return nil, err
    }
    
    return &FeatureService{
        client: client,
    }, nil
}

func (s *FeatureService) IsFeatureEnabledForUser(userID string, featureKey string) (bool, error) {
    ctx := openfeature.NewEvaluationContext(
        userID,
        map[string]interface{}{},
    )
    
    return s.client.BooleanValue(
        context.Background(),
        featureKey,
        false,
        ctx,
    )
}

func (s *FeatureService) GetConfigForUser(userID string, configKey string) (map[string]interface{}, error) {
    ctx := openfeature.NewEvaluationContext(
        userID,
        map[string]interface{}{},
    )
    
    return s.client.ObjectValue(
        context.Background(),
        configKey,
        map[string]interface{}{},
        ctx,
    )
}
```

### 6. Environment Configuration

Create a `.env` file:

```bash
# .env
DEVCYCLE_SERVER_SDK_KEY=your_server_sdk_key_here
GO_ENV=development
```

Load with godotenv:

```bash
go get github.com/joho/godotenv
```

```go
import _ "github.com/joho/godotenv/autoload"
```

After installation, run your Go application and verify everything works with no errors.

## Troubleshooting

**Common Issues:**

1. **"Provider not initialized" error:**
   - Ensure OpenFeature.SetProvider() is called
   - Check that DevCycle client initializes successfully
   - Verify your SDK key is correct (server SDK key)

2. **Feature flags returning default values only:**
   - Confirm the provider is initialized before use
   - Check that evaluation context has targetingKey
   - Verify feature flags are configured in DevCycle dashboard

3. **Import errors:**
   - Run `go mod tidy` to resolve dependencies
   - Check Go version compatibility (1.19+)
   - Verify import paths are correct

4. **Connection/Network errors:**
   - Check network connectivity
   - Verify the server SDK key is valid
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
- [OpenFeature Go SDK](https://github.com/open-feature/go-sdk)
- [DevCycle Go SDK](https://docs.devcycle.com/sdk/server-side-sdks/go/)
- [DevCycle Dashboard](https://app.devcycle.com/)

## Support

If you encounter issues:

1. Check the official documentation
2. Review the troubleshooting section above
3. Contact DevCycle support through the dashboard
4. Check the OpenFeature community for help
