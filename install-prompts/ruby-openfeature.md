# DevCycle Ruby OpenFeature Provider Installation Prompt

You are helping to install and configure the DevCycle OpenFeature Provider for Ruby server applications. Follow this complete guide to successfully integrate DevCycle feature flags using the OpenFeature standard. Do not install any Variables as part of this process, the user can ask for you to do that later.

**Do not use this for:**

- Client-side applications (use appropriate client SDKs instead)
- JavaScript/Node.js applications (use Node.js SDK instead)
- Mobile applications (use iOS/Android SDKs instead)

## Required Information

Before proceeding, use your own analysis, the DevCycle MCP or web search to ensure you have:

- [ ] A DevCycle account and project set up
- [ ] A Development environment **Server SDK Key** (starts with `dvc_server_`)
- [ ] Ruby 2.7+ installed
- [ ] Bundler gem installed
- [ ] The most recent OpenFeature and DevCycle provider versions

**Security Note:** Use a SERVER SDK key for Ruby backend applications. Never expose server keys to client-side code. Store keys in environment variables or Rails credentials.

## Installation Steps

### 1. Install OpenFeature SDK and DevCycle Provider

Add to your `Gemfile`:

```ruby
gem 'openfeature-sdk'
gem 'devcycle-ruby-server-sdk'
gem 'devcycle-ruby-server-sdk-openfeature'
```

Then run:

```bash
bundle install
```

Or install directly:

```bash
gem install openfeature-sdk
gem install devcycle-ruby-server-sdk
gem install devcycle-ruby-server-sdk-openfeature
```

### 2. Initialize OpenFeature with DevCycle Provider

Create an OpenFeature configuration file (e.g., `config/initializers/openfeature.rb` for Rails):

```ruby
require 'openfeature'
require 'devcycle'
require 'devcycle/openfeature'

module OpenFeatureConfig
  class << self
    attr_reader :client
    
    def initialize_client
      sdk_key = ENV['DEVCYCLE_SERVER_SDK_KEY']
      
      if sdk_key.nil? || sdk_key.empty?
        raise 'DEVCYCLE_SERVER_SDK_KEY environment variable is not set'
      end
      
      begin
        # Initialize DevCycle client
        devcycle_client = DevCycle::Client.new(sdk_key)
        
        # Create DevCycle provider
        provider = DevCycle::OpenFeature::Provider.new(devcycle_client)
        
        # Set the provider for OpenFeature
        OpenFeature::SDK.configure do |config|
          config.provider = provider
        end
        
        # Get OpenFeature client
        @client = OpenFeature::SDK.build_client
        
        Rails.logger.info('OpenFeature with DevCycle initialized successfully') if defined?(Rails)
        @client
      rescue => e
        Rails.logger.error("Failed to initialize OpenFeature: #{e.message}") if defined?(Rails)
        raise
      end
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
    OpenFeatureConfig.client
  end
  
  # Clean up on shutdown
  at_exit do
    OpenFeatureConfig.shutdown
  end
end
```

### 3. Framework-Specific Integration

#### For Rails Applications

Create a concern for controllers:

```ruby
# app/controllers/concerns/feature_flaggable.rb
module FeatureFlaggable
  extend ActiveSupport::Concern
  
  included do
    helper_method :feature_enabled? if respond_to?(:helper_method)
  end
  
  def openfeature_client
    @openfeature_client ||= OpenFeatureConfig.client
  end
  
  def feature_enabled?(feature_key, default = false)
    context = build_evaluation_context
    openfeature_client.get_boolean_value(feature_key, default, context)
  rescue => e
    Rails.logger.error "Error checking feature #{feature_key}: #{e.message}"
    default
  end
  
  def get_string_config(config_key, default = '')
    context = build_evaluation_context
    openfeature_client.get_string_value(config_key, default, context)
  end
  
  def get_number_config(config_key, default = 0)
    context = build_evaluation_context
    openfeature_client.get_number_value(config_key, default, context)
  end
  
  def get_object_config(config_key, default = {})
    context = build_evaluation_context
    openfeature_client.get_object_value(config_key, default, context)
  end
  
  private
  
  def build_evaluation_context
    OpenFeature::SDK::EvaluationContext.new(
      targeting_key: current_user&.id || 'anonymous',
      attributes: {
        email: current_user&.email,
        plan: current_user&.subscription_plan,
        role: current_user&.role,
        authenticated: user_signed_in?
      }.compact
    )
  end
end

# Include in ApplicationController
class ApplicationController < ActionController::Base
  include FeatureFlaggable
end
```

