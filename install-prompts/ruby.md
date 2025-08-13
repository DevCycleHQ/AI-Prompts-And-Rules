# DevCycle Ruby SDK Installation Prompt

You are helping to install and configure the DevCycle Ruby SDK in a Ruby server application. Follow this complete guide to successfully integrate DevCycle feature flags. Do not install any Variables as part of this process, the user can ask for you to do that later.

**IMPORTANT: First detect the project configuration:**
- Check if it's a Rails application or plain Ruby
- Identify the Ruby version (requires Ruby 2.7+)
- Check for Gemfile and bundler usage

**Do not use the SDK for:**
- Client-side applications (use appropriate client SDKs instead)
- JavaScript/Node.js applications (use Node.js SDK instead)
- Mobile applications (use iOS/Android SDKs instead)

If you detect that the user is trying to have you install the Ruby SDK in an application where it will not work, please stop what you are doing and advise the user which SDK they should be using.

## Required Information

Before proceeding, use your own analysis, the DevCycle MCP or web search to ensure you have:
- [ ] A DevCycle account and project set up
- [ ] A Development environment **Server SDK Key** (starts with `dvc_server_`)
- [ ] Ruby 2.7+ installed
- [ ] Bundler gem installed
- [ ] The most recent DevCycle Ruby SDK version to install

**Security Note:** Use a SERVER SDK key for Ruby backend applications. Never expose server keys to client-side code. Store keys in environment variables or Rails credentials.

## Installation Steps

### 1. Install the DevCycle Ruby SDK

Add to your `Gemfile`:

```ruby
gem 'devcycle-ruby-server-sdk'
```

Then run:
```bash
bundle install
```

Or install directly:
```bash
gem install devcycle-ruby-server-sdk
```

### 2. Create DevCycle Configuration

Create a DevCycle initialization file (e.g., `config/initializers/devcycle.rb` for Rails):

```ruby
require 'devcycle'

module DevCycleConfig
  class << self
    attr_reader :client
    
    def initialize_client
      sdk_key = ENV['DEVCYCLE_SERVER_SDK_KEY'] || '<DEVCYCLE_SERVER_SDK_KEY>'
      
      if sdk_key.nil? || sdk_key.empty?
        raise 'DevCycle SDK key is not configured'
      end
      
      @client = DevCycle::Client.new(sdk_key)
      Rails.logger.info('DevCycle initialized successfully') if defined?(Rails)
      @client
    rescue => e
      Rails.logger.error("Failed to initialize DevCycle: #{e.message}") if defined?(Rails)
      raise
    end
    
    def client
      @client ||= initialize_client
    end
    
    def shutdown
      @client&.close
      @client = nil
    end
  end
end

# Initialize on Rails startup
if defined?(Rails)
  Rails.application.config.after_initialize do
    DevCycleConfig.client
  end
  
  # Clean up on shutdown
  at_exit do
    DevCycleConfig.shutdown
  end
end
```

### 3. Framework-Specific Integration

#### For Rails Applications

Create an initializer (`config/initializers/devcycle.rb`):

```ruby
require 'devcycle'

Rails.application.config.devcycle = DevCycle::Client.new(
  Rails.application.credentials.devcycle[:server_sdk_key] ||
  ENV['DEVCYCLE_SERVER_SDK_KEY']
)

# Optional: Create a helper module
module DevCycleHelper
  def devcycle_client
    Rails.application.config.devcycle
  end
  
  def feature_enabled?(key, user_id, default = false)
    user = DevCycle::User.new(
      user_id: user_id,
      email: current_user&.email,
      custom_data: {
        plan: current_user&.plan,
        role: current_user&.role
      }
    )
    
    devcycle_client.variable_value(user, key, default)
  rescue => e
    Rails.logger.error "DevCycle error: #{e.message}"
    default
  end
end

# Include in ApplicationController
class ApplicationController < ActionController::Base
  include DevCycleHelper
end
```

#### For Sinatra Applications

```ruby
require 'sinatra'
require 'devcycle'

class MyApp < Sinatra::Base
  configure do
    set :devcycle, DevCycle::Client.new(ENV['DEVCYCLE_SERVER_SDK_KEY'])
  end
  
  helpers do
    def devcycle_client
      settings.devcycle
    end
    
    def feature_enabled?(key, user_id, default = false)
      user = DevCycle::User.new(user_id: user_id)
      devcycle_client.variable_value(user, key, default)
    end
  end
  
  get '/' do
    'Sinatra app with DevCycle!'
  end
end
```

### 4. Using DevCycle in Your Application

Example service class:

```ruby
class FeatureService
  def initialize(devcycle_client = DevCycleConfig.client)
    @devcycle_client = devcycle_client
  end
  
  def feature_enabled_for_user?(feature_key, user_id, email = nil)
    user = DevCycle::User.new(
      user_id: user_id,
      email: email,
      custom_data: {
        plan: 'premium',
        role: 'admin'
      }
    )
    
    @devcycle_client.variable_value(user, feature_key, false)
  rescue => e
    Rails.logger.error "Error checking feature #{feature_key}: #{e.message}"
    false
  end
end
```

Example in a Rails controller:

```ruby
class FeaturesController < ApplicationController
  def check
    user_id = current_user&.id || 'anonymous'
    
    feature_enabled = feature_enabled?('new-feature', user_id)
    
    render json: { feature_enabled: feature_enabled }
  end
  
  def show
    @feature_service = FeatureService.new
    
    if @feature_service.feature_enabled_for_user?('premium-feature', current_user.id)
      render :premium_view
    else
      render :standard_view
    end
  end
end
```

### 5. Environment Configuration

Create environment configuration:

#### Using dotenv for Rails/Ruby

Add to `Gemfile`:
```ruby
gem 'dotenv-rails', groups: [:development, :test]
```

Create `.env` file:
```bash
# .env
DEVCYCLE_SERVER_SDK_KEY=your_server_sdk_key_here
RAILS_ENV=development
```

#### Using Rails Credentials

```bash
rails credentials:edit
```

Add:
```yaml
devcycle:
  server_sdk_key: your_server_sdk_key_here
```

After installation, run your Ruby application and verify everything works with no errors.

## Troubleshooting

**Common Issues:**

1. **"DevCycle client not initialized" error:**
   - Ensure the client is properly initialized on application startup
   - Check that your SDK key is correctly set (server SDK key)
   - Verify Rails initializers are loaded if using Rails

2. **Gem installation errors:**
   - Run `bundle update` to update dependencies
   - Check Ruby version compatibility (2.7+)
   - Clear bundler cache: `bundle clean --force`

3. **Environment variable not found:**
   - Ensure `.env` file is in the project root
   - Check that dotenv-rails is properly configured
   - Verify Rails credentials are decrypted correctly

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
- [Ruby SDK Documentation](https://docs.devcycle.com/sdk/server-side-sdks/ruby/)
- [DevCycle Dashboard](https://app.devcycle.com/)
- [Ruby SDK GitHub Repository](https://github.com/DevCycleHQ/ruby-server-sdk)
- [Feature Flag Best Practices](https://docs.devcycle.com/best-practices/)

## Support

If you encounter issues:
1. Check the official documentation
2. Review the troubleshooting section above
3. Contact DevCycle support through the dashboard
4. Check the GitHub repository for known issues
