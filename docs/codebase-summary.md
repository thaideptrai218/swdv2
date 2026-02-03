# HostelViet - Codebase Summary

**Version:** 1.0
**Last Updated:** 2026-02-03
**Status:** Planning Phase - Structure Defined

## Project Overview

HostelViet is a Turborepo-based monorepo containing multiple applications and shared packages for a hostel management SaaS platform targeting the Vietnam market.

**Repository:** `/home/welterial/projects/swdv2`
**Architecture:** Monorepo (Turborepo)
**Primary Language:** TypeScript
**Node Version:** 20+
**Package Manager:** pnpm 9+

## Monorepo Structure

```
swdv2/
├── apps/                       # Application packages
│   ├── web/                    # Guest booking portal (Next.js 15)
│   ├── dashboard/              # Owner dashboard (Next.js 15)
│   └── api/                    # Backend API (NestJS)
├── packages/                   # Shared packages
│   ├── ui/                     # Shared UI components (shadcn/ui)
│   ├── config/                 # Shared configurations
│   ├── database/               # Prisma schema and client
│   └── types/                  # Shared TypeScript types
├── docs/                       # Project documentation
├── plans/                      # Implementation plans and reports
├── .github/                    # GitHub workflows and actions
├── turbo.json                  # Turborepo configuration
├── package.json                # Root package.json
├── pnpm-workspace.yaml         # pnpm workspace config
├── .gitignore                  # Git ignore rules
└── README.md                   # Project overview
```

## Applications (apps/)

### 1. Web Portal (apps/web/)

**Purpose:** Public-facing Next.js application for travelers to search and book hostels

**Technology:**
- Framework: Next.js 15 (App Router)
- Styling: Tailwind CSS
- UI Library: shadcn/ui
- State: React Context + tRPC hooks
- Forms: React Hook Form + Zod

**Directory Structure:**
```
apps/web/
├── app/                        # Next.js App Router
│   ├── (public)/               # Public route group
│   │   ├── page.tsx            # Homepage
│   │   ├── search/             # Search results
│   │   └── properties/         # Property listings
│   ├── (auth)/                 # Auth route group
│   │   ├── login/
│   │   └── register/
│   ├── (protected)/            # Protected routes
│   │   ├── profile/
│   │   └── bookings/
│   ├── layout.tsx              # Root layout
│   └── globals.css             # Global styles
├── components/                 # React components
│   ├── property-card.tsx
│   ├── search-form.tsx
│   ├── booking-flow.tsx
│   └── layout/
│       ├── header.tsx
│       └── footer.tsx
├── lib/                        # Utilities and helpers
│   ├── trpc.ts                 # tRPC client setup
│   ├── utils.ts                # Utility functions
│   └── validators.ts           # Zod schemas
├── public/                     # Static assets
│   ├── images/
│   ├── icons/
│   └── favicon.ico
├── next.config.js              # Next.js config
├── tailwind.config.js          # Tailwind config
├── tsconfig.json               # TypeScript config
└── package.json
```

**Key Features:**
- Server-side rendering for SEO
- Full-text search with filters
- Real-time availability checking
- Multi-step booking flow
- VietQR/VNPay payment integration
- Guest profile management

**Environment Variables:**
```bash
NEXT_PUBLIC_API_URL=http://localhost:4000
NEXT_PUBLIC_WEB_URL=http://localhost:3000
```

### 2. Owner Dashboard (apps/dashboard/)

**Purpose:** Management interface for hostel owners and staff

**Technology:**
- Framework: Next.js 15 (App Router)
- Styling: Tailwind CSS + shadcn/ui
- Charts: Recharts
- State: Zustand + tRPC
- Real-time: Server-Sent Events (SSE)

**Directory Structure:**
```
apps/dashboard/
├── app/
│   ├── (auth)/
│   │   └── login/
│   ├── (dashboard)/            # Dashboard layout group
│   │   ├── layout.tsx          # Dashboard layout with sidebar
│   │   ├── page.tsx            # Dashboard overview
│   │   ├── properties/         # Property management
│   │   ├── bookings/           # Booking calendar
│   │   ├── analytics/          # Revenue reports
│   │   ├── staff/              # Staff management
│   │   ├── settings/           # Account settings
│   │   └── subscription/       # Billing
│   └── layout.tsx
├── components/
│   ├── analytics/
│   │   ├── revenue-chart.tsx
│   │   ├── occupancy-chart.tsx
│   │   └── stats-card.tsx
│   ├── bookings/
│   │   ├── booking-calendar.tsx
│   │   └── booking-list.tsx
│   ├── properties/
│   │   ├── property-form.tsx
│   │   └── room-management.tsx
│   └── layout/
│       ├── sidebar.tsx
│       └── top-bar.tsx
├── lib/
│   ├── trpc.ts
│   ├── sse-client.ts           # SSE connection handler
│   └── store.ts                # Zustand store
├── public/
└── package.json
```

