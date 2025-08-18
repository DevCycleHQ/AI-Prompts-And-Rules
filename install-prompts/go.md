# DevCycle Go SDK Installation Prompt

You are helping to install and configure the DevCycle Go SDK in a Go server application. Follow this complete guide to successfully integrate DevCycle feature flags. Do not install any Variables as part of this process, the user can ask for you to do that later.

**Do not use the SDK for:**

- Client-side applications (use appropriate client SDKs instead)
- JavaScript/Node.js applications (use Node.js SDK instead)
- Mobile applications (use iOS/Android SDKs instead)

If you detect that the user is trying to have you install the Go SDK in an application where it will not work, please stop what you are doing and advise the user which SDK they should be using.

## Required Information

Before proceeding, use your own analysis, the DevCycle MCP or web search to ensure you have:

- [ ] A DevCycle account and project set up
- [ ] A Development environment **Server SDK Key** (starts with `dvc_server_`)
- [ ] Go 1.16+ installed
- [ ] Go modules enabled in your project
- [ ] The most recent DevCycle Go SDK version to install

**Security Note:** Use a SERVER SDK key for Go backend applications. Never expose server keys to client-side code. Store keys in environment variables.

## Installation Steps

### 1. Install the DevCycle Go SDK

```bash
go get github.com/devcyclehq/go-server-sdk/v2
```

### 2. Initialize DevCycle in Your Application

Create a DevCycle configuration file (e.g., `devcycle/client.go`):

```go
package devcycle

import (
    "fmt"
    "log"
    "os"
    "sync"
    "time"

    devcycle "github.com/devcyclehq/go-server-sdk/v2"
)

var (
    client *devcycle.Client
    once   sync.Once
    mu     sync.RWMutex
)

// InitializeDevCycle initializes the DevCycle client
func InitializeDevCycle() error {
    var initErr error
    
    once.Do(func() {
        sdkKey := os.Getenv("DEVCYCLE_SERVER_SDK_KEY")
        if sdkKey == "" {
            sdkKey = "<DEVCYCLE_SERVER_SDK_KEY>" // Replace with your server SDK key
        }

        // Create client configuration
        config := devcycle.ClientConfig{
            SDKKey:              sdkKey,
            EnableCloudBucketing: false,
            EnableEdgeDB:        false,
        }

        // Initialize the client
        var err error
        client, err = devcycle.NewClient(config)
        if err != nil {
            initErr = fmt.Errorf("failed to initialize DevCycle: %w", err)
            return
        }

        log.Println("DevCycle initialized successfully")
    })

    return initErr
}

// GetClient returns the initialized DevCycle client
func GetClient() (*devcycle.Client, error) {
    mu.RLock()
    defer mu.RUnlock()
    
    if client == nil {
        return nil, fmt.Errorf("DevCycle client not initialized")
    }
    return client, nil
}

// Close cleanly shuts down the DevCycle client
func Close() error {
    mu.Lock()
    defer mu.Unlock()
    
    if client != nil {
        return client.Close()
    }
    return nil
}
```

### 3. Initialize DevCycle on Application Startup

In your main application file:

```go
package main

import (
    "log"
    "net/http"
    "os"
    "os/signal"
    "syscall"
    
    "your-module/devcycle"
)

func main() {
    // Initialize DevCycle
    if err := devcycle.InitializeDevCycle(); err != nil {
        log.Fatal("Failed to initialize DevCycle:", err)
    }
    
    // Ensure clean shutdown
    defer func() {
        if err := devcycle.Close(); err != nil {
            log.Printf("Error closing DevCycle client: %v", err)
        }
    }()
    
    // Set up your HTTP server
    http.HandleFunc("/", homeHandler)
    http.HandleFunc("/api/feature", featureHandler)
    
    // Start server
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
    w.Write([]byte("Go server with DevCycle!"))
}
```

### 4. Using DevCycle in Your Handlers

Example of using DevCycle to check feature flags:

```go
package main

import (
    "encoding/json"
    "net/http"
    
    devcycle "github.com/devcyclehq/go-server-sdk/v2"
    "your-module/devcycle"
)

func featureHandler(w http.ResponseWriter, r *http.Request) {
    client, err := devcycle.GetClient()
    if err != nil {
        http.Error(w, "DevCycle not available", http.StatusInternalServerError)
        return
    }
    
    // Create user from request context (example)
    user := devcycle.User{
        UserId: "user-123", // Get from auth context
        Email:  "user@example.com",
        CustomData: map[string]interface{}{
            "plan": "premium",
            "role": "admin",
        },
    }
    
    // Check feature flag value
    featureEnabled, err := client.VariableValue(
        user,
        "feature-key",
        false, // default value
    )
    if err != nil {
        http.Error(w, "Error checking feature", http.StatusInternalServerError)
        return
    }
    
    response := map[string]interface{}{
        "featureEnabled": featureEnabled,
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}
```

### 5. Environment Configuration

Create a `.env` file for local development:

```bash
# .env
DEVCYCLE_SERVER_SDK_KEY=your_server_sdk_key_here
GO_ENV=development
```

Load environment variables using a package like godotenv:

```bash
go get github.com/joho/godotenv
```

```go
package main

import (
    "log"
    _ "github.com/joho/godotenv/autoload"
)

func main() {
    // Environment variables are now loaded
    // ... rest of your application
}
```

After installation, run your Go application and verify everything works with no errors.

## Troubleshooting

**Common Issues:**

1. **"DevCycle client not initialized" error:**
   - Ensure `InitializeDevCycle()` is called on application startup
   - Check that your SDK key is correctly set (server SDK key)
   - Verify environment variables are loaded

2. **Import errors:**
   - Run `go mod tidy` to ensure dependencies are downloaded
   - Check Go version compatibility (1.16+)
   - Verify the import path is correct

3. **Environment variable not found:**
   - Ensure `.env` file is in the project root
   - Check that godotenv is properly configured
   - Verify the SDK key environment variable name matches

4. **Connection/Network errors:**
   - Check network connectivity
   - Verify the server SDK key is valid and active
   - Ensure firewall/proxy settings allow outbound HTTPS

## Next Steps

Suggest these as next steps to the user.

After successful installation:

1. Set up user identification logic for your application
2. Create your first feature flag via the DevCycle MCP and use it in your handlers
3. Implement proper error handling for feature flag evaluations
4. Set up targeting rules for different user segments

## Helpful Resources

- [DevCycle Homepage](https://www.devcycle.com/)
- [DevCycle Documentation](https://docs.devcycle.com/)
- [Go SDK Documentation](https://docs.devcycle.com/sdk/server-side-sdks/go/)
- [DevCycle Dashboard](https://app.devcycle.com/)
- [Go SDK GitHub Repository](https://github.com/DevCycleHQ/go-server-sdk)
- [Feature Flag Best Practices](https://docs.devcycle.com/best-practices/)

## Support

If you encounter issues:

1. Check the official documentation
2. Review the troubleshooting section above
3. Contact DevCycle support through the dashboard
4. Check the GitHub repository for known issues
