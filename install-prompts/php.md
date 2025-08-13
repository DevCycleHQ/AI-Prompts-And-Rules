# DevCycle PHP SDK Installation Prompt

You are helping to install and configure the DevCycle PHP SDK in a PHP server application. Follow this complete guide to successfully integrate DevCycle feature flags. Do not install any Variables as part of this process, the user can ask for you to do that later.

**IMPORTANT: First detect the project configuration:**
- Check if using Composer for dependency management
- Identify the framework: Laravel, Symfony, Slim, or plain PHP
- Check PHP version (requires PHP 7.4+)

**Do not use the SDK for:**
- Client-side applications (use appropriate client SDKs instead)
- JavaScript/Node.js applications (use Node.js SDK instead)
- Mobile applications (use iOS/Android SDKs instead)

If you detect that the user is trying to have you install the PHP SDK in an application where it will not work, please stop what you are doing and advise the user which SDK they should be using.

## Required Information

Before proceeding, use your own analysis, the DevCycle MCP or web search to ensure you have:
- [ ] A DevCycle account and project set up
- [ ] A Development environment **Server SDK Key** (starts with `dvc_server_`)
- [ ] PHP 7.4+ installed
- [ ] Composer package manager installed
- [ ] The most recent DevCycle PHP SDK version to install

**Security Note:** Use a SERVER SDK key for PHP backend applications. Never expose server keys to client-side code. Store keys in environment variables or configuration files.

## Installation Steps

### 1. Install the DevCycle PHP SDK

```bash
composer require devcycle/php-server-sdk
```

### 2. Create DevCycle Configuration

Create a DevCycle configuration file (e.g., `devcycle.php` or in your config directory):

```php
<?php
namespace App\Config;

use DevCycle\Sdk\DevCycleClient;
use DevCycle\Model\DevCycleUser;

class DevCycleConfig {
    private static ?DevCycleClient $client = null;
    
    public static function getClient(): DevCycleClient {
        if (self::$client === null) {
            self::initialize();
        }
        return self::$client;
    }
    
    private static function initialize(): void {
        $sdkKey = $_ENV['DEVCYCLE_SERVER_SDK_KEY'] ?? '<DEVCYCLE_SERVER_SDK_KEY>';
        
        if (empty($sdkKey)) {
            throw new \RuntimeException('DevCycle SDK key is not configured');
        }
        
        try {
            self::$client = new DevCycleClient($sdkKey);
            error_log('DevCycle initialized successfully');
        } catch (\Exception $e) {
            error_log('Failed to initialize DevCycle: ' . $e->getMessage());
            throw $e;
        }
    }
    
    public static function shutdown(): void {
        if (self::$client !== null) {
            self::$client->close();
            self::$client = null;
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
use DevCycle\Sdk\DevCycleClient;

class DevCycleServiceProvider extends ServiceProvider
{
    public function register()
    {
        $this->app->singleton(DevCycleClient::class, function ($app) {
            $sdkKey = config('services.devcycle.server_sdk_key');
            
            if (empty($sdkKey)) {
                throw new \RuntimeException('DevCycle SDK key is not configured');
            }
            
            return new DevCycleClient($sdkKey);
        });
    }
    
    public function boot()
    {
        // Optional: Close client on application shutdown
        $this->app->terminating(function () {
            if ($this->app->bound(DevCycleClient::class)) {
                $this->app->make(DevCycleClient::class)->close();
            }
        });
    }
}
```

Register the provider in `config/app.php`:

```php
'providers' => [
    // ...
    App\Providers\DevCycleServiceProvider::class,
],
```

Add to `config/services.php`:

```php
'devcycle' => [
    'server_sdk_key' => env('DEVCYCLE_SERVER_SDK_KEY'),
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
        public: true
```

#### For Plain PHP Applications

Initialize in your bootstrap file:

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
require_once __DIR__ . '/config/devcycle.php';

use App\Config\DevCycleConfig;

