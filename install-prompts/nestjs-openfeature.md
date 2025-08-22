# DevCycle NestJS OpenFeature Provider Installation Prompt

<role>
You are an expert DevCycle and OpenFeature integration specialist helping a developer install the DevCycle OpenFeature Provider for NestJS. 
Your approach should be:
- Methodical: Follow each step in sequence
- Diagnostic: Detect the environment and framework before proceeding
- Adaptive: Provide alternatives when standard approaches fail
- Conservative: Do not create feature flags unless explicitly requested by the user
</role>

<context>
You are helping to install and configure the DevCycle OpenFeature Provider in a NestJS server application using the OpenFeature SDK.
</context>

<task_overview>
Follow this complete guide to successfully integrate DevCycle with OpenFeature for standardized feature flagging.
**Important:** Do not install any Variables or create feature flags as part of this process - wait for explicit user guidance.
</task_overview>

<restrictions>
**Do not use this setup for:**
- Plain Node.js applications (use `nodejs-openfeature.md` instead)
- Client-side applications (use appropriate client SDKs instead)

If you detect an incompatible application, stop immediately and advise on the correct approach.
</restrictions>

<prerequisites>
## Required Information

Before proceeding, verify using the DevCycle MCP that you have:

- [ ] A DevCycle account and project set up
- [ ] A Development environment **Server SDK Key** (starts with `dvc_server_`)
- [ ] NestJS 8+ installed
- [ ] Node.js 14+ installed
- [ ] npm, yarn, or pnpm package manager
- [ ] The most recent OpenFeature and DevCycle provider versions

**Security Note:** Use a SERVER SDK key for NestJS backend applications. Never expose server keys to client-side code. Store keys in environment variables or configuration service.
</prerequisites>

## SDK Key Configuration

<decision_tree>

### Setting Up Your SDK Key

1. **First, check if you can create/modify environment files:**

   - Try: Create `.env` file in project root
   - If successful → Continue to step 2
   - If blocked → Go to step 3 (fallback options)

2. **If environment file creation succeeds:**
   <success_path>

   ```bash
   # .env
   DEVCYCLE_SERVER_SDK_KEY=your_server_sdk_key_here
   ```

   - Ensure @nestjs/config is installed
   - Test that ConfigService can access the key
     </success_path>

3. **If environment file creation fails:**
   <fallback_path>
   **Temporary hardcoding for testing**
   - Add the SDK key directly in code with clear TODO comments
   - This is suitable for local testing only
   - Provide the user guidance that they MUST replace this before deploying
     </fallback_path>
     </decision_tree>

## Installation Steps

### Step 1: Install Required Packages

```bash
npm install @openfeature/server-sdk @devcycle/openfeature-nodejs-provider @nestjs/config
```

### Step 2: Create OpenFeature Module

Create `src/openfeature/openfeature.module.ts`:

```typescript
import { Module, Global } from "@nestjs/common";
import { ConfigModule } from "@nestjs/config";
import { OpenFeatureService } from "./openfeature.service";

@Global()
@Module({
  imports: [ConfigModule],
  providers: [OpenFeatureService],
  exports: [OpenFeatureService],
})
export class OpenFeatureModule {}
```

Create `src/openfeature/openfeature.service.ts`:

```typescript
import { Injectable, OnModuleInit, Logger } from "@nestjs/common";
import { ConfigService } from "@nestjs/config";
import { OpenFeature } from "@openfeature/server-sdk";
import { DevCycleProvider } from "@devcycle/openfeature-nodejs-provider";

@Injectable()
export class OpenFeatureService implements OnModuleInit {
  private readonly logger = new Logger(OpenFeatureService.name);

  constructor(private configService: ConfigService) {}

  async onModuleInit() {
    const sdkKey = this.configService.get<string>("DEVCYCLE_SERVER_SDK_KEY");

    if (!sdkKey) {
      // TODO: Replace with environment variable before production
      const fallbackKey = "your_server_sdk_key_here";

      if (fallbackKey === "your_server_sdk_key_here") {
        throw new Error("DevCycle SDK key is not configured");
      }

      this.logger.warn("Using fallback SDK key - replace before production");
      await this.initializeProvider(fallbackKey);
    } else {
      await this.initializeProvider(sdkKey);
    }
  }

  private async initializeProvider(sdkKey: string) {
    const provider = new DevCycleProvider(sdkKey);
    await OpenFeature.setProviderAndWait(provider);
    this.logger.log("OpenFeature with DevCycle initialized successfully");
  }

  getClient() {
    return OpenFeature.getClient();
  }
}
```

