# DevCycle Python OpenFeature Provider Installation Prompt

You are helping to install and configure the DevCycle OpenFeature Provider for Python server applications. Follow this complete guide to successfully integrate DevCycle feature flags using the OpenFeature standard. Do not install any Variables as part of this process, the user can ask for you to do that later.

**Do not use this for:**

- Client-side applications (use appropriate client SDKs instead)
- JavaScript/Node.js applications (use Node.js SDK instead)
- Mobile applications (use iOS/Android SDKs instead)

## Required Information

Before proceeding, use your own analysis, the DevCycle MCP or web search to ensure you have:

- [ ] A DevCycle account and project set up
- [ ] A Development environment **Server SDK Key** (starts with `dvc_server_`)
- [ ] Python 3.8+ installed
- [ ] pip or poetry package manager
- [ ] The most recent OpenFeature and DevCycle provider versions

**Security Note:** Use a SERVER SDK key for Python backend applications. Never expose server keys to client-side code. Store keys in environment variables.

## Installation Steps

### 1. Install OpenFeature SDK and DevCycle Provider

```bash
# Using pip
pip install openfeature-sdk devcycle-python-server-sdk devcycle-python-server-sdk-openfeature

# Using poetry
poetry add openfeature-sdk devcycle-python-server-sdk devcycle-python-server-sdk-openfeature
```

### 2. Initialize OpenFeature with DevCycle Provider

Create an OpenFeature configuration module (e.g., `openfeature_config.py`):

```python
import os
from openfeature import api
from openfeature.client import OpenFeatureClient
from openfeature.evaluation_context import EvaluationContext
from devcycle_python_server_sdk import DevCycleLocalClient
from devcycle_python_server_sdk.openfeature import DevCycleProvider

# Global client instance
openfeature_client: OpenFeatureClient = None

def init_openfeature():
    """Initialize OpenFeature with DevCycle provider."""
    global openfeature_client
    
    if openfeature_client is not None:
        return openfeature_client
    
    sdk_key = os.environ.get('DEVCYCLE_SERVER_SDK_KEY')
    if not sdk_key:
        raise ValueError("DEVCYCLE_SERVER_SDK_KEY environment variable is not set")
    
    try:
        # Initialize DevCycle client
        devcycle_client = DevCycleLocalClient(sdk_key)
        
        # Create DevCycle provider
        provider = DevCycleProvider(devcycle_client)
        
        # Set the provider for OpenFeature
        api.set_provider(provider)
        
        # Get OpenFeature client
        openfeature_client = api.get_client()
        
        print("OpenFeature with DevCycle initialized successfully")
        return openfeature_client
    
    except Exception as e:
        print(f"Failed to initialize OpenFeature: {e}")
        raise

def get_openfeature_client() -> OpenFeatureClient:
    """Get the initialized OpenFeature client."""
    if openfeature_client is None:
        raise RuntimeError("OpenFeature client not initialized. Call init_openfeature() first.")
    return openfeature_client
```

### 3. Framework-Specific Integration

#### For Flask Applications

```python
from flask import Flask, request, jsonify
from openfeature_config import init_openfeature, get_openfeature_client
from openfeature.evaluation_context import EvaluationContext

app = Flask(__name__)

# Initialize OpenFeature on app startup
@app.before_first_request
def initialize_services():
    init_openfeature()

@app.route('/api/feature/<feature_key>')
def check_feature(feature_key):
    client = get_openfeature_client()
    
    # Create evaluation context
    user_id = request.headers.get('X-User-Id', 'anonymous')
    context = EvaluationContext(
        targeting_key=user_id,
        attributes={
            'email': request.headers.get('X-User-Email'),
            'plan': 'premium',
            'role': 'admin'
        }
    )
    
    # Get feature flag value
    feature_enabled = client.get_boolean_value(
        flag_key=feature_key,
        default_value=False,
        evaluation_context=context
    )
    
    return jsonify({
        'feature_key': feature_key,
        'enabled': feature_enabled,
        'user_id': user_id
    })

if __name__ == '__main__':
    init_openfeature()
    app.run(debug=True)
```

#### For Django Applications

In your Django app configuration:

```python
# myapp/apps.py
from django.apps import AppConfig
from openfeature_config import init_openfeature

class MyAppConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'myapp'
    
    def ready(self):
        # Initialize OpenFeature when Django starts
        init_openfeature()
```

Django view example:

