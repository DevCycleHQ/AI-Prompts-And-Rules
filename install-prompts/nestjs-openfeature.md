# DevCycle NestJS OpenFeature Provider Installation Prompt

You are helping to install and configure the DevCycle OpenFeature Provider for NestJS server applications. Follow this complete guide to successfully integrate DevCycle feature flags using the OpenFeature standard. Do not install any Variables as part of this process, the user can ask for you to do that later.

**Do not use this for:**

- Plain Node.js applications (use `nodejs-openfeature.md` instead)
- Client-side applications (use appropriate client SDKs instead)
- Express applications without NestJS (use Node.js OpenFeature instead)

## Required Information

Before proceeding, use your own analysis, the DevCycle MCP or web search to ensure you have:

- [ ] A DevCycle account and project set up
- [ ] A Development environment **Server SDK Key** (starts with `dvc_server_`)
- [ ] NestJS 8+ installed
- [ ] Node.js 14+ installed
- [ ] npm, yarn, or pnpm package manager
- [ ] The most recent OpenFeature and DevCycle provider versions

**Security Note:** Use a SERVER SDK key for NestJS backend applications. Never expose server keys to client-side code. Store keys in environment variables or configuration service.

## Installation Steps

### 1. Install OpenFeature SDK and DevCycle Provider

```bash
# Using npm
npm install --save @openfeature/server-sdk @openfeature/nestjs-sdk @devcycle/openfeature-nodejs-provider @devcycle/nodejs-server-sdk

# Using yarn
yarn add @openfeature/server-sdk @openfeature/nestjs-sdk @devcycle/openfeature-nodejs-provider @devcycle/nodejs-server-sdk

# Using pnpm
pnpm add @openfeature/server-sdk @openfeature/nestjs-sdk @devcycle/openfeature-nodejs-provider @devcycle/nodejs-server-sdk
```

### 2. Create OpenFeature Module

Create an OpenFeature module (`src/openfeature/openfeature.module.ts`):

```typescript
import { Module, Global } from '@nestjs/common';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { OpenFeatureModule as NestOpenFeatureModule } from '@openfeature/nestjs-sdk';
import { OpenFeature } from '@openfeature/server-sdk';
import { DevCycleProvider } from '@devcycle/openfeature-nodejs-provider';
import { initializeDevCycle } from '@devcycle/nodejs-server-sdk';

@Global()
@Module({
  imports: [
    ConfigModule,
    NestOpenFeatureModule.forRootAsync({
      imports: [ConfigModule],
      useFactory: async (configService: ConfigService) => {
        const sdkKey = configService.get<string>('DEVCYCLE_SERVER_SDK_KEY');
        
        if (!sdkKey) {
          throw new Error('DEVCYCLE_SERVER_SDK_KEY is not configured');
        }
        
        // Initialize DevCycle client
        const devcycleClient = await initializeDevCycle(sdkKey).onClientInitialized();
        
        // Create DevCycle provider
        const provider = new DevCycleProvider(devcycleClient);
        
        // Set the provider
        await OpenFeature.setProviderAndWait(provider);
        
        return {
          defaultProvider: provider,
          providers: {
            devcycle: provider,
          },
        };
      },
      inject: [ConfigService],
    }),
  ],
  exports: [NestOpenFeatureModule],
})
export class OpenFeatureModule {}
```

### 3. Create OpenFeature Service

Create a service wrapper (`src/openfeature/openfeature.service.ts`):

