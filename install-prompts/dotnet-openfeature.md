# DevCycle .NET OpenFeature Provider Installation Prompt

You are helping to install and configure the DevCycle OpenFeature Provider for .NET server applications. Follow this complete guide to successfully integrate DevCycle feature flags using the OpenFeature standard. Do not install any Variables as part of this process, the user can ask for you to do that later.

**Do not use this for:**

- Client-side Blazor WebAssembly (use appropriate client SDK approach)
- Unity games (consider Unity-specific patterns)
- Mobile applications (use iOS/Android SDKs instead)

## Required Information

Before proceeding, use your own analysis, the DevCycle MCP or web search to ensure you have:

- [ ] A DevCycle account and project set up
- [ ] A Development environment **Server SDK Key** (starts with `dvc_server_`)
- [ ] .NET Core 3.1+ or .NET 5+ installed
- [ ] Visual Studio, VS Code, or .NET CLI available
- [ ] The most recent OpenFeature and DevCycle provider versions

**Security Note:** Use a SERVER SDK key for .NET backend applications. Never expose server keys to client-side code. Store keys in appsettings.json, environment variables, or Azure Key Vault.

## SDK Key Configuration

**IMPORTANT:** After obtaining the SDK key, you must set it up properly:

1. **First, attempt to add the SDK key to appsettings.json or appsettings.Development.json**:

   ```json
   {
     "DevCycle": {
       "ServerSdkKey": "your_server_sdk_key_here"
     }
   }
   ```

   Or use environment variables or user secrets for development.

2. **If you cannot create or modify configuration files** (due to system restrictions or security policies), ask the user:

   - "I'm unable to create/modify configuration files. Would you like me to:
     a) Temporarily hardcode the SDK key for testing purposes (you'll need to update it later for production)
     b) Provide you with the SDK key and instructions so you can set it up yourself?"

3. **Based on the user's response:**
   - If they choose hardcoding: Add a clear comment indicating this is temporary and should be replaced with configuration files
   - If they choose manual setup: Provide them with the SDK key and clear instructions on how to set it up

**Note:** Always prefer configuration files, user secrets, or Azure Key Vault over hardcoding for security reasons.

## Installation Steps

### 1. Install OpenFeature SDK and DevCycle Provider

#### Using .NET CLI

```bash
dotnet add package OpenFeature
dotnet add package DevCycle.SDK.Server.Local
dotnet add package DevCycle.SDK.Server.Local.OpenFeature
```

#### Using Package Manager Console

```powershell
Install-Package OpenFeature
Install-Package DevCycle.SDK.Server.Local
Install-Package DevCycle.SDK.Server.Local.OpenFeature
```

#### Using PackageReference

Add to your `.csproj` file:

```xml
<ItemGroup>
  <PackageReference Include="OpenFeature" Version="1.*" />
  <PackageReference Include="DevCycle.SDK.Server.Local" Version="2.*" />
  <PackageReference Include="DevCycle.SDK.Server.Local.OpenFeature" Version="2.*" />
</ItemGroup>
```

### 2. Configure OpenFeature with DevCycle Provider

#### For ASP.NET Core Applications

In `Program.cs` (.NET 6+):

```csharp
using OpenFeature;
using DevCycle.SDK.Server.Local.Api;
using DevCycle.SDK.Server.Local.OpenFeature;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container
builder.Services.AddControllers();

// Configure OpenFeature with DevCycle
builder.Services.AddSingleton(serviceProvider =>
{
    var configuration = serviceProvider.GetRequiredService<IConfiguration>();
    var sdkKey = configuration["DevCycle:ServerSdkKey"]
        ?? Environment.GetEnvironmentVariable("DEVCYCLE_SERVER_SDK_KEY")
        ?? throw new InvalidOperationException("DevCycle SDK key is not configured");

    // Initialize DevCycle client
    var devCycleClient = new DevCycleLocalClient(sdkKey);

    // Create DevCycle provider
    var provider = new DevCycleProvider(devCycleClient);

    // Set the provider for OpenFeature
    Api.Instance.SetProviderAsync(provider).GetAwaiter().GetResult();

    // Return the OpenFeature client
    return Api.Instance.GetClient();
});

var app = builder.Build();

// Configure the HTTP request pipeline
app.UseRouting();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

For older versions using `Startup.cs`:

```csharp
using OpenFeature;
using DevCycle.SDK.Server.Local.Api;
using DevCycle.SDK.Server.Local.OpenFeature;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

public class Startup
{
    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }

    public IConfiguration Configuration { get; }

    public void ConfigureServices(IServiceCollection services)
    {
        services.AddControllers();

        // Configure OpenFeature with DevCycle
        services.AddSingleton(serviceProvider =>
        {
            var sdkKey = Configuration["DevCycle:ServerSdkKey"]
                ?? Environment.GetEnvironmentVariable("DEVCYCLE_SERVER_SDK_KEY");

            if (string.IsNullOrEmpty(sdkKey))
            {
                throw new InvalidOperationException("DevCycle SDK key is not configured");
            }

            // Initialize DevCycle client
            var devCycleClient = new DevCycleLocalClient(sdkKey);

            // Create DevCycle provider
            var provider = new DevCycleProvider(devCycleClient);

            // Set the provider
            Api.Instance.SetProviderAsync(provider).GetAwaiter().GetResult();

            return Api.Instance.GetClient();
        });
    }

    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }

        app.UseRouting();
        app.UseAuthorization();
        app.UseEndpoints(endpoints =>
        {
            endpoints.MapControllers();
        });
    }
}
```

### 3. Application Configuration

Add to `appsettings.json`:

```json
{
  "DevCycle": {
    "ServerSdkKey": "your_server_sdk_key_here"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information"
    }
  }
}
```

### 4. Using Feature Flags with OpenFeature

Create a feature service:

```csharp
using OpenFeature;
using OpenFeature.Model;

