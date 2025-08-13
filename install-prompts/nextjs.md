# DevCycle Next.js SDK Installation Prompt

You are helping to install and configure the DevCycle Next.js SDK in a Next.js application. Follow this complete guide to successfully integrate DevCycle feature flags. Do not install any Variables as part of this process, the user can ask for you to do that later.

**Do not use the SDK for:**
- React applications without Next.js (use `@devcycle/react-client-sdk` instead)
- Plain JavaScript/TypeScript web apps (use `@devcycle/js-client-sdk` instead)
- React Native mobile apps (use `@devcycle/react-native-client-sdk`)
- Node.js backend services without Next.js (use `@devcycle/nodejs-server-sdk`)

If you detect that the user is trying to have you install the Next.js SDK in an application where it will not work, please stop what you are doing and advise the user which SDK they should be using.

## Required Information

Before proceeding, use your own analysis, the DevCycle MCP or web search to ensure you have:
- [ ] A DevCycle account and project set up
- [ ] A Development environment **Server SDK Key** (starts with `dvc_server_`)
- [ ] A Development environment **Client SDK Key** (starts with `dvc_client_`)
- [ ] Next.js 14.1+ and React 18.2+ installed
- [ ] The most recent DevCycle Next.js SDK version to install

**Security Note:** You need BOTH a server SDK key and a client SDK key. The server key is used privately on the server, while the client key is public and sent to browsers. Never expose the server key to the client.

## Installation Steps

### 1. Install the DevCycle Next.js SDK

```bash
# Using npm
npm install --save @devcycle/nextjs-sdk

# Using yarn
yarn add @devcycle/nextjs-sdk

# Using pnpm
pnpm add @devcycle/nextjs-sdk
```

### 2. TypeScript Configuration (if using TypeScript)

If using TypeScript with App Router, ensure your `tsconfig.json` includes:

```json
{
  "compilerOptions": {
    "moduleResolution": "bundler"
  }
}
```

Required minimum versions for TypeScript:
- TypeScript: 5.1.3+
- @types/react: 18.2.8+

## Setup for App Router (Recommended)

### 3. Create DevCycle Context File

Create a new file `app/devcycle.ts` (or similar location) to set up DevCycle:

```typescript
import { setupDevCycle } from '@devcycle/nextjs-sdk/server'

const getUserIdentity = async () => {
  // Return a user object for DevCycle
  // You can customize this later to fetch real user data
  return {
    user_id: 'default-user',
    isAnonymous: false
  }
}

export const { getVariableValue, getClientContext } = setupDevCycle({
  serverSDKKey: process.env.DEVCYCLE_SERVER_SDK_KEY ?? '',
  clientSDKKey: process.env.NEXT_PUBLIC_DEVCYCLE_CLIENT_SDK_KEY ?? '',
  userGetter: getUserIdentity,
  options: {}
})
```

### 4. Add DevCycle Provider to Root Layout

In your root layout file (typically `app/layout.tsx`):

```typescript
import { DevCycleClientsideProvider } from '@devcycle/nextjs-sdk'
import { getClientContext } from './devcycle'

export default async function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body>
        <DevCycleClientsideProvider context={getClientContext()}>
          {children}
        </DevCycleClientsideProvider>
      </body>
    </html>
  )
}
```

### 5. Set Environment Variables

Create or update your `.env.local` file:

```bash
DEVCYCLE_SERVER_SDK_KEY=your_server_sdk_key_here
NEXT_PUBLIC_DEVCYCLE_CLIENT_SDK_KEY=your_client_sdk_key_here
```

**Important:** The client SDK key MUST be prefixed with `NEXT_PUBLIC_` to be available in browser code.

## Setup for Pages Router (Legacy)

### 3. Wrap Your App Component

In your `pages/_app.tsx` file:

```typescript
import React from 'react'
import type { AppProps } from 'next/app'
import { appWithDevCycle } from '@devcycle/nextjs-sdk/pages'

function MyApp({ Component, pageProps }: AppProps) {
  return <Component {...pageProps} />
}

export default appWithDevCycle(MyApp)
```

### 4. Add Server-Side Support to Pages

In each page where you want to use DevCycle, add:

```typescript
import { GetServerSideProps } from 'next'
import { getServerSideDevCycle } from '@devcycle/nextjs-sdk/pages'

export const getServerSideProps: GetServerSideProps = async (context) => {
  const user = {
    user_id: 'default-user',
    isAnonymous: false
  }
  
  const devcycleData = await getServerSideDevCycle({
    serverSDKKey: process.env.DEVCYCLE_SERVER_SDK_KEY ?? '',
    clientSDKKey: process.env.NEXT_PUBLIC_DEVCYCLE_CLIENT_SDK_KEY ?? '',
    user,
    context
  })

  return {
    props: {
      ...devcycleData
    }
  }
}
```

### 5. Set Environment Variables

Create or update your `.env.local` file:

```bash
DEVCYCLE_SERVER_SDK_KEY=your_server_sdk_key_here
NEXT_PUBLIC_DEVCYCLE_CLIENT_SDK_KEY=your_client_sdk_key_here
```

After installation, restart your development server and verify everything runs with no errors.

## Troubleshooting

**Common Issues:**

1. **"DevCycle is not initialized" error:**
   - Ensure the DevCycleClientsideProvider wraps your app (App Router)
   - Check that both SDK keys are correctly set in environment variables
   - Verify you're using the correct SDK key types (server vs client)

2. **TypeScript errors with async components:**
   - Update TypeScript to 5.1.3+ and @types/react to 18.2.8+
   - Ensure `moduleResolution: "bundler"` is set in tsconfig.json

3. **Realtime updates not working (Next.js < 14.1):**
   - Upgrade to Next.js 14.1+ or disable realtime updates in options

4. **Environment variables not found:**
   - Ensure client key uses `NEXT_PUBLIC_` prefix
   - Restart the development server after adding environment variables
   - Check that `.env.local` is in your project root

## Next Steps

Suggest these as next steps to the user.

After successful installation:
1. Customize the `getUserIdentity` function to fetch real user data
2. Create your first feature flag via the DevCycle MCP and use it in your components
3. Learn how to use variables in both server and client components
4. Set up targeting rules for different user segments

## Helpful Resources

- [DevCycle Homepage](https://www.devcycle.com/)
- [DevCycle Documentation](https://docs.devcycle.com/)
- [Next.js SDK Documentation](https://docs.devcycle.com/sdk/client-side-sdks/nextjs/)
- [DevCycle Dashboard](https://app.devcycle.com/)
- [Feature Flag Best Practices](https://docs.devcycle.com/best-practices/)

## Support

If you encounter issues:
1. Check the official documentation
2. Review the troubleshooting section above
3. Contact DevCycle support through the dashboard
4. Check the GitHub repository for known issues
