# DevCycle PHP OpenFeature Provider Installation Prompt

You are helping to install and configure the DevCycle OpenFeature Provider for PHP server applications. Follow this complete guide to successfully integrate DevCycle feature flags using the OpenFeature standard. Do not install any Variables as part of this process, the user can ask for you to do that later.

**Do not use this for:**

- Client-side applications (use appropriate client SDKs instead)
- JavaScript/Node.js applications (use Node.js SDK instead)
- Mobile applications (use iOS/Android SDKs instead)

## Required Information

Before proceeding, use your own analysis, the DevCycle MCP or web search to ensure you have:

- [ ] A DevCycle account and project set up
- [ ] A Development environment **Server SDK Key** (starts with `dvc_server_`)
- [ ] PHP 7.4+ installed
- [ ] Composer package manager installed
- [ ] The most recent OpenFeature and DevCycle provider versions

**Security Note:** Use a SERVER SDK key for PHP backend applications. Never expose server keys to client-side code. Store keys in environment variables or configuration files.

## Installation Steps

### 1. Install OpenFeature SDK and DevCycle Provider

```bash
composer require open-feature/sdk
composer require devcycle/php-server-sdk
composer require devcycle/php-server-sdk-openfeature
```

### 2. Initialize OpenFeature with DevCycle Provider

Create an OpenFeature configuration file (e.g., `openfeature_config.php`):

```php
<?php
namespace App\Config;

use OpenFeature\OpenFeatureAPI;
use OpenFeature\implementation\flags\EvaluationContext;
use OpenFeature\implementation\flags\Attributes;
use DevCycle\Sdk\DevCycleClient;
use DevCycle\OpenFeature\DevCycleProvider;

class OpenFeatureConfig {
    private static ?OpenFeatureAPI $api = null;
    private static ?DevCycleClient $devCycleClient = null;
    
    public static function initialize(): void {
        if (self::$api !== null) {
            return;
        }
        
        $sdkKey = $_ENV['DEVCYCLE_SERVER_SDK_KEY'] ?? null;
        
        if (empty($sdkKey)) {
            throw new \RuntimeException('DEVCYCLE_SERVER_SDK_KEY is not configured');
        }
        
        try {
            // Initialize DevCycle client
            self::$devCycleClient = new DevCycleClient($sdkKey);
            
            // Create DevCycle provider
            $provider = new DevCycleProvider(self::$devCycleClient);
            
            // Get OpenFeature API instance and set provider
            self::$api = OpenFeatureAPI::getInstance();
            self::$api->setProvider($provider);
            
            error_log('OpenFeature with DevCycle initialized successfully');
        } catch (\Exception $e) {
            error_log('Failed to initialize OpenFeature: ' . $e->getMessage());
            throw $e;
        }
    }
    
    public static function getClient(): \OpenFeature\interfaces\flags\FlagValueResolver {
        if (self::$api === null) {
            throw new \RuntimeException('OpenFeature not initialized. Call initialize() first.');
        }
        
        return self::$api->getClient();
    }
    
    public static function shutdown(): void {
        if (self::$devCycleClient !== null) {
            self::$devCycleClient->close();
            self::$devCycleClient = null;
        }
    }
}
```

### 3. Framework-Specific Integration

#### For Laravel Applications

Create a service provider:

```php
<?php
namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use OpenFeature\OpenFeatureAPI;
use OpenFeature\interfaces\flags\FlagValueResolver;
use DevCycle\Sdk\DevCycleClient;
use DevCycle\OpenFeature\DevCycleProvider;

class OpenFeatureServiceProvider extends ServiceProvider
{
    public function register()
    {
        $this->app->singleton(FlagValueResolver::class, function ($app) {
            $sdkKey = config('services.devcycle.server_sdk_key');
            
            if (empty($sdkKey)) {
                throw new \RuntimeException('DevCycle SDK key is not configured');
            }
            
            // Initialize DevCycle client
            $devCycleClient = new DevCycleClient($sdkKey);
            
            // Create DevCycle provider
            $provider = new DevCycleProvider($devCycleClient);
            
            // Set provider
            $api = OpenFeatureAPI::getInstance();
            $api->setProvider($provider);
            
            return $api->getClient();
        });
    }
    
    public function boot()
    {
        // Optional: Close client on application shutdown
        $this->app->terminating(function () {
            // Cleanup if needed
        });
    }
}
```

Register the provider in `config/app.php`:

```php
'providers' => [
    // ...
    App\Providers\OpenFeatureServiceProvider::class,
],
```

#### For Symfony Applications

Create a service configuration:

