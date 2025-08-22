# DevCycle Python SDK Installation Prompt

You are helping to install and configure the DevCycle Python SDK in a Python server application. Follow this complete guide to successfully integrate DevCycle feature flags. Do not install any Variables as part of this process, the user can ask for you to do that later.

**Do not use the SDK for:**

- Client-side applications (use appropriate client SDKs instead)
- JavaScript/Node.js applications (use Node.js SDK instead)
- Mobile applications (use iOS/Android SDKs instead)

If you detect that the user is trying to have you install the Python SDK in an application where it will not work, please stop what you are doing and advise the user which SDK they should be using.

## Required Information

Before proceeding, use your own analysis, the DevCycle MCP or web search to ensure you have:

- [ ] A DevCycle account and project set up
- [ ] A Development environment **Server SDK Key** (starts with `dvc_server_`)
- [ ] Python 3.7+ installed
- [ ] pip or poetry package manager
- [ ] The most recent DevCycle Python SDK version to install

**Security Note:** Use a SERVER SDK key for Python backend applications. Never expose server keys to client-side code. Store keys in environment variables.

## SDK Key Configuration

**IMPORTANT:** After obtaining the SDK key, you must set it up properly:

1. **First, attempt to create an environment file** (.env) in the project root:

   ```bash
   # .env
   DEVCYCLE_SERVER_SDK_KEY=your_server_sdk_key_here
   ```

   And ensure python-dotenv is configured if not already present.

2. **If you cannot create or modify environment files** (due to system restrictions or security policies), ask the user:

   - "I'm unable to create/modify environment files. Would you like me to:
     a) Temporarily hardcode the SDK key for testing purposes (you'll need to update it later for production)
     b) Provide you with the SDK key and instructions so you can set it up yourself?"

3. **Based on the user's response:**
   - If they choose hardcoding: Add a clear comment indicating this is temporary and should be replaced with environment variables
   - If they choose manual setup: Provide them with the SDK key and clear instructions on how to set up the environment variable

**Note:** Always prefer environment variables over hardcoding for security reasons.

## Installation Steps

### 1. Install the DevCycle Python SDK

```bash
# Using pip
pip install devcycle-python-server-sdk

# Using poetry
poetry add devcycle-python-server-sdk
```

### 2. Initialize DevCycle in Your Application

Create a DevCycle initialization module (e.g., `devcycle_config.py`):

```python
import os
from devcycle_python_server_sdk import DevCycleLocalClient

# Global client instance
devcycle_client = None

def init_devcycle():
    """Initialize the DevCycle client."""
    global devcycle_client

    if devcycle_client is None:
        sdk_key = os.environ.get('DEVCYCLE_SERVER_SDK_KEY', '<DEVCYCLE_SERVER_SDK_KEY>')

        # Initialize the client
        devcycle_client = DevCycleLocalClient(sdk_key)
        print("DevCycle initialized successfully")

    return devcycle_client

def get_devcycle_client():
    """Get the initialized DevCycle client."""
    if devcycle_client is None:
        raise RuntimeError("DevCycle client not initialized. Call init_devcycle() first.")
    return devcycle_client
```

### 3. Initialize DevCycle on Application Startup

#### For Flask Applications

```python
from flask import Flask
from devcycle_config import init_devcycle

app = Flask(__name__)

# Initialize DevCycle when the app starts
@app.before_first_request
def initialize_services():
    init_devcycle()

@app.route('/')
def hello():
    return 'Flask app with DevCycle!'

if __name__ == '__main__':
    # Initialize DevCycle before running the app
    init_devcycle()
    app.run(debug=True)
```

#### For Django Applications

In your Django settings or AppConfig:

```python
# myapp/apps.py
from django.apps import AppConfig
from devcycle_config import init_devcycle

class MyAppConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'myapp'

    def ready(self):
        # Initialize DevCycle when Django starts
        init_devcycle()
```

#### For FastAPI Applications

```python
from fastapi import FastAPI
from contextlib import asynccontextmanager
from devcycle_config import init_devcycle

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup
    init_devcycle()
    yield
    # Shutdown (cleanup if needed)

app = FastAPI(lifespan=lifespan)

@app.get("/")
def read_root():
    return {"message": "FastAPI with DevCycle!"}
```

### 4. Using DevCycle in Your Application

Example of using DevCycle to check feature flags:

```python
from devcycle_config import get_devcycle_client
from devcycle_python_server_sdk import DevCycleUser

def check_feature_for_user(user_id, email=None):
    """Check if a feature is enabled for a specific user."""
    client = get_devcycle_client()

    # Create user object
    user = DevCycleUser(
        user_id=user_id,
        email=email,
        custom_data={
            'plan': 'premium',
            'role': 'admin'
        }
    )

    # Check feature flag value
    feature_enabled = client.variable_value(
        user,
        'feature-key',
        False  # default value
    )

    return feature_enabled

# Example usage in a route
@app.route('/api/feature')
def feature_endpoint():
    user_id = request.user.id if hasattr(request, 'user') else 'anonymous'

    if check_feature_for_user(user_id):
        return {'feature': 'enabled'}
    else:
        return {'feature': 'disabled'}
```

### 5. Environment Configuration

Create a `.env` file for your environment variables:

```bash
# .env
DEVCYCLE_SERVER_SDK_KEY=your_server_sdk_key_here
ENVIRONMENT=development
```

Install python-dotenv if not already installed:

```bash
pip install python-dotenv
```

Load environment variables in your main application:

```python
from dotenv import load_dotenv

# Load environment variables
load_dotenv()
```

After installation, run your Python application and verify everything works with no errors.

## Troubleshooting

**Common Issues:**

1. **"DevCycle client not initialized" error:**

   - Ensure `init_devcycle()` is called on application startup
   - Check that your SDK key is correctly set (server SDK key)
   - Verify environment variables are loaded

2. **Import errors:**

   - Ensure the SDK is installed: `pip list | grep devcycle`
   - Check Python version compatibility (3.7+)
   - Verify virtual environment is activated if using one

3. **Environment variable not found:**

   - Ensure `.env` file is in the project root
   - Check that python-dotenv is installed and loaded
   - Verify the SDK key environment variable name matches

4. **Connection/Network errors:**
   - Check network connectivity
   - Verify the server SDK key is valid and active
   - Ensure firewall/proxy settings allow outbound HTTPS

## Next Steps

Suggest these as next steps to the user.

After successful installation:

1. Set up user identification logic for your application
2. **Wait for user guidance** before creating any feature flags or DevCycle Variables - do not create them proactively
3. Implement proper error handling for feature flag evaluations if needed
4. Help set up targeting rules for different user segments when requested

## Helpful Resources

- [DevCycle Homepage](https://www.devcycle.com/)
- [DevCycle Documentation](https://docs.devcycle.com/)
- [Python SDK Documentation](https://docs.devcycle.com/sdk/server-side-sdks/python/)
- [DevCycle Dashboard](https://app.devcycle.com/)
- [Python SDK GitHub Repository](https://github.com/DevCycleHQ/python-server-sdk)
- [Feature Flag Best Practices](https://docs.devcycle.com/best-practices/)

## Support

If you encounter issues:

1. Check the official documentation
2. Review the troubleshooting section above
3. Contact DevCycle support through the dashboard
4. Check the GitHub repository for known issues