// Load environment variables (if using phpdotenv)
$dotenv = Dotenv\Dotenv::createImmutable(__DIR__);
$dotenv->load();

// Initialize DevCycle
$devcycleClient = DevCycleConfig::getClient();

// Register shutdown function
register_shutdown_function(function() {
    DevCycleConfig::shutdown();
});
```

### 4. Using DevCycle in Your Application

Example of using DevCycle to check feature flags:

```php
<?php
use DevCycle\Model\DevCycleUser;

class FeatureService {
    private $devcycleClient;
    
    public function __construct($devcycleClient) {
        $this->devcycleClient = $devcycleClient;
    }
    
    public function isFeatureEnabled($userId, $email = null) {
        // Create user object
        $user = new DevCycleUser([
            'user_id' => $userId,
            'email' => $email,
            'custom_data' => [
                'plan' => 'premium',
                'role' => 'admin'
            ]
        ]);
        
        // Check feature flag value
        $variable = $this->devcycleClient->variableValue(
            $user,
            'feature-key',
            false // default value
        );
        
        return $variable;
    }
}
```

Example in a Laravel controller:

```php
<?php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use DevCycle\Sdk\DevCycleClient;
use DevCycle\Model\DevCycleUser;

class FeatureController extends Controller
{
    private $devcycleClient;
    
    public function __construct(DevCycleClient $devcycleClient)
    {
        $this->devcycleClient = $devcycleClient;
    }
    
    public function checkFeature(Request $request)
    {
        $userId = $request->user()->id ?? 'anonymous';
        
        $user = new DevCycleUser([
            'user_id' => $userId,
            'email' => $request->user()->email ?? null
        ]);
        
        $featureEnabled = $this->devcycleClient->variableValue(
            $user,
            'feature-key',
            false
        );
        
        return response()->json([
            'featureEnabled' => $featureEnabled
        ]);
    }
}
```

### 5. Environment Configuration

Create a `.env` file for your environment variables:

```bash
# .env
DEVCYCLE_SERVER_SDK_KEY=your_server_sdk_key_here
APP_ENV=development
```

For plain PHP, install phpdotenv if needed:
```bash
composer require vlucas/phpdotenv
```

After installation, run your PHP application and verify everything works with no errors.

## Troubleshooting

**Common Issues:**

1. **"DevCycle client not initialized" error:**
   - Ensure the client is properly initialized on application startup
   - Check that your SDK key is correctly set (server SDK key)
   - Verify service provider is registered if using Laravel

2. **Composer installation errors:**
   - Run `composer update` to update dependencies
   - Check PHP version compatibility (7.4+)
   - Clear Composer cache: `composer clear-cache`

3. **Environment variable not found:**
   - Ensure `.env` file is in the project root
   - Check that phpdotenv is configured if using it
   - Verify the SDK key environment variable name matches

4. **Connection/Network errors:**
   - Check network connectivity
   - Verify the server SDK key is valid and active
   - Ensure firewall/proxy settings allow outbound HTTPS

## Next Steps

Suggest these as next steps to the user.

After successful installation:
1. Set up user identification logic for your application
2. Create your first feature flag via the DevCycle MCP and use it in your controllers
3. Implement proper error handling for feature flag evaluations
4. Set up targeting rules for different user segments

## Helpful Resources

- [DevCycle Homepage](https://www.devcycle.com/)
- [DevCycle Documentation](https://docs.devcycle.com/)
- [PHP SDK Documentation](https://docs.devcycle.com/sdk/server-side-sdks/php/)
- [DevCycle Dashboard](https://app.devcycle.com/)
- [PHP SDK GitHub Repository](https://github.com/DevCycleHQ/php-server-sdk)
- [Feature Flag Best Practices](https://docs.devcycle.com/best-practices/)

## Support

If you encounter issues:
1. Check the official documentation
2. Review the troubleshooting section above
3. Contact DevCycle support through the dashboard
4. Check the GitHub repository for known issues
