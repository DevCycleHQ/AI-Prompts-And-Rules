# DevCycle React OpenFeature Provider Installation Prompt

You are helping to install and configure the DevCycle OpenFeature Provider for React applications. Follow this complete guide to successfully integrate DevCycle feature flags using the OpenFeature standard. Do not install any Variables as part of this process, the user can ask for you to do that later.

**Do not use this for:**

- Next.js applications (use Next.js SDK instead)
- React Native apps (use React Native SDK instead)
- Non-React JavaScript applications (use `javascript-openfeature.md` instead)
- Server-side React rendering without browser context

## Required Information

Before proceeding, use your own analysis, the DevCycle MCP or web search to ensure you have:

- [ ] A DevCycle account and project set up
- [ ] A Development environment **Client SDK Key** (starts with `dvc_client_`)
- [ ] React 16.8+ installed (hooks support required)
- [ ] The most recent OpenFeature and DevCycle provider versions

**Security Note:** Use a CLIENT SDK key for React apps, not a server SDK key. Store it in environment variables like `REACT_APP_DEVCYCLE_CLIENT_SDK_KEY`.

## Installation Steps

### 1. Install OpenFeature SDK and DevCycle Provider

```bash
# Using npm
npm install --save @openfeature/react-sdk @devcycle/openfeature-web-provider

# Using yarn
yarn add @openfeature/react-sdk @devcycle/openfeature-web-provider

# Using pnpm
pnpm add @openfeature/react-sdk @devcycle/openfeature-web-provider
```

### 2. Set Up OpenFeature with DevCycle Provider

In your main App component or entry point:

```javascript
import React from 'react';
import { OpenFeatureProvider, OpenFeature } from '@openfeature/react-sdk';
import { DevCycleProvider } from '@devcycle/openfeature-web-provider';

// Initialize OpenFeature with DevCycle
async function setupOpenFeature() {
  // Set initial user context
  await OpenFeature.setContext({
    targetingKey: 'default-user', // Required: unique user identifier
    email: 'user@example.com', // Optional
    anonymous: false
  });
  
  // Create and set the DevCycle provider
  const devCycleProvider = new DevCycleProvider(
    process.env.REACT_APP_DEVCYCLE_CLIENT_SDK_KEY // Use environment variable
  );
  
  await OpenFeature.setProviderAndWait(devCycleProvider);
}

// Call setup before rendering
setupOpenFeature();

function App() {
  return (
    <OpenFeatureProvider>
      <div className="App">
        <h1>My React App with OpenFeature</h1>
        {/* Your app components */}
      </div>
    </OpenFeatureProvider>
  );
}

export default App;
```

### 3. Alternative: Initialize in useEffect

For more control over initialization timing:

```javascript
import React, { useEffect, useState } from 'react';
import { OpenFeatureProvider, OpenFeature } from '@openfeature/react-sdk';
import { DevCycleProvider } from '@devcycle/openfeature-web-provider';

function App() {
  const [isReady, setIsReady] = useState(false);

  useEffect(() => {
    const initializeFeatureFlags = async () => {
      await OpenFeature.setContext({
        targetingKey: 'default-user',
        email: 'user@example.com'
      });
      
      const provider = new DevCycleProvider(
        process.env.REACT_APP_DEVCYCLE_CLIENT_SDK_KEY
      );
      
      await OpenFeature.setProviderAndWait(provider);
      setIsReady(true);
    };

    initializeFeatureFlags().catch(console.error);
  }, []);

  if (!isReady) {
    return <div>Loading...</div>;
  }

  return (
    <OpenFeatureProvider>
      <div className="App">
        {/* Your app components */}
      </div>
    </OpenFeatureProvider>
  );
}

export default App;
```

### 4. Using Feature Flags with OpenFeature Hooks

```javascript
import React from 'react';
import { useBooleanFlagValue, useStringFlagValue, useNumberFlagValue, useObjectFlagValue } from '@openfeature/react-sdk';

function FeatureComponent() {
  // Boolean flag
  const showNewFeature = useBooleanFlagValue('new-feature', false);
  
  // String flag
  const buttonText = useStringFlagValue('button-text', 'Click Here');
  
  // Number flag
  const maxItems = useNumberFlagValue('max-items', 10);
  
  // Object flag
  const uiConfig = useObjectFlagValue('ui-config', {
    theme: 'light',
    fontSize: 14
  });

  return (
    <div style={{ fontSize: uiConfig.fontSize }}>
      {showNewFeature ? (
        <div>
          <h2>New Feature!</h2>
          <button>{buttonText}</button>
          <p>Showing max {maxItems} items</p>
        </div>
      ) : (
        <div>
          <h2>Standard Feature</h2>
        </div>
      )}
    </div>
  );
}
```

### 5. Updating User Context

```javascript
import { useContext } from 'react';
import { OpenFeature } from '@openfeature/react-sdk';

function UserProfile() {
  const handleLogin = async (userId, userEmail) => {
    // Update context when user logs in
    await OpenFeature.setContext({
      targetingKey: userId,
      email: userEmail,
      authenticated: true,
      plan: 'premium',
      role: 'admin'
    });
    
    // Feature flags will automatically re-evaluate
  };

  const handleLogout = async () => {
    // Set anonymous context on logout
    await OpenFeature.setContext({
      targetingKey: `anon-${Date.now()}`,
      anonymous: true
    });
  };

  return (
    <div>
      <button onClick={() => handleLogin('user123', 'user@example.com')}>
        Login
      </button>
      <button onClick={handleLogout}>Logout</button>
    </div>
  );
}
```

### 6. Environment Configuration

Create environment files for different environments:

```bash
# .env.development
REACT_APP_DEVCYCLE_CLIENT_SDK_KEY=your_dev_client_sdk_key

# .env.production
REACT_APP_DEVCYCLE_CLIENT_SDK_KEY=your_prod_client_sdk_key
```

After installation, run your React application and verify everything works with no errors.

## Troubleshooting

**Common Issues:**

1. **"Provider not ready" error:**
   - Ensure OpenFeatureProvider wraps your app
   - Wait for provider initialization before rendering
   - Check that your SDK key is correctly set (client SDK key)

2. **Hooks returning default values only:**
   - Confirm the provider has finished initializing
   - Check that your feature flags are configured in DevCycle dashboard
   - Verify the context has a valid `targetingKey`

3. **React hooks errors:**
   - Ensure hooks are called at the top level of components
   - Check that React version is 16.8+
   - Verify OpenFeatureProvider is in the component tree

4. **TypeScript errors:**
   - Both packages include TypeScript definitions
   - May need to install @types/react if not present

## Next Steps

Suggest these as next steps to the user.

After successful installation:

1. Create your first feature flag via the DevCycle MCP
2. Implement feature flag-based routing
3. Set up error boundaries for feature components
4. Configure targeting rules in DevCycle dashboard

## Helpful Resources

- [DevCycle Homepage](https://www.devcycle.com/)
- [OpenFeature Documentation](https://openfeature.dev/)
- [DevCycle OpenFeature Provider](https://docs.devcycle.com/sdk/client-side-sdks/react/react-openfeature/)
- [OpenFeature React SDK](https://openfeature.dev/docs/reference/technologies/client/web/react)
- [DevCycle Dashboard](https://app.devcycle.com/)

## Support

If you encounter issues:

1. Check the official documentation
2. Review the troubleshooting section above
3. Contact DevCycle support through the dashboard
4. Check the OpenFeature community for help
