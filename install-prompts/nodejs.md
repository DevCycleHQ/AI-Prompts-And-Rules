# DevCycle Node.js SDK Installation Prompt

You are helping to install and configure the DevCycle Node.js SDK in a Node.js server application. Follow this complete guide to successfully integrate DevCycle feature flags. Do not install any Variables as part of this process, the user can ask for you to do that later.

**Do not use the SDK for:**

- Client-side JavaScript applications (use `@devcycle/js-client-sdk` instead)
- React applications (use `@devcycle/react-client-sdk` instead)
- Next.js applications (use `@devcycle/nextjs-sdk` instead)
- NestJS applications (use NestJS-specific SDK instead)

If you detect that the user is trying to have you install the Node.js SDK in an application where it will not work, please stop what you are doing and advise the user which SDK they should be using.

## Required Information

Before proceeding, use your own analysis, the DevCycle MCP or web search to ensure you have:

- [ ] A DevCycle account and project set up
- [ ] A Development environment **Server SDK Key** (starts with `dvc_server_`)
- [ ] Node.js 12+ installed
- [ ] npm, yarn, or pnpm package manager
- [ ] The most recent DevCycle Node.js SDK version to install

**Security Note:** Use a SERVER SDK key for Node.js backend applications. Never expose server keys to client-side code. Store keys in environment variables.

## Installation Steps

### 1. Install the DevCycle Node.js SDK

```bash
# Using npm
npm install --save @devcycle/nodejs-server-sdk

# Using yarn
yarn add @devcycle/nodejs-server-sdk

# Using pnpm
pnpm add @devcycle/nodejs-server-sdk
```

### 2. Initialize DevCycle in Your Application

Create a DevCycle initialization file (e.g., `devcycle.js` or `devcycle.ts`):

#### JavaScript Example

```javascript
const { initializeDevCycle } = require('@devcycle/nodejs-server-sdk')

let devcycleClient

async function initDevCycle() {
  if (!devcycleClient) {
    // Initialize the DevCycle client
    devcycleClient = await initializeDevCycle(
      process.env.DEVCYCLE_SERVER_SDK_KEY || '<DEVCYCLE_SERVER_SDK_KEY>'
    ).onClientInitialized()
    
    console.log('DevCycle initialized successfully')
  }
  return devcycleClient
}

// Export the initialization function and getter
module.exports = {
  initDevCycle,
  getDevCycleClient: () => devcycleClient
}
```

#### TypeScript Example

```typescript
import { initializeDevCycle, DevCycleClient } from '@devcycle/nodejs-server-sdk'

let devcycleClient: DevCycleClient | null = null

export async function initDevCycle(): Promise<DevCycleClient> {
  if (!devcycleClient) {
    // Initialize the DevCycle client
    devcycleClient = await initializeDevCycle(
      process.env.DEVCYCLE_SERVER_SDK_KEY || '<DEVCYCLE_SERVER_SDK_KEY>'
    ).onClientInitialized()
    
    console.log('DevCycle initialized successfully')
  }
  return devcycleClient
}

export function getDevCycleClient(): DevCycleClient | null {
  return devcycleClient
}
```

### 3. Initialize DevCycle on Application Startup

In your main application file (e.g., `app.js`, `index.js`, or `server.js`):

```javascript
const express = require('express')
const { initDevCycle } = require('./devcycle')

const app = express()
const PORT = process.env.PORT || 3000

async function startServer() {
  try {
    // Initialize DevCycle before starting the server
    await initDevCycle()
    
    // Your application routes and middleware
    app.get('/', (req, res) => {
      res.send('Server is running with DevCycle!')
    })
    
    app.listen(PORT, () => {
      console.log(`Server is running on port ${PORT}`)
    })
  } catch (error) {
    console.error('Failed to initialize server:', error)
    process.exit(1)
  }
}

startServer()
```

### 4. Using DevCycle in Routes/Controllers

Example of using DevCycle in an Express route:

```javascript
const { getDevCycleClient } = require('./devcycle')

app.get('/api/feature', async (req, res) => {
  const devcycleClient = getDevCycleClient()
  
  // Define user for this request
  const user = {
    user_id: req.user?.id || 'anonymous',
    email: req.user?.email,
    name: req.user?.name,
    customData: {
      plan: req.user?.plan,
      role: req.user?.role
    }
  }
  
  // Check feature flag value
  const featureEnabled = await devcycleClient.variableValue(
    user,
    'feature-key',
    false  // default value
  )
  
  res.json({ featureEnabled })
})
```

### 5. Environment Configuration

Create a `.env` file for your environment variables:

```bash
# .env
DEVCYCLE_SERVER_SDK_KEY=your_server_sdk_key_here
NODE_ENV=development
```

Install dotenv if not already installed:

```bash
npm install --save dotenv
```

Load environment variables at the top of your main file:

```javascript
require('dotenv').config()
```

After installation, run your Node.js application and verify everything works with no errors.

## Troubleshooting

**Common Issues:**

1. **"DevCycle client is not initialized" error:**
   - Ensure `initDevCycle()` is called before using the client
   - Check that your SDK key is correctly set (server SDK key)
   - Verify the initialization promise resolves successfully

2. **Environment variable not found:**
   - Ensure `.env` file is in the project root
   - Check that dotenv is installed and configured
   - Verify the SDK key environment variable name matches

3. **TypeScript compilation errors:**
   - Ensure @types/node is installed
   - Check that TypeScript version is compatible
   - Verify import statements match your module system

4. **Connection/Network errors:**
   - Check network connectivity
   - Verify the server SDK key is valid and active
   - Ensure firewall/proxy settings allow outbound HTTPS

## Next Steps

Suggest these as next steps to the user.

After successful installation:

1. Set up user identification logic for your application
2. Create your first feature flag via the DevCycle MCP and use it in your routes
3. Implement proper error handling for feature flag evaluations
4. Set up targeting rules for different user segments

## Helpful Resources

- [DevCycle Homepage](https://www.devcycle.com/)
- [DevCycle Documentation](https://docs.devcycle.com/)
- [Node.js SDK Documentation](https://docs.devcycle.com/sdk/server-side-sdks/node/)
- [DevCycle Dashboard](https://app.devcycle.com/)
- [Node.js SDK GitHub Repository](https://github.com/DevCycleHQ/js-sdks)
- [Feature Flag Best Practices](https://docs.devcycle.com/best-practices/)

## Support

If you encounter issues:

1. Check the official documentation
2. Review the troubleshooting section above
3. Contact DevCycle support through the dashboard
4. Check the GitHub repository for known issues