```python
# myapp/views.py
from django.http import JsonResponse
from openfeature_config import get_openfeature_client
from openfeature.evaluation_context import EvaluationContext

def check_feature(request, feature_key):
    client = get_openfeature_client()
    
    # Create evaluation context
    user_id = request.user.id if request.user.is_authenticated else 'anonymous'
    context = EvaluationContext(
        targeting_key=str(user_id),
        attributes={
            'email': request.user.email if request.user.is_authenticated else None,
            'authenticated': request.user.is_authenticated
        }
    )
    
    # Get different types of feature flags
    bool_value = client.get_boolean_value(feature_key, False, context)
    string_value = client.get_string_value(f"{feature_key}_text", "default", context)
    number_value = client.get_integer_value(f"{feature_key}_limit", 100, context)
    
    return JsonResponse({
        'boolean': bool_value,
        'string': string_value,
        'number': number_value
    })
```

#### For FastAPI Applications

```python
from fastapi import FastAPI, Header
from contextlib import asynccontextmanager
from openfeature_config import init_openfeature, get_openfeature_client
from openfeature.evaluation_context import EvaluationContext

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup
    init_openfeature()
    yield
    # Shutdown (cleanup if needed)

app = FastAPI(lifespan=lifespan)

@app.get("/api/feature/{feature_key}")
async def check_feature(
    feature_key: str,
    user_id: str = Header(default="anonymous", alias="X-User-Id"),
    user_email: str = Header(default=None, alias="X-User-Email")
):
    client = get_openfeature_client()
    
    # Create evaluation context
    context = EvaluationContext(
        targeting_key=user_id,
        attributes={
            'email': user_email,
            'plan': 'premium',
            'role': 'admin'
        }
    )
    
    # Get feature flag values
    feature_enabled = client.get_boolean_value(feature_key, False, context)
    button_text = client.get_string_value(f"{feature_key}_text", "Click Here", context)
    rate_limit = client.get_integer_value(f"{feature_key}_limit", 100, context)
    
    # Get object value
    default_config = {'theme': 'light', 'fontSize': 14}
    ui_config = client.get_object_value(f"{feature_key}_config", default_config, context)
    
    return {
        "feature_enabled": feature_enabled,
        "button_text": button_text,
        "rate_limit": rate_limit,
        "ui_config": ui_config
    }
```

### 4. Service Class Example

```python
from openfeature_config import get_openfeature_client
from openfeature.evaluation_context import EvaluationContext
from typing import Any, Dict

class FeatureService:
    def __init__(self):
        self.client = get_openfeature_client()
    
    def is_feature_enabled_for_user(self, user_id: str, feature_key: str) -> bool:
        """Check if a feature is enabled for a specific user."""
        context = EvaluationContext(
            targeting_key=user_id,
            attributes={}
        )
        
        return self.client.get_boolean_value(feature_key, False, context)
    
    def get_config_for_user(self, user_id: str, config_key: str) -> Dict[str, Any]:
        """Get configuration object for a user."""
        context = EvaluationContext(
            targeting_key=user_id,
            attributes={}
        )
        
        default_config = {'enabled': False}
        return self.client.get_object_value(config_key, default_config, context)
    
    def get_all_flags_for_user(self, user_id: str, email: str = None) -> Dict[str, Any]:
        """Get all feature flag values for a user."""
        context = EvaluationContext(
            targeting_key=user_id,
            attributes={'email': email} if email else {}
        )
        
        # Example of getting multiple flag values
        return {
            'new_feature': self.client.get_boolean_value('new-feature', False, context),
            'button_text': self.client.get_string_value('button-text', 'Default', context),
            'api_limit': self.client.get_integer_value('api-limit', 100, context)
        }
```

### 5. Environment Configuration

Create a `.env` file:

```bash
# .env
DEVCYCLE_SERVER_SDK_KEY=your_server_sdk_key_here
ENVIRONMENT=development
```

Load with python-dotenv:

```python
from dotenv import load_dotenv
load_dotenv()
```

After installation, run your Python application and verify everything works with no errors.

## Troubleshooting

**Common Issues:**

1. **"Provider not initialized" error:**
   - Ensure `api.set_provider()` is called
   - Check that DevCycle client initializes successfully
   - Verify your SDK key is correct (server SDK key)

2. **Import errors:**
   - Ensure all packages are installed: `pip list | grep -E "openfeature|devcycle"`
   - Check Python version compatibility (3.8+)
   - Verify virtual environment is activated if using one

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
- [OpenFeature Python SDK](https://github.com/open-feature/python-sdk)
- [DevCycle Python SDK](https://docs.devcycle.com/sdk/server-side-sdks/python/)
- [DevCycle Dashboard](https://app.devcycle.com/)

## Support

If you encounter issues:

1. Check the official documentation
2. Review the troubleshooting section above
3. Contact DevCycle support through the dashboard
4. Check the OpenFeature community for help
