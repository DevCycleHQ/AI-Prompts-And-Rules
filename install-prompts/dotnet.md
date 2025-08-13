# DevCycle .NET SDK Installation Prompt

You are helping to install and configure the DevCycle .NET SDK in a .NET server application. Follow this complete guide to successfully integrate DevCycle feature flags. Do not install any Variables as part of this process, the user can ask for you to do that later.

**IMPORTANT: First detect the project configuration:**
- Check the .NET version (.NET Core 3.1+ or .NET 5+)
- Identify the project type: ASP.NET Core, Console App, or other
- Check the package manager: NuGet Package Manager, .NET CLI, or PackageReference

**Do not use the SDK for:**
- Client-side Blazor WebAssembly (use appropriate client SDK approach)
- Unity games (consider Unity-specific patterns)
- Mobile applications (use iOS/Android SDKs instead)

If you detect that the user is trying to have you install the .NET SDK in an application where it will not work, please stop what you are doing and advise the user which SDK they should be using.

## Required Information

Before proceeding, use your own analysis, the DevCycle MCP or web search to ensure you have:
- [ ] A DevCycle account and project set up
- [ ] A Development environment **Server SDK Key** (starts with `dvc_server_`)
- [ ] .NET Core 3.1+ or .NET 5+ installed
- [ ] Visual Studio, VS Code, or .NET CLI available
- [ ] The most recent DevCycle .NET SDK version to install

**Security Note:** Use a SERVER SDK key for .NET backend applications. Never expose server keys to client-side code. Store keys in appsettings.json, environment variables, or Azure Key Vault.

## Installation Steps

### 1. Install the DevCycle .NET SDK

#### Using .NET CLI

```bash
dotnet add package DevCycle.SDK.Server.Local
```

#### Using Package Manager Console

```powershell
Install-Package DevCycle.SDK.Server.Local
```

#### Using PackageReference

Add to your `.csproj` file:

```xml
<ItemGroup>
  <PackageReference Include="DevCycle.SDK.Server.Local" Version="2.*" />
</ItemGroup>
```

### 2. Configure DevCycle in Your Application

#### For ASP.NET Core Applications

In `Program.cs` (.NET 6+):

```csharp
using DevCycle.SDK.Server.Local.Api;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container
builder.Services.AddControllers();

// Configure DevCycle
builder.Services.AddSingleton<IDevCycleClient>(serviceProvider =>
{
    var configuration = serviceProvider.GetRequiredService<IConfiguration>();
    var sdkKey = configuration["DevCycle:ServerSdkKey"] 
        ?? Environment.GetEnvironmentVariable("DEVCYCLE_SERVER_SDK_KEY")
        ?? throw new InvalidOperationException("DevCycle SDK key is not configured");
    
    var client = new DevCycleLocalClient(sdkKey);
    return client;
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
using DevCycle.SDK.Server.Local.Api;
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
        
        // Configure DevCycle
        services.AddSingleton<IDevCycleClient>(serviceProvider =>
        {
            var sdkKey = Configuration["DevCycle:ServerSdkKey"] 
                ?? Environment.GetEnvironmentVariable("DEVCYCLE_SERVER_SDK_KEY");
                
            if (string.IsNullOrEmpty(sdkKey))
            {
                throw new InvalidOperationException("DevCycle SDK key is not configured");
            }
            
            return new DevCycleLocalClient(sdkKey);
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

For production, use `appsettings.Production.json` or environment variables:

```json
{
  "DevCycle": {
    "ServerSdkKey": "${DEVCYCLE_SERVER_SDK_KEY}"
  }
}
```

### 4. Using DevCycle in Your Application

Create a feature service:

```csharp
using DevCycle.SDK.Server.Common.Model;
using DevCycle.SDK.Server.Local.Api;

public interface IFeatureService
{
    Task<bool> IsFeatureEnabledAsync(string featureKey, string userId, string email = null);
}

