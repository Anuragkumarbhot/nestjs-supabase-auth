# WARPPOINT: Space War

WARPPOINT: Space War is a large-scale multiplayer sci-fi action game built with **Godot Engine 4**.  
Players engage in space combat, planetary warfare, and capital ship boarding while contributing to a persistent galaxy-scale conflict.

---

## 🚀 Overview

WARPPOINT blends:
- Space dogfights
- Ground combat on hostile planets
- Capital ship warfare and boarding missions
- Persistent faction-driven galaxy wars

Every mission affects the larger universe through a server-authoritative simulation designed for fairness, scalability, and long-term LiveOps support.

---

## 🛠 Engine & Tools

- **Engine:** Godot Engine 4.x
- **Scripting:** GDScript (primary), optional C# (Mono)
- **Platforms:** PC (Windows/Linux) — scalable to others
- **Version Control:** Git + Git LFS
- **Build Style:** Headless server + client builds (planned)

---

## 🎮 Core Features

### Multiplayer
- Server-authoritative architecture
- Squad-based gameplay
- PvE + PvP encounters
- Session recovery & reconnect support

### Combat
- Spacecraft dogfighting
- Planet-side FPS/TPS combat
- Capital ship combat & boarding
- Server-validated damage and hit logic

### AI
- Server-controlled AI
- Planet-aware enemies and creatures
- Boarding defender AI with escalation logic

### Galaxy War
- Persistent galaxy state
- Factions and territory influence
- Seasonal progression
- Outcome-driven universe changes

### LiveOps (Planned)
- Telemetry & metrics
- Balance tuning hooks
- Seasonal content and events
- AI adaptation across seasons

---

## 📁 Project Structure (High-Level)# nestjs-supabase-auth

## Installation

### Install peer dependencies

Using npm:
```
npm install passport passport-jwt @nestjs/passport
npm install --save-dev @types/passport-jwt
```

Using yarn:
```
yarn add passport passport-jwt @nestjs/passport
yarn add -D @types/passport-jwt
```

### Install strategy

Using npm:
```
npm install nestjs-supabase-auth
```

Using yarn:
```
yarn add nestjs-supabase-auth
```

## Example

### Extends the strategy to create your own strategy

In this example, I'm passing supabase related options through dotenv and [env-cmd](https://github.com/toddbluhm/env-cmd) package. 

```ts
import { Injectable } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { ExtractJwt } from 'passport-jwt';
import { SupabaseAuthStrategy } from 'nestjs-supabase-auth';

@Injectable()
export class SupabaseStrategy extends PassportStrategy(
  SupabaseAuthStrategy,
  'supabase',
) {
  public constructor() {
    super({
      supabaseUrl: process.env.SUPABASE_URL,
      supabaseKey: process.env.SUPABASE_KEY,
      supabaseOptions: {},
      supabaseJwtSecret: process.env.SUPABASE_JWT_SECRET,
      extractor: ExtractJwt.fromAuthHeaderAsBearerToken(),
    });
  }

  async validate(payload: any): Promise<any> {
    return super.validate(payload);
  }

  authenticate(req) { 
    super.authenticate(req);
  }
}
```

### Add the strategy to your auth module

```ts
import { Module } from '@nestjs/common';
import { AuthService } from './auth.service';
import { AuthResolver } from './auth.resolver';
import { SupabaseStrategy } from './supabase.strategy';
import { PassportModule } from '@nestjs/passport';
import supabase from '../../supabase';

@Module({
  imports: [PassportModule],
  providers: [
    AuthService,
    AuthResolver,
    SupabaseStrategy,
  ],
  exports: [AuthService, SupabaseStrategy],
})
export class AuthModule {}
```
### Protect your routes

#### Example for Graphql

gql-auth-guard.ts
```ts
import { ExecutionContext, Injectable } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';
import { GqlExecutionContext } from '@nestjs/graphql';

@Injectable()
export class GqlAuthGuard extends AuthGuard('supabase') {
  getRequest(context: ExecutionContext) {
    const ctx = GqlExecutionContext.create(context);
    return ctx.getContext().req;
  }
}
```

auth.resolver - You can use the guard in any resolver. 

```ts
import { UseGuards } from '@nestjs/common';
import { Args, Context, Query, Mutation, Resolver } from '@nestjs/graphql';
import { GqlAuthGuard } from 'src/common/guards/auth.guard';
import { CurrentUser } from '../../common/decorators/current-user';
import { IUser } from '../user/models/user.interface';
import { AuthService } from './auth.service';
import { SignupInput } from './dto/signup.input';
import { AuthResult } from './models/auth-result';
import { AuthUser as SupabaseAuthUser } from '@supabase/supabase-js';
import { LoginInput } from './dto/login.input';

@Resolver()
export class AuthResolver {
  constructor(private readonly authService: AuthService) {}

  @Query(() => IUser, { name: 'viewer' })
  @UseGuards(GqlAuthGuard)
  async me(@CurrentUser() user: SupabaseAuthUser) {
    return user;
  }
  ...
}

```

### CurrentUser decorator

```ts
import { createParamDecorator, ExecutionContext } from "@nestjs/common";
import { GqlExecutionContext } from "@nestjs/graphql";

export const CurrentUser = createParamDecorator(
  (_data: unknown, context: ExecutionContext) => {
    const ctx = GqlExecutionContext.create(context);
    return ctx.getContext().req.user;
  },
);

```
