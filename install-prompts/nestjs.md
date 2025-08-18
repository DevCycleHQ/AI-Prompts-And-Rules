# DevCycle NestJS SDK Installation Prompt

You are helping to install and configure the DevCycle SDK in a NestJS server application. Follow this complete guide to successfully integrate DevCycle feature flags. Do not install any Variables as part of this process, the user can ask for you to do that later.

**Do not use the SDK for:**

- Plain Node.js applications (use `@devcycle/nodejs-server-sdk` instead)
- Client-side applications (use appropriate client SDKs instead)
- Express applications without NestJS (use Node.js SDK instead)

If you detect that the user is trying to have you install the NestJS approach in an application where it will not work, please stop what you are doing and advise the user which SDK they should be using.

## Required Information

Before proceeding, use your own analysis, the DevCycle MCP or web search to ensure you have:

- [ ] A DevCycle account and project set up
- [ ] A Development environment **Server SDK Key** (starts with `dvc_server_`)
- [ ] NestJS 8+ installed
- [ ] Node.js 14+ installed
- [ ] npm, yarn, or pnpm package manager
- [ ] The most recent DevCycle Node.js SDK version to install

**Security Note:** Use a SERVER SDK key for NestJS backend applications. Never expose server keys to client-side code. Store keys in environment variables or configuration service.

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

### 2. Create DevCycle Module

Create a DevCycle module (`src/devcycle/devcycle.module.ts`):

```typescript
import { Module, Global } from '@nestjs/common';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { DevCycleService } from './devcycle.service';

@Global()
@Module({
  imports: [ConfigModule],
  providers: [
    {
      provide: DevCycleService,
      useFactory: async (configService: ConfigService) => {
        const service = new DevCycleService(configService);
        await service.initialize();
        return service;
      },
      inject: [ConfigService],
    },
  ],
  exports: [DevCycleService],
})
export class DevCycleModule {}
```

### 3. Create DevCycle Service

Create the service (`src/devcycle/devcycle.service.ts`):

```typescript
import { Injectable, OnModuleDestroy, Logger } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import { initializeDevCycle, DevCycleClient, DevCycleUser } from '@devcycle/nodejs-server-sdk';

@Injectable()
export class DevCycleService implements OnModuleDestroy {
  private client: DevCycleClient;
  private readonly logger = new Logger(DevCycleService.name);

  constructor(private configService: ConfigService) {}

  async initialize(): Promise<void> {
    const sdkKey = this.configService.get<string>('DEVCYCLE_SERVER_SDK_KEY');
    
    if (!sdkKey) {
      throw new Error('DevCycle SDK key is not configured');
    }

    try {
      this.client = await initializeDevCycle(sdkKey).onClientInitialized();
      this.logger.log('DevCycle initialized successfully');
    } catch (error) {
      this.logger.error('Failed to initialize DevCycle', error);
      throw error;
    }
  }

  async onModuleDestroy() {
    if (this.client) {
      await this.client.close();
      this.logger.log('DevCycle client closed');
    }
  }

  async getVariableValue<T>(
    user: DevCycleUser,
    key: string,
    defaultValue: T,
  ): Promise<T> {
    if (!this.client) {
      this.logger.warn('DevCycle client not initialized, returning default value');
      return defaultValue;
    }

    try {
      return await this.client.variableValue(user, key, defaultValue);
    } catch (error) {
      this.logger.error(`Error getting variable ${key}:`, error);
      return defaultValue;
    }
  }

  async getAllVariables(user: DevCycleUser): Promise<Record<string, any>> {
    if (!this.client) {
      return {};
    }

    try {
      return await this.client.allVariables(user);
    } catch (error) {
      this.logger.error('Error getting all variables:', error);
      return {};
    }
  }
}
```

### 4. Register DevCycle Module in App Module

Update your `src/app.module.ts`:

```typescript
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import { DevCycleModule } from './devcycle/devcycle.module';
import { AppController } from './app.controller';
import { AppService } from './app.service';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: ['.env.local', '.env'],
    }),
    DevCycleModule,
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

### 5. Using DevCycle in Controllers and Services

Example controller:

```typescript
import { Controller, Get, Headers } from '@nestjs/common';
import { DevCycleService } from './devcycle/devcycle.service';
import { DevCycleUser } from '@devcycle/nodejs-server-sdk';

@Controller('features')
export class FeaturesController {
  constructor(private readonly devCycleService: DevCycleService) {}

