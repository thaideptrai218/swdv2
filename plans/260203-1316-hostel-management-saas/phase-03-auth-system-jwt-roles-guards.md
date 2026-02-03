---
title: "Phase 03: Auth System"
status: pending
priority: P1
effort: 3d
---

# Phase 03: Auth System - JWT, Roles, Guards

## Context Links

- [Tech Architecture Report](../reports/researcher-260203-1204-tech-architecture.md)
- [Phase 02: Database Design](./phase-02-database-design-prisma-schema-rls-migrations.md)

## Overview

Implement custom JWT authentication with Passport.js. Support dual auth flows: staff (tenant-scoped) and guests (public). Role-based access control with NestJS guards.

## Key Insights

- Custom JWT over third-party auth for multi-tenant flexibility
- Access tokens (15min) + Refresh tokens (7 days)
- Staff users scoped to tenant; guests are global
- Roles: OWNER > MANAGER > STAFF hierarchy

## Requirements

### Functional
- Staff login with email/password
- Guest registration/login (optional OAuth later)
- JWT access + refresh token flow
- Role-based route protection
- Tenant context injection from JWT
- Password reset flow
- Session invalidation

### Non-Functional
- Token validation <10ms
- Secure password storage (bcrypt)
- Refresh token rotation
- Rate limiting on auth endpoints

## Architecture

### Token Structure

```typescript
// Access Token Claims
{
  sub: string;       // user ID
  email: string;
  role: 'OWNER' | 'MANAGER' | 'STAFF' | 'GUEST';
  tenantId?: string; // null for guests
  type: 'access';
  iat: number;
  exp: number;       // 15 minutes
}

// Refresh Token Claims
{
  sub: string;
  type: 'refresh';
  jti: string;       // unique token ID for rotation
  iat: number;
  exp: number;       // 7 days
}
```

### Auth Flow

```
1. Login → Validate credentials → Issue access + refresh tokens
2. API Request → Validate access token → Extract user/tenant context
3. Token Expired → Use refresh token → Issue new token pair
4. Logout → Invalidate refresh token in Redis
```

## Related Code Files

### Create
- `apps/api/src/modules/auth/auth.module.ts`
- `apps/api/src/modules/auth/auth.service.ts`
- `apps/api/src/modules/auth/auth.controller.ts`
- `apps/api/src/modules/auth/strategies/jwt.strategy.ts`
- `apps/api/src/modules/auth/strategies/jwt-refresh.strategy.ts`
- `apps/api/src/modules/auth/guards/jwt-auth.guard.ts`
- `apps/api/src/modules/auth/guards/roles.guard.ts`
- `apps/api/src/modules/auth/decorators/roles.decorator.ts`
- `apps/api/src/modules/auth/decorators/current-user.decorator.ts`
- `apps/api/src/modules/auth/dto/login.dto.ts`
- `apps/api/src/modules/auth/dto/register.dto.ts`
- `packages/shared/src/schemas/auth.schema.ts`

## Implementation Steps

### 1. Install Dependencies (Day 1)

```bash
cd apps/api
pnpm add @nestjs/passport @nestjs/jwt passport passport-jwt bcrypt
pnpm add -D @types/passport-jwt @types/bcrypt
```

### 2. JWT Strategy (Day 1)

```typescript
// apps/api/src/modules/auth/strategies/jwt.strategy.ts
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { ExtractJwt, Strategy } from 'passport-jwt';
import { ConfigService } from '@nestjs/config';

export interface JwtPayload {
  sub: string;
  email: string;
  role: string;
  tenantId?: string;
  type: 'access' | 'refresh';
}

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy, 'jwt') {
  constructor(config: ConfigService) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: config.get('JWT_SECRET'),
    });
  }

  async validate(payload: JwtPayload) {
    if (payload.type !== 'access') {
      throw new UnauthorizedException('Invalid token type');
    }

    return {
      id: payload.sub,
      email: payload.email,
      role: payload.role,
      tenantId: payload.tenantId,
    };
  }
}
```

### 3. Auth Service (Day 1)

