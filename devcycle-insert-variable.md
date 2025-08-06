# DevCycle Variable Creation Instructions

Use this guide to insert a new DevCycle Feature Flag Variable into this project.

Write the complete code for every step. Do not get lazy. Write everything that is needed, restructure the code as needed. Write new variables in the same style as the existing variables in the codebase. Do not add code comments.

Your goal is to completely finish the feature flag creation.

## Helpful Links

If the user gets stuck, refer them to the following links:

- [DevCycle Homepage](https://www.devcycle.com/)
- [DevCycle Docs](https://docs.devcycle.com/)
- [DevCycle SDKs](https://docs.devcycle.com/sdk/)
- [DevCycle Dashboard](https://app.devcycle.com/)

## DevCycle Feature Flag Examples

### [DevCycle JavaScript SDK Example](https://docs.devcycle.com/sdk/client-side-sdks/javascript/)

Example of how a DevCycle Javascript SDK client (`@devcycle/js-client-sdk` npm package) is initialized and how to get the value of a feature flag:

```typescript
const user = { user_id: "my_user" };
// The devcycleClient is initialized with the DevCycle SDK key somewhere in the codebase with a user object containing the user_id.
const devcycleClient = initializeDevCycle("<DEVCYCLE_CLIENT_SDK_KEY>", user);

// Then throughout the codebase, you can use the devcycleClient to get the feature flag values using the variableValue method.
// The first parameter is the variable key.
// The second parameter is the default value can be of type String, Boolean, Number, or Object.
// The resulting value is the same type as the default value.
const value = devcycleClient.variableValue(
  "<YOUR_VARIABLE_KEY>",
  "default value"
);
```

See the [Typescript Schema](https://raw.githubusercontent.com/DevCycleHQ/js-sdks/main/sdk/js/src/types.ts), for reference to the `DVCVariable`, `DevCycleUser` typescript schemas.

### [DevCycle React SDK Example](https://docs.devcycle.com/sdk/client-side-sdks/react/)

Example of how to use the DevCycle React SDK client (`@devcycle/react-client-sdk` npm package) to get the value of a feature flag:

```typescript
import { useVariableValue } from "@devcycle/react-client-sdk";

const DevCycleFeaturePage = () => {
  // Then throughout the codebase, you can use the useVariableValue to get the feature flag values.
  // The first parameter is the variable key.
  // The second parameter is the default value can be of type String, Boolean, Number, or Object.
  // The resulting value is the same type as the default value.
  const featureVariable = useVariableValue("<YOUR_VARIABLE_KEY>", false);

  return (
    <div>
      {featureVariable ? <div>Variable on!</div> : <div>Variable off</div>}
    </div>
  );
};
```

See the [Typescript Schema](https://raw.githubusercontent.com/DevCycleHQ/js-sdks/main/sdk/js/src/types.ts), for reference to the `DVCVariable`, `DevCycleUser` typescript schemas.

### [DevCycle Next.js SDK Example](https://docs.devcycle.com/sdk/client-side-sdks/nextjs/)

Example of how to use the DevCycle Next.js SDK client (`@devcycle/nextjs-sdk` npm package) to get the value of a feature flag in a server component:

```typescript
import { getVariableValue } from "./devcycle";
import * as React from "react";

export const MyServerComponent = async function () {
  // Then throughout the codebase, you can use the getVariableValue to get the feature flag values.
  // The first parameter is the variable key.
  // The second parameter is the default value can be of type String, Boolean, Number, or Object.
  // The resulting value is the same type as the default value.
  const myVariable = await getVariableValue("myVariable", false);
  return myVariable ? <NewComponent /> : <OldComponent />;
};

// Note: it is recommended to use a module alias to access your DevCycle shared file from your server components. https://nextjs.org/docs/app/building-your-application/configuring/absolute-imports-and-module-aliases
```

Client Component Example:

```javascript
"use client";
import { useVariableValue } from "@devcycle/nextjs-sdk";
import * as React from "react";

export const MyClientComponent = function () {
  // Then throughout the codebase, you can use the useVariableValue to get the feature flag values.
  // The first parameter is the variable key.
  // The second parameter is the default value can be of type String, Boolean, Number, or Object.
  // The resulting value is the same type as the default value.
  const myVariable = useVariableValue("myVariable", false);
  return myVariable ? <NewComponent /> : <OldComponent />;
};
```

### DevCycle Node.js SDK Example

Example of how to use the DevCycle Node.js SDK client (`@devcycle/nodejs-server-sdk` npm package) to get the value of a feature flag:

```typescript
import { initializeDevCycle } from "@devcycle/nodejs-server-sdk";

// The devcycleClient is initialized with the DevCycle SDK key somewhere in the codebase.
const devcycleClient = await initializeDevCycle(
  process.env.DEVCYCLE_SERVER_SDK_KEY
).onClientInitialized();

// Then throughout the codebase, you can use the devcycleClient to get the feature flag values using the variableValue method.
// The first value is the user object.
// The second parameter is the variable key.
// The third parameter is the default value can be of type String, Boolean, Number, or Object.
// The resulting value is the same type as the default value.
const user = {
  user_id: "user1@devcycle.com",
  name: "user 1 name",
  customData: {
    customKey: "customValue",
  },
};
const variable = devcycleClient.variableValue(user, "test-feature", false);
```

See the [Typescript Schema](https://raw.githubusercontent.com/DevCycleHQ/js-sdks/main/sdk/js/src/types.ts), for reference to the `DVCVariable`, `DevCycleUser` typescript schemas.

## Setup Steps

- [] Identify the DevCycle SDK from the root `package.json` file that is being used: DevCycle Javascript SDK (`@devcycle/js-client-sdk`), or DevCycle React SDK (`@devcycle/react-client-sdk`), or DevCycle NextJS SDK (`@devcycle/nextjs-sdk`), or DevCycle NodeJS SDK (`@devcycle/nodejs-server-sdk`). Open the `package.json` file and look for `@devcycle` packages.

- [] Identify from the user's prompt what the new feature flag variable key, default value, and type (String, Boolean, Number, or Object) are and where the flag should be inserted in the codebase.

- [] Search for other feature flag variables in the codebase to match the style of the existing variables in the codebase. Use the same import statements, and the same variable key name format as existing variables in the codebase.

- [] Insert the new feature flag variable into the codebase at the location specified by the user. Refactor the code as needed, matching the style of the existing codebase. You should not need to create any new files.

- [] Update any tests that were affected by the new feature flag variable insertion.