public class FeatureService : IFeatureService
{
    private readonly IDevCycleClient _devCycleClient;
    private readonly ILogger<FeatureService> _logger;

    public FeatureService(IDevCycleClient devCycleClient, ILogger<FeatureService> logger)
    {
        _devCycleClient = devCycleClient;
        _logger = logger;
    }

    public async Task<bool> IsFeatureEnabledAsync(string featureKey, string userId, string email = null)
    {
        try
        {
            var user = new DevCycleUser(userId)
            {
                Email = email,
                CustomData = new Dictionary<string, object>
                {
                    { "plan", "premium" },
                    { "role", "admin" }
                }
            };

            var variable = await _devCycleClient.VariableAsync(user, featureKey, false);
            return variable.Value;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error checking feature {FeatureKey} for user {UserId}", featureKey, userId);
            return false; // Return default value on error
        }
    }
}
```

Register the service:

```csharp
builder.Services.AddScoped<IFeatureService, FeatureService>();
```

Example controller:

```csharp
using Microsoft.AspNetCore.Mvc;

[ApiController]
[Route("api/[controller]")]
public class FeaturesController : ControllerBase
{
    private readonly IFeatureService _featureService;

    public FeaturesController(IFeatureService featureService)
    {
        _featureService = featureService;
    }

    [HttpGet("check/{featureKey}")]
    public async Task<IActionResult> CheckFeature(string featureKey)
    {
        var userId = User.Identity?.Name ?? "anonymous";
        var email = User.Claims.FirstOrDefault(c => c.Type == "email")?.Value;
        
        var isEnabled = await _featureService.IsFeatureEnabledAsync(featureKey, userId, email);
        
        return Ok(new { featureEnabled = isEnabled });
    }
}
```

### 5. Clean Shutdown

Ensure proper cleanup on application shutdown:

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        var host = CreateHostBuilder(args).Build();
        
        var lifetime = host.Services.GetRequiredService<IHostApplicationLifetime>();
        lifetime.ApplicationStopping.Register(() =>
        {
            var devCycleClient = host.Services.GetService<IDevCycleClient>();
            devCycleClient?.Dispose();
        });
        
        host.Run();
    }
}
```

After installation, build and run your .NET application to verify everything works with no errors.

## Troubleshooting

**Common Issues:**

1. **"DevCycle client not initialized" error:**
   - Ensure the client is registered in dependency injection
   - Check that your SDK key is correctly set (server SDK key)
   - Verify configuration is loaded properly

2. **NuGet package installation errors:**
   - Clear NuGet cache: `dotnet nuget locals all --clear`
   - Update NuGet package sources
   - Check .NET version compatibility

3. **Configuration not found:**
   - Ensure appsettings.json is copied to output directory
   - Check environment-specific configuration files
   - Verify environment variables are set

4. **Connection/Network errors:**
   - Check network connectivity
   - Verify the server SDK key is valid and active
   - Ensure firewall/proxy settings allow outbound HTTPS

## Next Steps

Suggest these as next steps to the user.

After successful installation:
1. Set up user identification logic for your application
2. Create your first feature flag via the DevCycle MCP and use it in your controllers
3. Implement proper error handling and logging
4. Set up targeting rules for different user segments

## Helpful Resources

- [DevCycle Homepage](https://www.devcycle.com/)
- [DevCycle Documentation](https://docs.devcycle.com/)
- [.NET SDK Documentation](https://docs.devcycle.com/sdk/server-side-sdks/dotnet/)
- [DevCycle Dashboard](https://app.devcycle.com/)
- [.NET SDK GitHub Repository](https://github.com/DevCycleHQ/dotnet-server-sdk)
- [Feature Flag Best Practices](https://docs.devcycle.com/best-practices/)

## Support

If you encounter issues:
1. Check the official documentation
2. Review the troubleshooting section above
3. Contact DevCycle support through the dashboard
4. Check the GitHub repository for known issues