```typescript
// apps/api/src/modules/auth/auth.service.ts
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { PrismaService } from '@/database/prisma.service';
import { RedisService } from '@/redis/redis.service';
import * as bcrypt from 'bcrypt';
import { v4 as uuid } from 'uuid';

@Injectable()
export class AuthService {
  constructor(
    private prisma: PrismaService,
    private jwt: JwtService,
    private redis: RedisService,
  ) {}

  async validateUser(email: string, password: string, isStaff = true) {
    const user = isStaff
      ? await this.prisma.user.findFirst({ where: { email } })
      : await this.prisma.guest.findUnique({ where: { email } });

    if (!user || !user.password) {
      throw new UnauthorizedException('Invalid credentials');
    }

    const isValid = await bcrypt.compare(password, user.password);
    if (!isValid) {
      throw new UnauthorizedException('Invalid credentials');
    }

    return user;
  }

  async login(user: any, isStaff = true) {
    const payload = {
      sub: user.id,
      email: user.email,
      role: isStaff ? user.role : 'GUEST',
      tenantId: user.tenantId,
    };

    const jti = uuid();

    const accessToken = this.jwt.sign(
      { ...payload, type: 'access' },
      { expiresIn: '15m' }
    );

    const refreshToken = this.jwt.sign(
      { sub: user.id, type: 'refresh', jti },
      { expiresIn: '7d' }
    );

    // Store refresh token in Redis for invalidation
    await this.redis.set(
      `refresh:${user.id}:${jti}`,
      '1',
      60 * 60 * 24 * 7 // 7 days
    );

    return { accessToken, refreshToken };
  }

  async refresh(refreshToken: string) {
    try {
      const payload = this.jwt.verify(refreshToken);

      if (payload.type !== 'refresh') {
        throw new UnauthorizedException('Invalid token type');
      }

      // Check if token is still valid in Redis
      const isValid = await this.redis.get(`refresh:${payload.sub}:${payload.jti}`);
      if (!isValid) {
        throw new UnauthorizedException('Token revoked');
      }

      // Rotate refresh token
      await this.redis.del(`refresh:${payload.sub}:${payload.jti}`);

      const user = await this.prisma.user.findUnique({
        where: { id: payload.sub },
      });

      if (!user) {
        throw new UnauthorizedException('User not found');
      }

      return this.login(user);
    } catch {
      throw new UnauthorizedException('Invalid refresh token');
    }
  }

  async logout(userId: string, jti: string) {
    await this.redis.del(`refresh:${userId}:${jti}`);
  }

  async logoutAll(userId: string) {
    const keys = await this.redis.keys(`refresh:${userId}:*`);
    if (keys.length > 0) {
      await this.redis.del(...keys);
    }
  }
}
```

### 4. Roles Guard (Day 2)

```typescript
// apps/api/src/modules/auth/guards/roles.guard.ts
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { ROLES_KEY } from '../decorators/roles.decorator';

const ROLE_HIERARCHY = {
  OWNER: 3,
  MANAGER: 2,
  STAFF: 1,
  GUEST: 0,
};

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<string[]>(
      ROLES_KEY,
      [context.getHandler(), context.getClass()]
    );

    if (!requiredRoles) {
      return true;
    }

    const { user } = context.switchToHttp().getRequest();
    const userLevel = ROLE_HIERARCHY[user.role] ?? 0;

    return requiredRoles.some(
      (role) => userLevel >= ROLE_HIERARCHY[role]
    );
  }
}
```

### 5. Decorators (Day 2)

```typescript
// apps/api/src/modules/auth/decorators/roles.decorator.ts
import { SetMetadata } from '@nestjs/common';

export const ROLES_KEY = 'roles';
export const Roles = (...roles: string[]) => SetMetadata(ROLES_KEY, roles);

// apps/api/src/modules/auth/decorators/current-user.decorator.ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const CurrentUser = createParamDecorator(
  (data: string, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    const user = request.user;
    return data ? user?.[data] : user;
  },
);

// apps/api/src/modules/auth/decorators/public.decorator.ts
import { SetMetadata } from '@nestjs/common';

export const IS_PUBLIC_KEY = 'isPublic';
export const Public = () => SetMetadata(IS_PUBLIC_KEY, true);
```

### 6. Auth Controller (Day 2)