public interface IFeatureService
{
    Task<bool> IsFeatureEnabledAsync(string featureKey, string userId);
    Task<string> GetStringConfigAsync(string configKey, string userId);
    Task<int> GetNumberConfigAsync(string configKey, string userId);
    Task<Structure> GetObjectConfigAsync(string configKey, string userId);
}

public class FeatureService : IFeatureService
{
    private readonly FeatureClient _openFeatureClient;
    private readonly ILogger<FeatureService> _logger;

    public FeatureService(FeatureClient openFeatureClient, ILogger<FeatureService> logger)
    {
        _openFeatureClient = openFeatureClient;
        _logger = logger;
    }

    public async Task<bool> IsFeatureEnabledAsync(string featureKey, string userId)
    {
        try
        {
            // Create evaluation context
            var context = EvaluationContext.Builder()
                .Set("targetingKey", userId)
                .Set("plan", "premium")
                .Set("role", "admin")
                .Build();

            // Get boolean value
            var result = await _openFeatureClient.GetBooleanValueAsync(featureKey, false, context);
            return result;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error checking feature {FeatureKey} for user {UserId}", featureKey, userId);
            return false;
        }
    }

    public async Task<string> GetStringConfigAsync(string configKey, string userId)
    {
        var context = EvaluationContext.Builder()
            .Set("targetingKey", userId)
            .Build();

        return await _openFeatureClient.GetStringValueAsync(configKey, "default", context);
    }

    public async Task<int> GetNumberConfigAsync(string configKey, string userId)
    {
        var context = EvaluationContext.Builder()
            .Set("targetingKey", userId)
            .Build();

        return await _openFeatureClient.GetIntegerValueAsync(configKey, 100, context);
    }

    public async Task<Structure> GetObjectConfigAsync(string configKey, string userId)
    {
        var context = EvaluationContext.Builder()
            .Set("targetingKey", userId)
            .Build();

        var defaultValue = new Structure(new Dictionary<string, Value>
        {
            { "theme", new Value("light") },
            { "fontSize", new Value(14) }
        });

        return await _openFeatureClient.GetObjectValueAsync(configKey, defaultValue, context);
    }
}
```

Register the service:

```csharp
builder.Services.AddScoped<IFeatureService, FeatureService>();
```

### 5. Controller Example

```csharp
using Microsoft.AspNetCore.Mvc;
using OpenFeature;
using OpenFeature.Model;

[ApiController]
[Route("api/[controller]")]
public class FeaturesController : ControllerBase
{
    private readonly FeatureClient _openFeatureClient;
    private readonly ILogger<FeaturesController> _logger;

    public FeaturesController(FeatureClient openFeatureClient, ILogger<FeaturesController> logger)
    {
        _openFeatureClient = openFeatureClient;
        _logger = logger;
    }

    [HttpGet("check/{featureKey}")]
    public async Task<IActionResult> CheckFeature(string featureKey)
    {
        var userId = User.Identity?.Name ?? "anonymous";

        var context = EvaluationContext.Builder()
            .Set("targetingKey", userId)
            .Set("email", User.Claims.FirstOrDefault(c => c.Type == "email")?.Value)
            .Build();

        var isEnabled = await _openFeatureClient.GetBooleanValueAsync(featureKey, false, context);

        return Ok(new { featureEnabled = isEnabled, userId });
    }
}
```

After installation, build and run your .NET application to verify everything works with no errors.

## Troubleshooting

**Common Issues:**

1. **"Provider not initialized" error:**

   - Ensure Api.Instance.SetProviderAsync() completes
   - Check that your SDK key is correctly set (server SDK key)
   - Verify dependency injection is configured properly

2. **NuGet package installation errors:**

   - Clear NuGet cache: `dotnet nuget locals all --clear`
   - Update NuGet package sources
   - Check .NET version compatibility

3. **Client returning default values only:**

   - Confirm the provider is initialized before use
   - Check that evaluation context has targetingKey

4. **Connection/Network errors:**
   - Check network connectivity
   - Verify the server SDK key is valid and active
   - Ensure firewall settings allow HTTPS

## Next Steps

Suggest these as next steps to the user.

After successful installation:

1. **Wait for user guidance** before creating any feature flags or DevCycle Variables - do not create them proactively
2. When requested, help implement evaluation context strategies
3. Set up proper error handling and logging
4. Help configure targeting rules in DevCycle dashboard when asked

## Helpful Resources

- [DevCycle Homepage](https://www.devcycle.com/)
- [OpenFeature Documentation](https://openfeature.dev/)
- [OpenFeature .NET SDK](https://github.com/open-feature/dotnet-sdk)
- [DevCycle .NET SDK](https://docs.devcycle.com/sdk/server-side-sdks/dotnet/)
- [DevCycle Dashboard](https://app.devcycle.com/)

## Support

If you encounter issues:

1. Check the official documentation
2. Review the troubleshooting section above
3. Contact DevCycle support through the dashboard
4. Check the OpenFeature community for help