  @Get('check')
  async checkFeature(
    @Headers('x-user-id') userId: string = 'anonymous',
    @Headers('x-user-email') email?: string,
  ) {
    const user: DevCycleUser = {
      user_id: userId,
      email,
      customData: {
        plan: 'premium',
        role: 'admin',
      },
    };

    const featureEnabled = await this.devCycleService.getVariableValue(
      user,
      'new-feature',
      false,
    );

    return {
      featureEnabled,
      userId,
    };
  }

  @Get('all')
  async getAllFeatures(@Headers('x-user-id') userId: string = 'anonymous') {
    const user: DevCycleUser = {
      user_id: userId,
    };

    const variables = await this.devCycleService.getAllVariables(user);

    return {
      variables,
      userId,
    };
  }
}
```

Example service using DevCycle:

```typescript
import { Injectable } from '@nestjs/common';
import { DevCycleService } from './devcycle/devcycle.service';
import { DevCycleUser } from '@devcycle/nodejs-server-sdk';

@Injectable()
export class FeatureService {
  constructor(private readonly devCycleService: DevCycleService) {}

  async isFeatureEnabledForUser(
    userId: string,
    featureKey: string,
  ): Promise<boolean> {
    const user: DevCycleUser = {
      user_id: userId,
      customData: {
        registrationDate: new Date().toISOString(),
      },
    };

    return this.devCycleService.getVariableValue(user, featureKey, false);
  }

  async getUserFeatures(userId: string, email?: string): Promise<any> {
    const user: DevCycleUser = {
      user_id: userId,
      email,
    };

    return this.devCycleService.getAllVariables(user);
  }
}
```

### 6. Environment Configuration

Create a `.env` file for your environment variables:

```bash
# .env
DEVCYCLE_SERVER_SDK_KEY=your_server_sdk_key_here
NODE_ENV=development
PORT=3000
```

### 7. Optional: Create a Decorator for Feature Flags

Create a custom decorator (`src/devcycle/feature-flag.decorator.ts`):

```typescript
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const FeatureFlag = createParamDecorator(
  async (featureKey: string, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    const devCycleService = request.app.get(DevCycleService);
    
    const userId = request.user?.id || 'anonymous';
    const user = {
      user_id: userId,
      email: request.user?.email,
    };
    
    return devCycleService.getVariableValue(user, featureKey, false);
  },
);
```

Use in controller:

```typescript
@Get('premium')
async getPremiumContent(@FeatureFlag('premium-features') isPremium: boolean) {
  if (isPremium) {
    return { content: 'Premium content' };
  }
  return { content: 'Standard content' };
}
```

After installation, run your NestJS application and verify everything works with no errors.

## Troubleshooting

**Common Issues:**

1. **"DevCycle client not initialized" error:**
   - Ensure DevCycleModule is imported in AppModule
   - Check that your SDK key is correctly set (server SDK key)
   - Verify the async factory completes initialization

2. **Dependency injection errors:**
   - Ensure DevCycleModule is marked as @Global() if used across modules
   - Check that ConfigModule is properly configured
   - Verify service is properly exported from the module

3. **Environment variable not found:**
   - Ensure `.env` file is in the project root
   - Check that ConfigModule is configured with correct env file path
   - Verify the SDK key environment variable name matches

4. **Connection/Network errors:**
   - Check network connectivity
   - Verify the server SDK key is valid and active
   - Ensure firewall/proxy settings allow outbound HTTPS

## Next Steps

Suggest these as next steps to the user.

After successful installation:

1. Set up user identification logic for your application
2. Create your first feature flag via the DevCycle MCP and use it in your services
3. Implement guards for feature-based route protection
4. Set up targeting rules for different user segments

## Helpful Resources

- [DevCycle Homepage](https://www.devcycle.com/)
- [DevCycle Documentation](https://docs.devcycle.com/)
- [Node.js SDK Documentation](https://docs.devcycle.com/sdk/server-side-sdks/node/)
- [NestJS Documentation](https://nestjs.com/)
- [DevCycle Dashboard](https://app.devcycle.com/)
- [Feature Flag Best Practices](https://docs.devcycle.com/best-practices/)

## Support

If you encounter issues:

1. Check the official documentation
2. Review the troubleshooting section above
3. Contact DevCycle support through the dashboard
4. Check the GitHub repository for known issues