**Key Features:**
- Real-time booking notifications
- Interactive analytics dashboard
- Multi-property management
- Staff role and permission management
- Subscription and billing
- Booking calendar with drag-and-drop

**Environment Variables:**
```bash
NEXT_PUBLIC_API_URL=http://localhost:4000
NEXT_PUBLIC_DASHBOARD_URL=http://localhost:3001
```

### 3. Backend API (apps/api/)

**Purpose:** NestJS backend providing tRPC and REST APIs

**Technology:**
- Framework: NestJS
- API: tRPC + REST webhooks
- Auth: Passport.js (JWT strategy)
- Validation: class-validator
- ORM: Prisma
- Real-time: Server-Sent Events

**Directory Structure:**
```
apps/api/
├── src/
│   ├── main.ts                 # Application entry point
│   ├── app.module.ts           # Root module
│   ├── trpc/                   # tRPC configuration
│   │   ├── trpc.module.ts
│   │   ├── trpc.router.ts      # Router aggregation
│   │   └── context.ts          # tRPC context
│   ├── auth/                   # Authentication module
│   │   ├── auth.module.ts
│   │   ├── auth.service.ts
│   │   ├── auth.controller.ts
│   │   ├── strategies/
│   │   │   ├── jwt.strategy.ts
│   │   │   └── refresh.strategy.ts
│   │   └── guards/
│   │       ├── jwt-auth.guard.ts
│   │       └── roles.guard.ts
│   ├── user/                   # User management
│   │   ├── user.module.ts
│   │   ├── user.service.ts
│   │   ├── user.repository.ts
│   │   └── dto/
│   │       └── create-user.dto.ts
│   ├── property/               # Property domain
│   │   ├── property.module.ts
│   │   ├── property.service.ts
│   │   ├── property.repository.ts
│   │   ├── property.router.ts  # tRPC router
│   │   └── dto/
│   ├── booking/                # Booking domain
│   │   ├── booking.module.ts
│   │   ├── booking.service.ts
│   │   ├── booking.repository.ts
│   │   ├── booking.router.ts
│   │   └── booking.events.ts   # Event emitters
│   ├── payment/                # Payment integration
│   │   ├── payment.module.ts
│   │   ├── payment.service.ts
│   │   ├── providers/
│   │   │   ├── sepay.provider.ts
│   │   │   └── vnpay.provider.ts
│   │   └── webhooks/
│   │       └── payment-webhook.controller.ts
│   ├── subscription/           # Subscription management
│   │   ├── subscription.module.ts
│   │   ├── subscription.service.ts
│   │   └── subscription.router.ts
│   ├── notification/           # Email/SMS notifications
│   │   ├── notification.module.ts
│   │   ├── notification.service.ts
│   │   ├── email.service.ts
│   │   └── sms.service.ts
│   ├── sse/                    # Server-Sent Events
│   │   ├── sse.module.ts
│   │   ├── sse.gateway.ts
│   │   └── sse.service.ts
│   ├── common/                 # Shared utilities
│   │   ├── decorators/
│   │   │   ├── current-user.decorator.ts
│   │   │   └── roles.decorator.ts
│   │   ├── filters/
│   │   │   └── http-exception.filter.ts
│   │   ├── interceptors/
│   │   │   ├── logging.interceptor.ts
│   │   │   └── timeout.interceptor.ts
│   │   ├── pipes/
│   │   │   └── validation.pipe.ts
│   │   └── middleware/
│   │       └── tenant-context.middleware.ts
│   └── config/                 # Configuration
│       ├── config.module.ts
│       ├── database.config.ts
│       ├── redis.config.ts
│       └── jwt.config.ts
├── test/                       # E2E tests
│   ├── auth.e2e-spec.ts
│   ├── booking.e2e-spec.ts
│   └── property.e2e-spec.ts
├── nest-cli.json               # NestJS CLI config
├── tsconfig.json
└── package.json
```

**Key Modules:**

| Module | Purpose | Dependencies |
|--------|---------|--------------|
| AuthModule | JWT authentication, role-based access | Passport, bcrypt |
| UserModule | User CRUD, profile management | Prisma |
| PropertyModule | Property and room management | Prisma, S3 |
| BookingModule | Booking lifecycle, availability | Prisma, Redis |
| PaymentModule | Payment processing, webhooks | SePay, VNPay |
| SubscriptionModule | Subscription plans, billing | Prisma, Stripe |
| NotificationModule | Email and SMS notifications | SendGrid, Twilio |
| SSEModule | Real-time updates | EventEmitter |