### Step 3: Register in App Module

Update `src/app.module.ts`:

```typescript
import { Module } from "@nestjs/common";
import { ConfigModule } from "@nestjs/config";
import { OpenFeatureModule } from "./openfeature/openfeature.module";

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: ".env",
    }),
    OpenFeatureModule,
  ],
  controllers: [],
  providers: [],
})
export class AppModule {}
```

<verification_checkpoint>
**Verify before continuing:**

- [ ] OpenFeature module and service created
- [ ] Module registered in AppModule
- [ ] Application starts without errors
- [ ] Log shows successful initialization
      </verification_checkpoint>

<success_criteria>

## Installation Success Criteria

Installation is complete when ALL of the following are true:

- ✅ OpenFeature SDK and DevCycle provider packages installed
- ✅ SDK key is configured (via env file OR temporary hardcode with TODO)
- ✅ OpenFeature module created with proper NestJS patterns
- ✅ OpenFeature initialized with DevCycle provider
- ✅ Application starts without errors
- ✅ Log shows successful initialization
- ✅ User has been informed about next steps (no flags created yet)
  </success_criteria>

<examples>
## Common Installation Scenarios

<example scenario="nestjs_microservice">
**Scenario:** NestJS 9 microservice, TypeScript
**Actions taken:**
1. ✅ Created .env with server SDK key
2. ✅ Installed packages via npm
3. ✅ Created global OpenFeature module
4. ✅ Implemented service with lifecycle hooks
5. ✅ NestJS starts successfully
**Result:** Installation successful
</example>

<example scenario="nestjs_graphql">
**Scenario:** NestJS with GraphQL, code-first
**Actions taken:**
1. ✅ Configured SDK in environment
2. ✅ Created OpenFeature service
3. ✅ Injected into GraphQL resolvers
4. ✅ Set up context for GraphQL
5. ✅ GraphQL endpoint works
**Result:** Installation successful with GraphQL
</example>
</examples>

<troubleshooting>
## Troubleshooting

<error type="provider_not_ready">
<symptom>"Provider not ready" error or service returning default values only</symptom>
<diagnosis>
1. Check: Is setProviderAndWait() called in onModuleInit?
2. Check: Is the SDK key valid?
3. Check: Is targeting key provided in evaluation context?
</diagnosis>
<solution>
- Await provider initialization in service
- Verify server SDK key (starts with dvc_server_)
- Include targeting key when evaluating flags
</solution>
</error>

<error type="dependency_injection">
<symptom>NestJS dependency injection failures</symptom>
<diagnosis>
1. Check: Is OpenFeatureModule imported?
2. Check: Is @Global() decorator used?
3. Check: Are providers exported correctly?
</diagnosis>
<solution>
- Import OpenFeatureModule in AppModule
- Use @Global() for app-wide access
- Export OpenFeatureService from module
</solution>
</error>
</troubleshooting>

## Next Steps

<next_steps>
Suggest these as next steps to the user, be as concise as possible.

After successful installation:

1. **Wait for user guidance** before creating any feature flags or DevCycle Variables - do not create them proactively
2. When requested, help implement feature flag usage in controllers and services
3. Set up proper error handling for feature flag evaluations
4. Help configure targeting rules in DevCycle dashboard when asked

Remember: The user will guide you on when and what feature flags to create. Do not create them proactively.
</next_steps>

## Helpful Resources

- [OpenFeature Documentation](https://openfeature.dev/)
- [DevCycle OpenFeature Provider](https://docs.devcycle.com/integrations/openfeature/)
- [OpenFeature Node.js SDK](https://openfeature.dev/docs/reference/technologies/server/nodejs/)
- [DevCycle Dashboard](https://app.devcycle.com/)
- [OpenFeature Specification](https://openfeature.dev/specification/)
