# DevCycle SDK Installation Instructions

Use this guide to install DevCycle SDK into a project.

Write the complete code for every step. Do not get lazy. Write everything that is needed.

Your goal is to completely finish the installation of the DevCycle SDK into the codebase.

## Helpful Links

If the user gets stuck, refer them to the following links:

- [DevCycle Homepage](https://www.devcycle.com/)
- [DevCycle Docs](https://docs.devcycle.com/)
- [DevCycle SDKs](https://docs.devcycle.com/sdk/)
- [DevCycle Dashboard](https://app.devcycle.com/)

## DevCycle Feature Flag Examples

### DevCycle Node.js SDK Example

Example of how to install and then use the DevCycle Node.js SDK (`@devcycle/nodejs-server-sdk` npm package) client to get the value of a feature flag:

```typescript
import { initializeDevCycle } from "@devcycle/nodejs-server-sdk";

const devcycleClient = await initializeDevCycle(
  process.env.DEVCYCLE_SERVER_SDK_KEY || "<DEVCYCLE_SERVER_SDK_KEY>"
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

- [] Identify from the application what version of the DevCycle SDK that should be used: DevCycle Javascript Web SDK (`@devcycle/js-client-sdk`), or DevCycle React SDK (`@devcycle/react-client-sdk`), or DevCycle NextJS SDK (`@devcycle/nextjs-sdk`), or DevCycle NodeJS SDK (`@devcycle/nodejs-server-sdk`).

- [] Add the DevCycle SDK to the package.json file.

- [] Install the DevCycle SDK into the codebase at the starting point of the application. The DevCycle SDK should be installed as early as possible in the application lifecycle. Setup the DevCycle SDK Client as a singleton that can be used throughout the application.

- [] Identify from the user's prompt if they want a new feature flag added as well, and what the variable key, default value, and type (String, Boolean, Number, or Object) are and where the flag should be inserted in the codebase.

- [] If required insert the new feature flag variable into the codebase at the location specified by the user. Refactor the code as needed, matching the style of the existing codebase. You should not need to create any new files.

- [] Instruct the user to install the updated package using the package manager that is being used by the application.