**Environment Variables:**
```bash
DATABASE_URL=postgresql://...
REDIS_URL=redis://localhost:6379
JWT_ACCESS_SECRET=...
JWT_REFRESH_SECRET=...
SEPAY_API_KEY=...
VNPAY_TMN_CODE=...
AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...
```

## Shared Packages (packages/)

### 1. UI Package (packages/ui/)

**Purpose:** Shared React components based on shadcn/ui

**Structure:**
```
packages/ui/
├── src/
│   ├── button.tsx              # Button variants
│   ├── card.tsx                # Card components
│   ├── form.tsx                # Form elements
│   ├── input.tsx               # Input fields
│   ├── select.tsx              # Select dropdowns
│   ├── dialog.tsx              # Modal dialogs
│   ├── table.tsx               # Data tables
│   ├── toast.tsx               # Notifications
│   └── index.ts                # Barrel exports
├── tailwind.config.js
├── tsconfig.json
└── package.json
```

**Usage:**
```typescript
import { Button, Card, Input } from '@hostelviet/ui';
```

### 2. Config Package (packages/config/)

**Purpose:** Shared configurations for ESLint, TypeScript, and Tailwind

**Structure:**
```
packages/config/
├── eslint/
│   ├── base.js                 # Base ESLint config
│   ├── nextjs.js               # Next.js specific
│   └── nestjs.js               # NestJS specific
├── typescript/
│   ├── base.json               # Base TS config
│   ├── nextjs.json             # Next.js TS config
│   └── nestjs.json             # NestJS TS config
├── tailwind/
│   └── tailwind.config.js      # Shared Tailwind config
└── package.json
```

**Usage:**
```json
// apps/web/tsconfig.json
{
  "extends": "@hostelviet/config/typescript/nextjs.json"
}
```

### 3. Database Package (packages/database/)

**Purpose:** Prisma schema and database client

**Structure:**
```
packages/database/
├── prisma/
│   ├── schema.prisma           # Database schema
│   ├── migrations/             # Migration files
│   │   └── YYYYMMDDHHMMSS_init/
│   └── seed.ts                 # Database seeding
├── src/
│   ├── client.ts               # Prisma client singleton
│   ├── types.ts                # Generated types
│   └── index.ts
├── package.json
└── README.md
```

**Key Entities:**
- Tenant (hostel organization)
- User (owners, staff, guests)
- Property (hostels)
- Room (physical rooms)
- Bed (individual beds in dorms)
- Booking (reservations)
- Subscription (billing plans)
- Payment (payment records)
- Review (guest reviews)

**Usage:**
```typescript
import { prisma } from '@hostelviet/database';

const properties = await prisma.property.findMany();
```

### 4. Types Package (packages/types/)

**Purpose:** Shared TypeScript types and interfaces

**Structure:**
```
packages/types/
├── src/
│   ├── api/                    # API response types
│   │   ├── property.ts
│   │   ├── booking.ts
│   │   └── user.ts
│   ├── models/                 # Domain models
│   │   ├── property.ts
│   │   ├── booking.ts
│   │   └── user.ts
│   ├── common/                 # Utility types
│   │   ├── pagination.ts
│   │   └── response.ts
│   └── index.ts
├── tsconfig.json
└── package.json
```

**Usage:**
```typescript
import type { Property, Booking, ApiResponse } from '@hostelviet/types';
```

## Configuration Files

### Root Configuration

#### turbo.json
```json
{
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [".next/**", "dist/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "lint": {},
    "test": {
      "dependsOn": ["^build"]
    }
  }
}
```

#### pnpm-workspace.yaml
```yaml
packages:
  - 'apps/*'
  - 'packages/*'
```

#### package.json (Root)
```json
{
  "name": "hostelviet",
  "private": true,
  "scripts": {
    "dev": "turbo run dev",
    "build": "turbo run build",
    "test": "turbo run test",
    "lint": "turbo run lint",
    "format": "prettier --write \"**/*.{ts,tsx,md}\"",
    "db:migrate": "pnpm --filter @hostelviet/database db:migrate",
    "db:seed": "pnpm --filter @hostelviet/database db:seed"
  },
  "devDependencies": {
    "turbo": "^2.0.0",
    "prettier": "^3.0.0",
    "typescript": "^5.3.0"
  }
}
```

## Development Workflow

### Local Development