#### For Sinatra Applications

```ruby
require 'sinatra'
require 'openfeature'
require_relative 'openfeature_config'

class MyApp < Sinatra::Base
  configure do
    set :openfeature_client, OpenFeatureConfig.client
  end
  
  helpers do
    def feature_enabled?(feature_key, default = false)
      context = OpenFeature::SDK::EvaluationContext.new(
        targeting_key: session[:user_id] || 'anonymous',
        attributes: {
          authenticated: !session[:user_id].nil?
        }
      )
      
      settings.openfeature_client.get_boolean_value(feature_key, default, context)
    end
  end
  
  get '/api/feature/:key' do
    feature_key = params[:key]
    enabled = feature_enabled?(feature_key)
    
    content_type :json
    { feature: feature_key, enabled: enabled }.to_json
  end
end
```

### 4. Using Feature Flags with OpenFeature

Service class example:

```ruby
class FeatureService
  def initialize(openfeature_client = OpenFeatureConfig.client)
    @client = openfeature_client
  end
  
  def feature_enabled_for_user?(feature_key, user_id, email = nil)
    context = OpenFeature::SDK::EvaluationContext.new(
      targeting_key: user_id,
      attributes: {
        email: email,
        plan: 'premium',
        role: 'admin'
      }.compact
    )
    
    @client.get_boolean_value(feature_key, false, context)
  rescue => e
    Rails.logger.error "Error checking feature #{feature_key}: #{e.message}"
    false
  end
  
  def get_all_flags_for_user(user_id)
    context = OpenFeature::SDK::EvaluationContext.new(
      targeting_key: user_id
    )
    
    {
      new_feature: @client.get_boolean_value('new-feature', false, context),
      button_text: @client.get_string_value('button-text', 'Click Here', context),
      max_items: @client.get_number_value('max-items', 10, context),
      ui_config: @client.get_object_value('ui-config', { theme: 'light' }, context)
    }
  end
end
```

Rails controller example:

```ruby
class FeaturesController < ApplicationController
  def check
    feature_key = params[:feature_key]
    
    # Using the concern method
    enabled = feature_enabled?(feature_key)
    
    # Or using different value types
    text = get_string_config("#{feature_key}_text", 'Default Text')
    limit = get_number_config("#{feature_key}_limit", 100)
    config = get_object_config("#{feature_key}_config", { enabled: false })
    
    render json: {
      feature_enabled: enabled,
      text: text,
      limit: limit,
      config: config
    }
  end
  
  def user_features
    service = FeatureService.new
    features = service.get_all_flags_for_user(current_user&.id || 'anonymous')
    
    render json: features
  end
end
```

### 5. Environment Configuration

Using dotenv for Rails/Ruby:

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

Using Rails credentials:

```bash
rails credentials:edit
```

Add:

```yaml
devcycle:
  server_sdk_key: your_server_sdk_key_here
```

Then update initialization:

```ruby
sdk_key = Rails.application.credentials.devcycle[:server_sdk_key] ||
          ENV['DEVCYCLE_SERVER_SDK_KEY']
```

After installation, run your Ruby application and verify everything works with no errors.

## Troubleshooting

**Common Issues:**

1. **"Provider not initialized" error:**
   - Ensure OpenFeature provider is set during initialization
   - Check that DevCycle client initializes successfully
   - Verify your SDK key is correct (server SDK key)

2. **Gem installation errors:**
   - Run `bundle update` to update dependencies
   - Check Ruby version compatibility (2.7+)
   - Clear bundler cache: `bundle clean --force`

3. **Feature flags returning default values only:**
   - Confirm the provider is initialized before use
   - Check that evaluation context has targeting_key
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
- [OpenFeature Ruby SDK](https://github.com/open-feature/ruby-sdk)
- [DevCycle Ruby SDK](https://docs.devcycle.com/sdk/server-side-sdks/ruby/)
- [DevCycle Dashboard](https://app.devcycle.com/)

## Support

If you encounter issues:

1. Check the official documentation
2. Review the troubleshooting section above
3. Contact DevCycle support through the dashboard
4. Check the OpenFeature community for help