```typescript
import { Injectable, Logger } from '@nestjs/common';
import { OpenFeatureService as NestOpenFeatureService } from '@openfeature/nestjs-sdk';
import { EvaluationContext } from '@openfeature/server-sdk';

@Injectable()
export class FeatureFlagService {
  private readonly logger = new Logger(FeatureFlagService.name);
  
  constructor(
    private readonly openFeatureService: NestOpenFeatureService,
  ) {}
  
  async getBooleanValue(
    key: string,
    defaultValue: boolean,
    userId: string,
    attributes?: Record<string, any>,
  ): Promise<boolean> {
    const context: EvaluationContext = {
      targetingKey: userId,
      ...attributes,
    };
    
    try {
      return await this.openFeatureService.getBooleanValue(key, defaultValue, context);
    } catch (error) {
      this.logger.error(`Error evaluating boolean flag ${key}:`, error);
      return defaultValue;
    }
  }
  
  async getStringValue(
    key: string,
    defaultValue: string,
    userId: string,
    attributes?: Record<string, any>,
  ): Promise<string> {
    const context: EvaluationContext = {
      targetingKey: userId,
      ...attributes,
    };
    
    try {
      return await this.openFeatureService.getStringValue(key, defaultValue, context);
    } catch (error) {
      this.logger.error(`Error evaluating string flag ${key}:`, error);
      return defaultValue;
    }
  }
  
  async getNumberValue(
    key: string,
    defaultValue: number,
    userId: string,
    attributes?: Record<string, any>,
  ): Promise<number> {
    const context: EvaluationContext = {
      targetingKey: userId,
      ...attributes,
    };
    
    try {
      return await this.openFeatureService.getNumberValue(key, defaultValue, context);
    } catch (error) {
      this.logger.error(`Error evaluating number flag ${key}:`, error);
      return defaultValue;
    }
  }
  
  async getObjectValue<T extends object>(
    key: string,
    defaultValue: T,
    userId: string,
    attributes?: Record<string, any>,
  ): Promise<T> {
    const context: EvaluationContext = {
      targetingKey: userId,
      ...attributes,
    };
    
    try {
      return await this.openFeatureService.getObjectValue(key, defaultValue, context) as T;
    } catch (error) {
      this.logger.error(`Error evaluating object flag ${key}:`, error);
      return defaultValue;
    }
  }
}
```

### 4. Register OpenFeature Module in App Module

Update your `src/app.module.ts`:

```typescript
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import { OpenFeatureModule } from './openfeature/openfeature.module';
import { FeatureFlagService } from './openfeature/openfeature.service';
import { AppController } from './app.controller';
import { AppService } from './app.service';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: ['.env.local', '.env'],
    }),
    OpenFeatureModule,
  ],
  controllers: [AppController],
  providers: [AppService, FeatureFlagService],
})
export class AppModule {}
```

### 5. Using OpenFeature in Controllers

Example controller:

```typescript
import { Controller, Get, Headers, Query } from '@nestjs/common';
import { FeatureFlagService } from './openfeature/openfeature.service';

@Controller('features')
export class FeaturesController {
  constructor(private readonly featureFlagService: FeatureFlagService) {}
  
  @Get('check')
  async checkFeature(
    @Query('feature') featureKey: string,
    @Headers('x-user-id') userId: string = 'anonymous',
    @Headers('x-user-email') email?: string,
  ) {
    const attributes = {
      email,
      plan: 'premium',
      role: 'admin',
    };
    
    const [boolValue, stringValue, numberValue, objectValue] = await Promise.all([
      this.featureFlagService.getBooleanValue(featureKey, false, userId, attributes),
      this.featureFlagService.getStringValue(`${featureKey}_text`, 'default', userId, attributes),
      this.featureFlagService.getNumberValue(`${featureKey}_limit`, 100, userId, attributes),
      this.featureFlagService.getObjectValue(
        `${featureKey}_config`,
        { theme: 'light', fontSize: 14 },
        userId,
        attributes,
      ),
    ]);
    
    return {
      featureKey,
      userId,
      values: {
        boolean: boolValue,
        string: stringValue,
        number: numberValue,
        object: objectValue,
      },
    };
  }
}
```

### 6. Using OpenFeature in Services

Example service:

```typescript
import { Injectable } from '@nestjs/common';
import { FeatureFlagService } from './openfeature/openfeature.service';

@Injectable()
export class UserService {
  constructor(private readonly featureFlagService: FeatureFlagService) {}
  
  async getUserDashboard(userId: string, email?: string) {
    const attributes = { email, authenticated: true };
    
    const showNewDashboard = await this.featureFlagService.getBooleanValue(
      'new-dashboard',
      false,
      userId,
      attributes,
    );
    
    if (showNewDashboard) {
      return this.getNewDashboard(userId);
    }
    
    return this.getLegacyDashboard(userId);
  }
  
  async getApiLimits(userId: string) {
    const rateLimit = await this.featureFlagService.getNumberValue(
      'api-rate-limit',
      100,
      userId,
    );
    
    const config = await this.featureFlagService.getObjectValue(
      'api-config',
      { timeout: 30000, retries: 3 },
      userId,
    );
    
    return {
      rateLimit,
      config,
    };
  }
  
  private async getNewDashboard(userId: string) {
    // New dashboard implementation
    return { type: 'new', userId };
  }
  
  private async getLegacyDashboard(userId: string) {
    // Legacy dashboard implementation
    return { type: 'legacy', userId };
  }
}
```

### 7. Optional: Create Guard for Feature Flags

Create a guard (`src/openfeature/feature-flag.guard.ts`):

```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { FeatureFlagService } from './openfeature.service';

export const RequireFeature = (featureKey: string) =>
  Reflect.metadata('featureKey', featureKey);

@Injectable()
export class FeatureFlagGuard implements CanActivate {
  constructor(
    private reflector: Reflector,
    private featureFlagService: FeatureFlagService,
  ) {}
  
  async canActivate(context: ExecutionContext): Promise<boolean> {
    const featureKey = this.reflector.get<string>('featureKey', context.getHandler());
    
    if (!featureKey) {
      return true;
    }
    
    const request = context.switchToHttp().getRequest();
    const userId = request.user?.id || 'anonymous';
    
    return this.featureFlagService.getBooleanValue(featureKey, false, userId);
  }
}
```

Use in controller:

```typescript
@Get('premium')
@UseGuards(FeatureFlagGuard)
@RequireFeature('premium-features')
async getPremiumContent() {
  return { content: 'Premium content' };
}
```

### 8. Environment Configuration

Create a `.env` file:

```bash
# .env
DEVCYCLE_SERVER_SDK_KEY=your_server_sdk_key_here
NODE_ENV=development
PORT=3000
```

After installation, run your NestJS application and verify everything works with no errors.

## Troubleshooting

**Common Issues:**

1. **"Provider not initialized" error:**
   - Ensure OpenFeatureModule is imported in AppModule
   - Check that DevCycle client initializes in the module factory
   - Verify your SDK key is correct (server SDK key)

2. **Dependency injection errors:**
   - Ensure all services are properly provided in their modules
   - Check that OpenFeatureModule is marked as @Global() if needed
   - Verify service imports and exports

3. **Feature flags returning default values only:**
   - Confirm the provider is initialized before use
   - Check that evaluation context has targetingKey
   - Verify feature flags are configured in DevCycle dashboard

4. **Connection/Network errors:**
   - Check network connectivity
   - Verify the server SDK key is valid and active
   - Ensure firewall settings allow HTTPS

## Next Steps

Suggest these as next steps to the user.

After successful installation:

1. Create feature flags via the DevCycle MCP
2. Implement guards for feature-based route protection
3. Set up evaluation context strategies
4. Configure targeting rules in DevCycle dashboard

## Helpful Resources

- [DevCycle Homepage](https://www.devcycle.com/)
- [OpenFeature Documentation](https://openfeature.dev/)
- [OpenFeature NestJS SDK](https://github.com/open-feature/js-sdk/tree/main/packages/nestjs-sdk)
- [DevCycle Node.js SDK](https://docs.devcycle.com/sdk/server-side-sdks/node/)
- [NestJS Documentation](https://nestjs.com/)
- [DevCycle Dashboard](https://app.devcycle.com/)

## Support

If you encounter issues:

1. Check the official documentation
2. Review the troubleshooting section above
3. Contact DevCycle support through the dashboard
4. Check the OpenFeature community for help