```yaml
# config/services.yaml
services:
    DevCycle\Sdk\DevCycleClient:
        arguments:
            - '%env(DEVCYCLE_SERVER_SDK_KEY)%'
    
    DevCycle\OpenFeature\DevCycleProvider:
        arguments:
            - '@DevCycle\Sdk\DevCycleClient'
    
    OpenFeature\OpenFeatureAPI:
        factory: ['OpenFeature\OpenFeatureAPI', 'getInstance']
        calls:
            - setProvider: ['@DevCycle\OpenFeature\DevCycleProvider']
    
    openfeature.client:
        factory: ['@OpenFeature\OpenFeatureAPI', 'getClient']
```

#### For Plain PHP Applications

Initialize in your bootstrap file:

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
require_once __DIR__ . '/openfeature_config.php';

use App\Config\OpenFeatureConfig;

// Load environment variables
$dotenv = Dotenv\Dotenv::createImmutable(__DIR__);
$dotenv->load();

// Initialize OpenFeature
OpenFeatureConfig::initialize();

// Register shutdown function
register_shutdown_function(function() {
    OpenFeatureConfig::shutdown();
});
```

### 4. Using Feature Flags with OpenFeature

```php
<?php
use OpenFeature\implementation\flags\EvaluationContext;
use OpenFeature\implementation\flags\Attributes;
use App\Config\OpenFeatureConfig;

class FeatureService {
    private $client;
    
    public function __construct() {
        $this->client = OpenFeatureConfig::getClient();
    }
    
    public function isFeatureEnabled($userId, $featureKey) {
        // Create evaluation context
        $attributes = new Attributes([
            'email' => 'user@example.com',
            'plan' => 'premium',
            'role' => 'admin'
        ]);
        
        $context = new EvaluationContext($userId, $attributes);
        
        // Get boolean value
        return $this->client->getBooleanValue($featureKey, false, $context);
    }
    
    public function getStringConfig($userId, $configKey) {
        $context = new EvaluationContext($userId);
        return $this->client->getStringValue($configKey, 'default', $context);
    }
    
    public function getNumberConfig($userId, $configKey) {
        $context = new EvaluationContext($userId);
        return $this->client->getIntegerValue($configKey, 100, $context);
    }
    
    public function getObjectConfig($userId, $configKey) {
        $context = new EvaluationContext($userId);
        
        $defaultValue = [
            'theme' => 'light',
            'fontSize' => 14
        ];
        
        return $this->client->getObjectValue($configKey, $defaultValue, $context);
    }
}
```

### 5. Laravel Controller Example

```php
<?php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use OpenFeature\interfaces\flags\FlagValueResolver;
use OpenFeature\implementation\flags\EvaluationContext;
use OpenFeature\implementation\flags\Attributes;

class FeatureController extends Controller
{
    private $openFeatureClient;
    
    public function __construct(FlagValueResolver $openFeatureClient)
    {
        $this->openFeatureClient = $openFeatureClient;
    }
    
    public function checkFeature(Request $request, $featureKey)
    {
        $userId = $request->user()->id ?? 'anonymous';
        
        // Create evaluation context
        $attributes = new Attributes([
            'email' => $request->user()->email ?? null,
            'authenticated' => $request->user() !== null
        ]);
        
        $context = new EvaluationContext($userId, $attributes);
        
        // Get different types of values
        $boolValue = $this->openFeatureClient->getBooleanValue($featureKey, false, $context);
        $stringValue = $this->openFeatureClient->getStringValue("{$featureKey}_text", 'default', $context);
        $numberValue = $this->openFeatureClient->getIntegerValue("{$featureKey}_limit", 100, $context);
        
        return response()->json([
            'featureEnabled' => $boolValue,
            'text' => $stringValue,
            'limit' => $numberValue
        ]);
    }
}
```

### 6. Environment Configuration

Create a `.env` file:

```bash
# .env
DEVCYCLE_SERVER_SDK_KEY=your_server_sdk_key_here
APP_ENV=development
```

After installation, run your PHP application and verify everything works with no errors.

## Troubleshooting

**Common Issues:**

1. **"Provider not initialized" error:**
   - Ensure OpenFeatureAPI::setProvider() is called
   - Check that DevCycle client initializes successfully
   - Verify your SDK key is correct (server SDK key)

2. **Composer installation errors:**
   - Run `composer update` to update dependencies
   - Check PHP version compatibility (7.4+)
   - Clear Composer cache: `composer clear-cache`

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
3. Set up proper error handling and logging
4. Configure targeting rules in DevCycle dashboard

## Helpful Resources

- [DevCycle Homepage](https://www.devcycle.com/)
- [OpenFeature Documentation](https://openfeature.dev/)
- [OpenFeature PHP SDK](https://github.com/open-feature/php-sdk)
- [DevCycle PHP SDK](https://docs.devcycle.com/sdk/server-side-sdks/php/)
- [DevCycle Dashboard](https://app.devcycle.com/)

## Support

If you encounter issues:

1. Check the official documentation
2. Review the troubleshooting section above
3. Contact DevCycle support through the dashboard
4. Check the OpenFeature community for help
