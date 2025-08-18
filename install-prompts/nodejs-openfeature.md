# DevCycle Node.js OpenFeature Provider Installation Prompt

You are helping to install and configure the DevCycle OpenFeature Provider for Node.js server applications. Follow this complete guide to successfully integrate DevCycle feature flags using the OpenFeature standard. Do not install any Variables as part of this process, the user can ask for you to do that later.

**Do not use this for:**

- Client-side JavaScript applications (use `javascript-openfeature.md` instead)
- React applications (use `react-openfeature.md` instead)
- NestJS applications (consider NestJS-specific patterns)

## Required Information

Before proceeding, use your own analysis, the DevCycle MCP or web search to ensure you have:

- [ ] A DevCycle account and project set up
- [ ] A Development environment **Server SDK Key** (starts with `dvc_server_`)
- [ ] Node.js 14+ installed
- [ ] npm, yarn, or pnpm package manager
- [ ] The most recent OpenFeature and DevCycle provider versions

**Security Note:** Use a SERVER SDK key for Node.js backend applications. Never expose server keys to client-side code. Store keys in environment variables.

## Installation Steps

### 1. Install OpenFeature SDK and DevCycle Provider

```bash
# Using npm
npm install --save @openfeature/server-sdk @devcycle/openfeature-nodejs-provider @devcycle/nodejs-server-sdk

# Using yarn
yarn add @openfeature/server-sdk @devcycle/openfeature-nodejs-provider @devcycle/nodejs-server-sdk

# Using pnpm
pnpm add @openfeature/server-sdk @devcycle/openfeature-nodejs-provider @devcycle/nodejs-server-sdk
```

### 2. Initialize OpenFeature with DevCycle Provider

Create an initialization file (e.g., `openfeature-config.js`):

```javascript
const { OpenFeature } = require('@openfeature/server-sdk');
const { DevCycleProvider } = require('@devcycle/openfeature-nodejs-provider');
const { initializeDevCycle } = require('@devcycle/nodejs-server-sdk');

let openFeatureClient;

async function initializeOpenFeature() {
  const sdkKey = process.env.DEVCYCLE_SERVER_SDK_KEY;
  
  if (!sdkKey) {
    throw new Error('DEVCYCLE_SERVER_SDK_KEY is not set');
  }
  
  // Initialize DevCycle client
  const devcycleClient = await initializeDevCycle(sdkKey).onClientInitialized();
  
  // Create DevCycle provider
  const provider = new DevCycleProvider(devcycleClient);
  
  // Set the provider for OpenFeature
  await OpenFeature.setProviderAndWait(provider);
  
  // Get the OpenFeature client
  openFeatureClient = OpenFeature.getClient();
  
  console.log('OpenFeature with DevCycle initialized successfully');
  return openFeatureClient;
}

function getOpenFeatureClient() {
  if (!openFeatureClient) {
    throw new Error('OpenFeature client not initialized');
  }
  return openFeatureClient;
}

module.exports = {
  initializeOpenFeature,
  getOpenFeatureClient
};
```

### 3. Initialize on Application Startup

In your main application file:

```javascript
const express = require('express');
const { initializeOpenFeature } = require('./openfeature-config');

const app = express();
const PORT = process.env.PORT || 3000;

async function startServer() {
  try {
    // Initialize OpenFeature with DevCycle
    await initializeOpenFeature();
    
    // Your application routes
    app.get('/', (req, res) => {
      res.send('Server with OpenFeature and DevCycle!');
    });
    
    app.listen(PORT, () => {
      console.log(`Server running on port ${PORT}`);
    });
  } catch (error) {
    console.error('Failed to start server:', error);
    process.exit(1);
  }
}

startServer();
```

### 4. Using Feature Flags in Your Application

```javascript
const { getOpenFeatureClient } = require('./openfeature-config');

// Example route using OpenFeature
app.get('/api/feature', async (req, res) => {
  const client = getOpenFeatureClient();
  
  // Set evaluation context for this request
  const context = {
    targetingKey: req.user?.id || 'anonymous',
    email: req.user?.email,
    name: req.user?.name,
    plan: req.user?.plan,
    role: req.user?.role
  };
  
  // Get feature flag values
  const featureEnabled = await client.getBooleanValue('new-feature', false, context);
  const buttonText = await client.getStringValue('button-text', 'Click Here', context);
  const rateLimit = await client.getNumberValue('rate-limit', 100, context);
  const config = await client.getObjectValue('ui-config', { theme: 'light' }, context);
  
  res.json({
    featureEnabled,
    buttonText,
    rateLimit,
    config
  });
});

// Service class example
class FeatureService {
  constructor() {
    this.client = getOpenFeatureClient();
  }
  
  async isFeatureEnabledForUser(userId, featureKey) {
    const context = {
      targetingKey: userId
    };
    
    return this.client.getBooleanValue(featureKey, false, context);
  }
  
  async getConfigForUser(userId, configKey, defaultValue) {
    const context = {
      targetingKey: userId
    };
    
    return this.client.getObjectValue(configKey, defaultValue, context);
  }
}
```

### 5. TypeScript Support

For TypeScript projects:

```typescript
import { OpenFeature, EvaluationContext } from '@openfeature/server-sdk';
import { DevCycleProvider } from '@devcycle/openfeature-nodejs-provider';
import { initializeDevCycle } from '@devcycle/nodejs-server-sdk';

let openFeatureClient: ReturnType<typeof OpenFeature.getClient>;

export async function initializeOpenFeature(): Promise<void> {
  const sdkKey = process.env.DEVCYCLE_SERVER_SDK_KEY;
  
  if (!sdkKey) {
    throw new Error('DEVCYCLE_SERVER_SDK_KEY is not set');
  }
  
  const devcycleClient = await initializeDevCycle(sdkKey).onClientInitialized();
  const provider = new DevCycleProvider(devcycleClient);
  
  await OpenFeature.setProviderAndWait(provider);
  openFeatureClient = OpenFeature.getClient();
}

export function getFeatureFlag(
  key: string,
  defaultValue: boolean,
  userId: string
): Promise<boolean> {
  const context: EvaluationContext = {
    targetingKey: userId
  };
  
  return openFeatureClient.getBooleanValue(key, defaultValue, context);
}
```

### 6. Environment Configuration

Create a `.env` file:

```bash
# .env
DEVCYCLE_SERVER_SDK_KEY=your_server_sdk_key_here
NODE_ENV=development
```

Load with dotenv:

```javascript
require('dotenv').config();
```

After installation, run your Node.js application and verify everything works with no errors.

## Troubleshooting

**Common Issues:**

1. **"Provider not initialized" error:**
   - Ensure `OpenFeature.setProviderAndWait()` is called
   - Check that DevCycle client initializes successfully
   - Verify your SDK key is correct (server SDK key)

2. **Feature flags returning default values only:**
   - Confirm the provider is initialized before use
   - Check that evaluation context has `targetingKey`
   - Verify feature flags are configured in DevCycle dashboard

3. **TypeScript compilation errors:**
   - Ensure all type packages are installed
   - Check TypeScript version compatibility
   - Verify import statements match module system

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
- [DevCycle Node.js OpenFeature Provider](https://docs.devcycle.com/sdk/server-side-sdks/node/node-openfeature/)
- [OpenFeature Node.js SDK](https://openfeature.dev/docs/reference/technologies/server/javascript)
- [DevCycle Dashboard](https://app.devcycle.com/)

## Support

If you encounter issues:

1. Check the official documentation
2. Review the troubleshooting section above
3. Contact DevCycle support through the dashboard
4. Check the OpenFeature community for help
