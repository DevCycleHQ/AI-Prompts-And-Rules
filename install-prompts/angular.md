# DevCycle Angular SDK Installation Prompt

You are helping to install and configure the DevCycle Angular SDK in an Angular application. Follow this complete guide to successfully integrate DevCycle feature flags. Do not install any Variables as part of this process, the user can ask for you to do that later.

**Do not use the SDK for:**

- React applications (use `@devcycle/react-client-sdk` instead)
- Next.js applications (use `@devcycle/nextjs-sdk` instead)
- Plain JavaScript/TypeScript web apps (use `@devcycle/js-client-sdk` instead)
- Vue.js applications (use `@devcycle/js-client-sdk` instead)
- Node.js backend services (use `@devcycle/nodejs-server-sdk`)

If you detect that the user is trying to have you install the Angular SDK in an application where it will not work, please stop what you are doing and advise the user which SDK they should be using.

## Required Information

Before proceeding, use your own analysis, the DevCycle MCP or web search to ensure you have:

- [ ] A DevCycle account and project set up
- [ ] A Development environment **Client SDK Key** (starts with `dvc_client_`)
- [ ] Angular application (Angular 12+)
- [ ] The most recent DevCycle Angular SDK version to install

**Security Note:** Use a CLIENT SDK key for Angular apps, not a server SDK key. Store it in environment variables where possible.

## SDK Key Configuration

**IMPORTANT:** After obtaining the SDK key, you must set it up properly:

1. **First, attempt to set up the SDK key in Angular environment files** (src/environments/environment.ts):

   ```typescript
   export const environment = {
     production: false,
     devcycleClientSdkKey: "your_client_sdk_key_here",
   };
   ```

2. **If you cannot create or modify environment files** (due to system restrictions or security policies), ask the user:

   - "I'm unable to create/modify environment files. Would you like me to:
     a) Temporarily hardcode the SDK key for testing purposes (you'll need to update it later for production)
     b) Provide you with the SDK key and instructions so you can set it up yourself?"

3. **Based on the user's response:**
   - If they choose hardcoding: Add a clear comment indicating this is temporary and should be replaced with environment configuration
   - If they choose manual setup: Provide them with the SDK key and clear instructions on how to set up the environment configuration

**Note:** Always prefer environment configuration over hardcoding for security reasons.

## Installation Steps

### 1. Install the DevCycle Angular SDK and OpenFeature SDK

```bash
# Using npm
npm install --save @devcycle/openfeature-angular-provider @openfeature/angular-sdk

# Using yarn
yarn add @devcycle/openfeature-angular-provider @openfeature/angular-sdk

# Using pnpm
pnpm add @devcycle/openfeature-angular-provider @openfeature/angular-sdk
```

### 2. Configure DevCycle in Your Root Module

In your root module file (typically `app.module.ts`), import and configure the DevCycle provider:

```typescript
import { NgModule } from "@angular/core";
import { BrowserModule } from "@angular/platform-browser";
import { OpenFeatureModule, OpenFeature } from "@openfeature/angular-sdk";
import DevCycleAngularProvider from "@devcycle/openfeature-angular-provider";
import { AppComponent } from "./app.component";

// Initialize the DevCycle provider
const devCycleProvider = new DevCycleAngularProvider(
  "<DEVCYCLE_CLIENT_SDK_KEY>" // Replace with your actual key or use environment.DEVCYCLE_CLIENT_SDK_KEY
);

// Set the user context - targetingKey or user_id is required
OpenFeature.setContext({
  targetingKey: "default-user", // Required: unique user identifier
  email: "user@example.com", // Optional: for better targeting
  name: "User Name", // Optional: for better targeting
});

@NgModule({
  declarations: [AppComponent],
  imports: [
    BrowserModule,
    OpenFeatureModule.forRoot({
      provider: devCycleProvider,
    }),
  ],
  providers: [],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

### 3. Set Up Environment Variables (Recommended)

Create or update your environment files to store the SDK key:

In `src/environments/environment.ts`:

```typescript
export const environment = {
  production: false,
  DEVCYCLE_CLIENT_SDK_KEY: "your_development_client_sdk_key_here",
};
```

In `src/environments/environment.prod.ts`:

```typescript
export const environment = {
  production: true,
  DEVCYCLE_CLIENT_SDK_KEY: "your_production_client_sdk_key_here",
};
```

Then update your module to use the environment variable:

```typescript
import { environment } from "../environments/environment";

const devCycleProvider = new DevCycleAngularProvider(
  environment.DEVCYCLE_CLIENT_SDK_KEY
);
```

After installation, run your Angular application and verify everything runs with no errors.

## Troubleshooting

**Common Issues:**

1. **"DevCycle is not initialized" error:**

   - Ensure OpenFeatureModule.forRoot() is in your root module imports
   - Check that your SDK key is correctly set
   - Verify you're using a CLIENT SDK key, not a server key

2. **"Missing targetingKey" error:**

   - Ensure you call `OpenFeature.setContext()` with either `targetingKey` or `user_id`
   - The context must be set before the provider is initialized

3. **TypeScript errors:**

   - The SDK includes TypeScript definitions
   - Ensure you're importing from the correct packages

4. **Module import errors:**
   - Make sure both packages are installed: `@devcycle/openfeature-angular-provider` and `@openfeature/angular-sdk`
   - Import OpenFeatureModule.forRoot() only in your root module

## Next Steps

Suggest these as next steps to the user.

After successful installation:

1. Update the user context with real user data when users log in
2. **Wait for user guidance** before creating any feature flags or DevCycle Variables - do not create them proactively
3. When requested, help implement feature flags in Angular components
4. Help set up targeting rules for different user segments when asked

## Helpful Resources

- [DevCycle Homepage](https://www.devcycle.com/)
- [DevCycle Documentation](https://docs.devcycle.com/)
- [Angular SDK Documentation](https://docs.devcycle.com/sdk/client-side-sdks/angular/)
- [OpenFeature Angular SDK](https://openfeature.dev/docs/reference/technologies/client/web/angular)
- [DevCycle Dashboard](https://app.devcycle.com/)
- [Feature Flag Best Practices](https://docs.devcycle.com/best-practices/)

## Support

If you encounter issues:

1. Check the official documentation
2. Review the troubleshooting section above
3. Contact DevCycle support through the dashboard
4. Check the GitHub repository for known issues