```bash
# Install dependencies
pnpm install

# Start all apps in dev mode
pnpm dev

# Start specific app
pnpm dev --filter=web
pnpm dev --filter=dashboard
pnpm dev --filter=api

# Run database migrations
pnpm db:migrate

# Seed database
pnpm db:seed
```

### Build Process

```bash
# Build all apps
pnpm build

# Build specific app
pnpm build --filter=web

# Type checking
pnpm typecheck

# Linting
pnpm lint

# Run tests
pnpm test
```

### Testing Structure

```
apps/api/
├── src/
│   └── **/*.spec.ts            # Unit tests (co-located)
├── test/
│   └── **/*.e2e-spec.ts        # E2E tests

apps/web/
├── __tests__/
│   ├── components/             # Component tests
│   └── pages/                  # Page tests
```

## Dependencies Overview

### Core Dependencies

**Frontend (Next.js):**
- next: ^15.0.0
- react: ^19.0.0
- react-dom: ^19.0.0
- @trpc/client: ^11.0.0
- @tanstack/react-query: ^5.0.0
- tailwindcss: ^3.4.0
- zod: ^3.22.0

**Backend (NestJS):**
- @nestjs/core: ^10.0.0
- @nestjs/common: ^10.0.0
- @trpc/server: ^11.0.0
- @prisma/client: ^5.0.0
- passport-jwt: ^4.0.0
- bcrypt: ^5.1.0
- class-validator: ^0.14.0
- class-transformer: ^0.5.0

**Database:**
- prisma: ^5.0.0
- @prisma/client: ^5.0.0

**Dev Dependencies:**
- typescript: ^5.3.0
- @types/node: ^20.0.0
- eslint: ^8.0.0
- prettier: ^3.0.0
- jest: ^29.0.0
- vitest: ^1.0.0

## Code Organization Patterns

### File Naming
- **Components:** kebab-case (e.g., `booking-card.tsx`)
- **Classes:** PascalCase files (e.g., `BookingService`)
- **Utilities:** kebab-case (e.g., `date-utils.ts`)
- **Constants:** SCREAMING_SNAKE_CASE in files

### Module Structure (NestJS)
```
feature/
├── feature.module.ts           # Module definition
├── feature.service.ts          # Business logic
├── feature.controller.ts       # HTTP/tRPC endpoints
├── feature.repository.ts       # Data access
├── feature.router.ts           # tRPC router (if applicable)
├── dto/                        # Data transfer objects
│   ├── create-feature.dto.ts
│   └── update-feature.dto.ts
└── __tests__/
    └── feature.service.spec.ts
```

### Component Structure (React)
```typescript
// 1. Imports
import { useState } from 'react';
import { Button } from '@hostelviet/ui';

// 2. Types
interface ComponentProps {
  // ...
}

// 3. Component
export function Component(props: ComponentProps) {
  // State
  // Effects
  // Handlers
  // Render
}

// 4. Exports
```

## Key Entry Points

| Application | Entry Point | Port |
|-------------|-------------|------|
| Web Portal | `apps/web/app/page.tsx` | 3000 |
| Dashboard | `apps/dashboard/app/page.tsx` | 3001 |
| API | `apps/api/src/main.ts` | 4000 |

## Documentation

All documentation is maintained in `/docs`:
- [README.md](../README.md) - Project overview
- [project-overview-pdr.md](./project-overview-pdr.md) - Product requirements
- [code-standards.md](./code-standards.md) - Coding conventions
- [system-architecture.md](./system-architecture.md) - Architecture design
- [design-guidelines.md](./design-guidelines.md) - UI/UX guidelines

## Implementation Plans

Detailed implementation plans are in `/plans/260203-1316-hostel-management-saas/`:
- Phase 01: Project setup, monorepo, CI/CD
- Phase 02: Database design, Prisma schema, RLS
- Phase 03: Auth system (JWT, roles, guards)
- Phase 04: Core API (tRPC routers, services)
- Phase 05: Owner dashboard UI
- Phase 06: Guest platform UI
- Phase 07: Payment integration (SePay, VNPay)
- Phase 08: Real-time notifications (SSE)
- Phase 09: Monitoring (ELK, Prometheus, Grafana)
- Phase 10: Testing and deployment

## Current Status

**Phase:** Planning Complete
**Next Steps:**
1. Initialize Turborepo with apps and packages
2. Setup Prisma schema and migrations
3. Implement authentication system
4. Build core API endpoints
5. Develop UI components and pages

---

**Last Updated:** 2026-02-03
**Maintained By:** Development Team
**Update Trigger:** After major structural changes
