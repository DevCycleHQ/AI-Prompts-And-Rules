# DevCycle React SDK Installation Prompt

You are helping to install and configure the DevCycle React SDK in a React web application. Follow this complete guide to successfully integrate DevCycle feature flags. Do not install any Variables as part of this process, the user can ask for you to do that later.

**Do not use the SDK for:**

- Next.js applications (use `@devcycle/nextjs-sdk` instead)
- React Native mobile apps (use `@devcycle/react-native-client-sdk`)
- Server-side React rendering without browser context
- Node.js backend services (use `@devcycle/nodejs-server-sdk`)

If you detect that the user is trying to have you install the React SDK in an application where it will not work, please stop what you are doing and advise the user which SDK they should be using.

## Required Information

Before proceeding, use your own analysis, the DevCycle MCP or web search to ensure you have:

- [ ] A DevCycle account and project set up
- [ ] A Development environment **Client SDK Key** (starts with `dvc_client_`)
- [ ] React 16.8+ installed in your project
- [ ] The most recent DevCycle React SDK version to install

**Security Note:** Use a CLIENT SDK key for React apps, not a server SDK key. Store it in environment variables like `REACT_APP_DEVCYCLE_CLIENT_SDK_KEY`.

## SDK Key Configuration

**IMPORTANT:** After obtaining the SDK key, you must set it up properly:

1. **First, attempt to create an environment file** (.env or .env.local) in the project root:

   ```bash
   # .env or .env.local
   REACT_APP_DEVCYCLE_CLIENT_SDK_KEY=your_client_sdk_key_here
   ```

2. **If you cannot create or modify environment files** (due to system restrictions or security policies), ask the user:

   - "I'm unable to create/modify environment files. Would you like me to:
     a) Temporarily hardcode the SDK key for testing purposes (you'll need to update it later for production)
     b) Provide you with the SDK key and instructions so you can set it up yourself?"

3. **Based on the user's response:**
   - If they choose hardcoding: Add a clear comment indicating this is temporary and should be replaced with environment variables
   - If they choose manual setup: Provide them with the SDK key and clear instructions on how to set up the environment variable

**Note:** Always prefer environment variables over hardcoding for security reasons.

## Installation Steps

### 1. Install the DevCycle React SDK

```bash
# Using npm
npm install --save @devcycle/react-client-sdk

# Using yarn
yarn add @devcycle/react-client-sdk

# Using pnpm
pnpm add @devcycle/react-client-sdk
```

### 2. Wrap Your App with DevCycle Provider

Add the DevCycle provider to your root App component. This should be done as early as possible in your application initialization:

```javascript
import React from "react";
import { withDevCycleProvider } from "@devcycle/react-client-sdk";

function App() {
  return (
    <div>
      <h1>My React App</h1>
      {/* Your app components here */}
    </div>
  );
}

export default withDevCycleProvider({
  sdkKey: process.env.REACT_APP_DEVCYCLE_CLIENT_SDK_KEY,
  user: {
    user_id: "example-user", // Required: unique identifier
    email: "user@example.com", // Optional: for better targeting
    isAnonymous: false, // Recommended to be explicit on whether or not to use an Anonymous ID
  },
})(App);
```

After installation, verify everything runs with no errors.

## Troubleshooting

**Common Issues:**

1. **"DevCycle is not initialized" error:**

   - Ensure the provider wraps your entire app
   - Check that your SDK key is correctly set in environment variables
   - Verify you're using a CLIENT SDK key, not a server key

2. **TypeScript errors:**
   - The SDK includes TypeScript definitions

## Next Steps

Suggest these as next steps to the user.

After successful installation:

1. **Wait for user guidance** before creating any feature flags or DevCycle Variables - do not create them proactively
2. When requested, help set up targeting rules for different user segments
3. Implement proper error boundaries around feature-flagged components if needed

## Helpful Resources

- [DevCycle Homepage](https://www.devcycle.com/)
- [DevCycle Documentation](https://docs.devcycle.com/)
- [React SDK Documentation](https://docs.devcycle.com/sdk/client-side-sdks/react/)
- [DevCycle Dashboard](https://app.devcycle.com/)
- [Feature Flag Best Practices](https://docs.devcycle.com/best-practices/)

## Support

If you encounter issues:

1. Check the official documentation
2. Review the troubleshooting section above
3. Contact DevCycle support through the dashboard
4. Check the GitHub repository for known issues
