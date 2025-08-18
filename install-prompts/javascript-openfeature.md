# DevCycle JavaScript OpenFeature Provider Installation Prompt

You are helping to install and configure the DevCycle OpenFeature Provider for JavaScript web applications. Follow this complete guide to successfully integrate DevCycle feature flags using the OpenFeature standard. Do not install any Variables as part of this process, the user can ask for you to do that later.

**Do not use this for:**

- React applications (use `javascript-openfeature.md` or `react-openfeature.md` instead)
- Node.js server applications (use `nodejs-openfeature.md` instead)
- Applications already using the DevCycle JavaScript SDK directly

## Required Information

Before proceeding, use your own analysis, the DevCycle MCP or web search to ensure you have:

- [ ] A DevCycle account and project set up
- [ ] A Development environment **Client SDK Key** (starts with `dvc_client_`)
- [ ] JavaScript application or website
- [ ] The most recent OpenFeature and DevCycle provider versions

**Security Note:** Use a CLIENT SDK key for JavaScript apps, not a server SDK key. Store it securely and avoid hardcoding in production.

## Installation Steps

### 1. Install OpenFeature SDK and DevCycle Provider

```bash
# Using npm
npm install --save @openfeature/web-sdk @devcycle/openfeature-web-provider

# Using yarn
yarn add @openfeature/web-sdk @devcycle/openfeature-web-provider

# Using pnpm
pnpm add @openfeature/web-sdk @devcycle/openfeature-web-provider
```

### 2. Initialize OpenFeature with DevCycle Provider

```javascript
import { OpenFeature } from '@openfeature/web-sdk';
import { DevCycleProvider } from '@devcycle/openfeature-web-provider';

async function initializeFeatureFlags() {
  // Create the DevCycle provider
  const devCycleProvider = new DevCycleProvider('<DEVCYCLE_CLIENT_SDK_KEY>'); // Replace with your client SDK key
  
  // Set user context
  await OpenFeature.setContext({
    targetingKey: 'default-user', // Required: unique user identifier
    email: 'user@example.com', // Optional: for better targeting
    name: 'User Name' // Optional: additional user data
  });
  
  // Set DevCycle as the provider and wait for initialization
  await OpenFeature.setProviderAndWait(devCycleProvider);
  
  console.log('OpenFeature with DevCycle initialized successfully');
}

// Initialize on page load
initializeFeatureFlags().catch(console.error);
```

### 3. Using Feature Flags in Your Application

```javascript
import { OpenFeature } from '@openfeature/web-sdk';

// Get the OpenFeature client
const client = OpenFeature.getClient();

// Check boolean feature flag
async function checkFeature() {
  const isEnabled = await client.getBooleanValue('new-feature', false);
  
  if (isEnabled) {
    console.log('New feature is enabled!');
    // Show new feature
  } else {
    console.log('Using default experience');
    // Show default experience
  }
}

// Get string value
async function getStringConfig() {
  const value = await client.getStringValue('button-text', 'Click Here');
  document.getElementById('myButton').textContent = value;
}

// Get number value
async function getNumberConfig() {
  const limit = await client.getNumberValue('api-rate-limit', 100);
  console.log(`Rate limit: ${limit}`);
}

// Get object/JSON value
async function getObjectConfig() {
  const config = await client.getObjectValue('ui-config', {
    theme: 'light',
    fontSize: 14
  });
  applyUIConfig(config);
}
```

### 4. Handling User Context Changes

```javascript
// Update user context when user logs in
async function onUserLogin(userId, userEmail) {
  await OpenFeature.setContext({
    targetingKey: userId,
    email: userEmail,
    authenticated: true,
    // Add any custom attributes for targeting
    plan: 'premium',
    role: 'admin'
  });
  
  // Feature flag values will automatically update
}

// Handle anonymous users
async function setAnonymousUser() {
  await OpenFeature.setContext({
    targetingKey: `anon-${Date.now()}`, // Generate unique ID for anonymous user
    anonymous: true
  });
}
```

### 5. Environment Configuration

For better security, use environment variables or configuration files:

```javascript
// config.js
export const config = {
  devcycleClientKey: process.env.DEVCYCLE_CLIENT_SDK_KEY || 'fallback_key',
  environment: process.env.NODE_ENV || 'development'
};

// main.js
import { config } from './config.js';

const devCycleProvider = new DevCycleProvider(config.devcycleClientKey);
```

After installation, run your application and verify everything works with no errors.

## Troubleshooting

**Common Issues:**

1. **"Provider not ready" error:**
   - Ensure you await `OpenFeature.setProviderAndWait()`
   - Check that your SDK key is correctly set (client SDK key)
   - Verify the context has a valid `targetingKey`

2. **Feature flags returning default values only:**
   - Confirm the provider has finished initializing
   - Check that your feature flags are properly configured in DevCycle dashboard
   - Verify the user context is set correctly

3. **TypeScript errors:**
   - Both packages include TypeScript definitions
   - Ensure your TypeScript version is compatible

4. **Build/bundling issues:**
   - Check that your bundler supports ES modules
   - May need to configure module resolution for some bundlers

## Next Steps

Suggest these as next steps to the user.

After successful installation:

1. Create your first feature flag via the DevCycle MCP
2. Set up event tracking for feature flag evaluations
3. Implement proper error handling
4. Configure targeting rules in DevCycle dashboard

## Helpful Resources

- [DevCycle Homepage](https://www.devcycle.com/)
- [OpenFeature Documentation](https://openfeature.dev/)
- [DevCycle OpenFeature Provider](https://docs.devcycle.com/sdk/client-side-sdks/javascript/javascript-openfeature/)
- [OpenFeature Web SDK](https://openfeature.dev/docs/reference/technologies/client/web/)
- [DevCycle Dashboard](https://app.devcycle.com/)

## Support

If you encounter issues:

1. Check the official documentation
2. Review the troubleshooting section above
3. Contact DevCycle support through the dashboard
4. Check the OpenFeature community for help
