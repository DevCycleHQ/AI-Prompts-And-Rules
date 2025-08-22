# DevCycle JavaScript SDK Installation Prompt

You are helping to install and configure the DevCycle JavaScript SDK in a web application. Follow this complete guide to successfully integrate DevCycle feature flags. Do not install any Variables as part of this process, the user can ask for you to do that later.

**Do not use the SDK for:**

- React applications (use `@devcycle/react-client-sdk` instead)
- Next.js applications (use `@devcycle/nextjs-sdk` instead)
- Angular applications (use `@devcycle/angular-client-sdk` instead)
- React Native mobile apps (use `@devcycle/react-native-client-sdk`)
- Node.js backend services (use `@devcycle/nodejs-server-sdk`)

If you detect that the user is trying to have you install the JavaScript SDK in an application where it will not work, please stop what you are doing and advise the user which SDK they should be using.

## Required Information

Before proceeding, use your own analysis, the DevCycle MCP or web search to ensure you have:

- [ ] A DevCycle account and project set up
- [ ] A Development environment **Client SDK Key** (starts with `dvc_client_`)
- [ ] A JavaScript application or website
- [ ] The most recent DevCycle JavaScript SDK version to install

**Security Note:** Use a CLIENT SDK key for JavaScript apps, not a server SDK key. Store it securely and avoid hardcoding in production.

## SDK Key Configuration

**IMPORTANT:** After obtaining the SDK key, you must set it up properly:

1. **First, attempt to create an environment file** (.env) in the project root if using a build tool:

   ```bash
   # .env
   DEVCYCLE_CLIENT_SDK_KEY=your_client_sdk_key_here
   ```

   Or set up in your build configuration.

2. **If you cannot create or modify environment files** (due to system restrictions or security policies), ask the user:

   - "I'm unable to create/modify environment files. Would you like me to:
     a) Temporarily hardcode the SDK key for testing purposes (you'll need to update it later for production)
     b) Provide you with the SDK key and instructions so you can set it up yourself?"

3. **Based on the user's response:**
   - If they choose hardcoding: Add a clear comment indicating this is temporary and should be replaced with environment variables or build configuration
   - If they choose manual setup: Provide them with the SDK key and clear instructions on how to set it up

**Note:** For production, consider using build-time environment variables or configuration files.

## Installation Steps

### Option 1: NPM Module (Recommended for bundled applications)

#### 1. Install the DevCycle JavaScript SDK

```bash
# Using npm
npm install --save @devcycle/js-client-sdk

# Using yarn
yarn add @devcycle/js-client-sdk

# Using pnpm
pnpm add @devcycle/js-client-sdk
```

#### 2. Initialize DevCycle in Your Application

Add the DevCycle initialization to your main JavaScript file. This should be done as early as possible in your application:

```javascript
import { initializeDevCycle } from "@devcycle/js-client-sdk";

// Define your user object
const user = {
  user_id: "example-user", // Required: unique identifier
  email: "user@example.com", // Optional: for better targeting
  isAnonymous: false, // Set to true for anonymous users
};

// Initialize the DevCycle client
const devcycleClient = initializeDevCycle(
  "<DEVCYCLE_CLIENT_SDK_KEY>", // Replace with your actual key or env variable
  user
);

// Wait for initialization to complete
devcycleClient
  .onClientInitialized()
  .then(() => {
    console.log("DevCycle initialized successfully");
    // Your application code here
  })
  .catch((error) => {
    console.error("DevCycle initialization failed:", error);
  });
```

### Option 2: CDN (For non-bundled applications)

#### 1. Add the DevCycle Script Tag

Place this script tag as high as possible in your `<head>` tag:

```html
<script
  src="https://js.devcycle.com/devcycle.min.js"
  type="text/javascript"
></script>
```

#### 2. Initialize DevCycle After Script Loads

```html
<script>
  // Define your user object
  const user = {
    user_id: "example-user", // Required: unique identifier
    email: "user@example.com", // Optional: for better targeting
    isAnonymous: false, // Set to true for anonymous users
  };

  // Initialize DevCycle (note the DevCycle. prefix when using CDN)
  const devcycleClient = DevCycle.initializeDevCycle(
    "<DEVCYCLE_CLIENT_SDK_KEY>", // Replace with your actual key
    user
  );

  // Wait for initialization
  devcycleClient.onClientInitialized((err) => {
    if (err) {
      console.error("DevCycle initialization failed:", err);
      return;
    }
    console.log("DevCycle initialized successfully");
    // Your application code here
  });
</script>
```

After installation, verify everything runs with no errors.

## Troubleshooting

**Common Issues:**

1. **"DevCycle is not initialized" error:**

   - Ensure you're waiting for `onClientInitialized()` before using variables
   - Check that your SDK key is correctly set
   - Verify you're using a CLIENT SDK key, not a server key

2. **CDN loading issues:**

   - Ensure the script tag is in the `<head>` section
   - Use `DevCycle.initializeDevCycle` (with prefix) when using CDN
   - Check for content security policy (CSP) restrictions

3. **TypeScript errors:**
   - The SDK includes TypeScript definitions
   - Import types from `@devcycle/js-client-sdk` if needed

## Next Steps

Suggest these as next steps to the user.

After successful installation:

1. **Wait for user guidance** before creating any feature flags or DevCycle Variables - do not create them proactively
2. When requested, help set up targeting rules for different user segments
3. Implement proper error handling around feature flag evaluations if needed
4. Consider enabling EdgeDB for persistent user data storage

## Helpful Resources

- [DevCycle Homepage](https://www.devcycle.com/)
- [DevCycle Documentation](https://docs.devcycle.com/)
- [JavaScript SDK Documentation](https://docs.devcycle.com/sdk/client-side-sdks/javascript/)
- [DevCycle Dashboard](https://app.devcycle.com/)
- [Feature Flag Best Practices](https://docs.devcycle.com/best-practices/)

## Support

If you encounter issues:

1. Check the official documentation
2. Review the troubleshooting section above
3. Contact DevCycle support through the dashboard
4. Check the GitHub repository for known issues