```typescript
// apps/api/src/modules/auth/auth.controller.ts
import { Controller, Post, Body, UseGuards, Req } from '@nestjs/common';
import { AuthService } from './auth.service';
import { LoginDto, RegisterDto, RefreshDto } from './dto';
import { Public } from './decorators/public.decorator';
import { CurrentUser } from './decorators/current-user.decorator';

@Controller('auth')
export class AuthController {
  constructor(private authService: AuthService) {}

  @Public()
  @Post('staff/login')
  async staffLogin(@Body() dto: LoginDto) {
    const user = await this.authService.validateUser(
      dto.email,
      dto.password,
      true
    );
    return this.authService.login(user, true);
  }

  @Public()
  @Post('guest/login')
  async guestLogin(@Body() dto: LoginDto) {
    const user = await this.authService.validateUser(
      dto.email,
      dto.password,
      false
    );
    return this.authService.login(user, false);
  }

  @Public()
  @Post('guest/register')
  async guestRegister(@Body() dto: RegisterDto) {
    return this.authService.registerGuest(dto);
  }

  @Public()
  @Post('refresh')
  async refresh(@Body() dto: RefreshDto) {
    return this.authService.refresh(dto.refreshToken);
  }

  @Post('logout')
  async logout(@CurrentUser() user: any, @Req() req: any) {
    const token = req.headers.authorization?.split(' ')[1];
    const payload = this.authService.decode(token);
    await this.authService.logout(user.id, payload.jti);
    return { success: true };
  }

  @Post('logout-all')
  async logoutAll(@CurrentUser() user: any) {
    await this.authService.logoutAll(user.id);
    return { success: true };
  }
}
```

### 7. Auth Module (Day 3)

```typescript
// apps/api/src/modules/auth/auth.module.ts
import { Module, Global } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { PassportModule } from '@nestjs/passport';
import { ConfigService } from '@nestjs/config';
import { AuthService } from './auth.service';
import { AuthController } from './auth.controller';
import { JwtStrategy } from './strategies/jwt.strategy';
import { JwtRefreshStrategy } from './strategies/jwt-refresh.strategy';
import { JwtAuthGuard } from './guards/jwt-auth.guard';
import { RolesGuard } from './guards/roles.guard';

@Global()
@Module({
  imports: [
    PassportModule.register({ defaultStrategy: 'jwt' }),
    JwtModule.registerAsync({
      inject: [ConfigService],
      useFactory: (config: ConfigService) => ({
        secret: config.get('JWT_SECRET'),
        signOptions: { expiresIn: '15m' },
      }),
    }),
  ],
  controllers: [AuthController],
  providers: [
    AuthService,
    JwtStrategy,
    JwtRefreshStrategy,
    {
      provide: 'APP_GUARD',
      useClass: JwtAuthGuard,
    },
    {
      provide: 'APP_GUARD',
      useClass: RolesGuard,
    },
  ],
  exports: [AuthService],
})
export class AuthModule {}
```

### 8. Validation DTOs (Day 3)

```typescript
// packages/shared/src/schemas/auth.schema.ts
import { z } from 'zod';

export const loginSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
});

export const registerSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
  name: z.string().min(2),
  phone: z.string().optional(),
});

export const refreshSchema = z.object({
  refreshToken: z.string(),
});

export type LoginInput = z.infer<typeof loginSchema>;
export type RegisterInput = z.infer<typeof registerSchema>;
```

## Todo List

- [ ] Install Passport.js and JWT dependencies
- [ ] Create JWT access token strategy
- [ ] Create JWT refresh token strategy
- [ ] Implement AuthService with login/refresh/logout
- [ ] Create RolesGuard with hierarchy
- [ ] Create decorators (@Roles, @CurrentUser, @Public)
- [ ] Build AuthController with all endpoints
- [ ] Configure global auth guard
- [ ] Add rate limiting to auth endpoints
- [ ] Create Zod validation schemas
- [ ] Write unit tests for auth service
- [ ] Test token refresh flow

## Success Criteria

- [ ] Staff login returns valid JWT tokens
- [ ] Guest registration + login works
- [ ] Protected routes reject invalid tokens
- [ ] Role-based access enforced (OWNER can access all)
- [ ] Token refresh rotates refresh token
- [ ] Logout invalidates refresh token
- [ ] Rate limiting blocks brute force

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| JWT secret leak | Critical | Use env vars, rotate secrets |
| Token theft | High | Short expiry, refresh rotation |
| Brute force attacks | Medium | Rate limiting, account lockout |

## Security Considerations

- JWT secret minimum 256 bits
- Passwords hashed with bcrypt (cost 10+)
- Refresh tokens stored in Redis with TTL
- Rate limit: 5 login attempts per minute
- HTTPS only in production
- Secure cookie flags if using cookies

## Next Steps

After completion, proceed to [Phase 04: Core API](./phase-04-core-api-trpc-routers-services.md)
