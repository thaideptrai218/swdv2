---
title: "Phase 02: Database Design"
status: pending
priority: P1
effort: 4d
---

# Phase 02: Database Design - Prisma Schema, RLS, Migrations

## Context Links

- [Tech Architecture Report](../reports/researcher-260203-1204-tech-architecture.md)
- [Phase 01: Project Setup](./phase-01-project-setup-monorepo-configs-cicd.md)

## Overview

Design multi-tenant database schema with Prisma ORM. Implement Row-Level Security (RLS) for tenant isolation. Create initial migrations and seed data.

## Key Insights

- Shared schema + RLS pattern: cost-effective, single DB
- All tenant-scoped tables include `tenant_id` column
- JSONB for flexible fields (amenities, settings, metadata)
- Soft deletes required for financial/booking records

## Requirements

### Functional
- Tenant (hostel organization) management
- Property/hostel records with address, settings
- Room types with bed configurations
- Individual bed tracking for dorms
- Booking with check-in/out, status workflow
- Guest profiles with booking history
- Staff users with role-based permissions
- Subscription and invoice tracking

### Non-Functional
- Query performance: <100ms for common operations
- RLS prevents cross-tenant data access
- Audit trail on all mutations
- Soft delete preservation

## Architecture

### Entity Relationship

```
Tenant (1) ─── (N) Property
Tenant (1) ─── (N) User (staff)
Tenant (1) ─── (N) Subscription

Property (1) ─── (N) Room
Room (1) ─── (N) Bed
Bed (N) ─── (N) Booking (via BookingBed)

Guest (1) ─── (N) Booking
Booking (1) ─── (N) Payment

Tenant (1) ─── (N) Invoice
```

### Multi-Tenancy Pattern

```sql
-- Every tenant-scoped table has:
tenant_id UUID NOT NULL REFERENCES tenants(id)

-- RLS Policy
ALTER TABLE bookings ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation ON bookings
  USING (tenant_id = current_setting('app.current_tenant')::UUID);
```

## Related Code Files

### Create
- `packages/database/prisma/schema.prisma`
- `packages/database/prisma/seed.ts`
- `packages/database/src/index.ts` - Prisma client export
- `packages/database/src/rls.sql` - RLS policies
- `apps/api/src/database/tenant-context.middleware.ts`

### Modify
- `packages/database/package.json` - add Prisma deps

## Implementation Steps

### 1. Core Schema Design (Day 1)

```prisma
// packages/database/prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// ─── TENANT ────────────────────────────────────────────
model Tenant {
  id          String   @id @default(uuid()) @db.Uuid
  name        String
  slug        String   @unique
  email       String   @unique
  phone       String?
  settings    Json     @default("{}")

  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  deletedAt   DateTime?

  properties    Property[]
  users         User[]
  subscriptions Subscription[]
  invoices      Invoice[]
  guests        Guest[]
  bookings      Booking[]

  @@index([slug])
}

// ─── USER (Staff) ──────────────────────────────────────
model User {
  id          String   @id @default(uuid()) @db.Uuid
  tenantId    String   @db.Uuid
  email       String
  password    String
  name        String
  role        UserRole @default(STAFF)
  permissions Json     @default("[]")
  isActive    Boolean  @default(true)

  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  deletedAt   DateTime?

  tenant      Tenant   @relation(fields: [tenantId], references: [id])

  @@unique([tenantId, email])
  @@index([tenantId])
}

enum UserRole {
  OWNER
  MANAGER
  STAFF
}

// ─── PROPERTY ──────────────────────────────────────────
model Property {
  id          String   @id @default(uuid()) @db.Uuid
  tenantId    String   @db.Uuid
  name        String
  description String?
  address     String
  city        String
  district    String?
  latitude    Float?
  longitude   Float?
  amenities   Json     @default("[]")
  images      Json     @default("[]")
  isActive    Boolean  @default(true)

  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  deletedAt   DateTime?

  tenant      Tenant   @relation(fields: [tenantId], references: [id])
  rooms       Room[]

  @@index([tenantId])
  @@index([city])
}

// ─── ROOM ──────────────────────────────────────────────
model Room {
  id          String   @id @default(uuid()) @db.Uuid
  propertyId  String   @db.Uuid
  name        String
  type        RoomType
  description String?
  capacity    Int
  basePrice   Decimal  @db.Decimal(12, 0)
  amenities   Json     @default("[]")
  images      Json     @default("[]")
  isActive    Boolean  @default(true)

  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  deletedAt   DateTime?

  property    Property @relation(fields: [propertyId], references: [id])
  beds        Bed[]

  @@index([propertyId])
}

enum RoomType {
  DORM_MIXED
  DORM_FEMALE
  DORM_MALE
  PRIVATE_SINGLE
  PRIVATE_DOUBLE
  PRIVATE_TWIN
  PRIVATE_FAMILY
}

// ─── BED ───────────────────────────────────────────────
model Bed {
  id          String   @id @default(uuid()) @db.Uuid
  roomId      String   @db.Uuid
  name        String   // "Bed 1", "Top Bunk A"
  type        BedType  @default(SINGLE)
  priceModifier Decimal @default(0) @db.Decimal(12, 0)
  isActive    Boolean  @default(true)

  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  room        Room     @relation(fields: [roomId], references: [id])
  bookingBeds BookingBed[]

  @@index([roomId])
}

enum BedType {
  SINGLE
  DOUBLE
  BUNK_TOP
  BUNK_BOTTOM
}

// ─── GUEST ─────────────────────────────────────────────
model Guest {
  id          String   @id @default(uuid()) @db.Uuid
  tenantId    String?  @db.Uuid
  email       String   @unique
  password    String?
  name        String
  phone       String?
  nationality String?
  idType      String?
  idNumber    String?

  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  tenant      Tenant?  @relation(fields: [tenantId], references: [id])
  bookings    Booking[]
  reviews     Review[]

  @@index([email])
}

// ─── BOOKING ───────────────────────────────────────────
model Booking {
  id          String        @id @default(uuid()) @db.Uuid
  tenantId    String        @db.Uuid
  guestId     String        @db.Uuid
  checkIn     DateTime      @db.Date
  checkOut    DateTime      @db.Date
  status      BookingStatus @default(PENDING)
  totalPrice  Decimal       @db.Decimal(12, 0)
  currency    String        @default("VND")
  notes       String?
  metadata    Json          @default("{}")

  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  cancelledAt DateTime?

  tenant      Tenant   @relation(fields: [tenantId], references: [id])
  guest       Guest    @relation(fields: [guestId], references: [id])
  bookingBeds BookingBed[]
  payments    Payment[]

  @@index([tenantId])
  @@index([guestId])
  @@index([checkIn, checkOut])
  @@index([status])
}

enum BookingStatus {
  PENDING
  CONFIRMED
  CHECKED_IN
  CHECKED_OUT
  CANCELLED
  NO_SHOW
}

// ─── BOOKING BED (Junction) ────────────────────────────
model BookingBed {
  id        String   @id @default(uuid()) @db.Uuid
  bookingId String   @db.Uuid
  bedId     String   @db.Uuid
  pricePerNight Decimal @db.Decimal(12, 0)

  booking   Booking  @relation(fields: [bookingId], references: [id])
  bed       Bed      @relation(fields: [bedId], references: [id])

  @@unique([bookingId, bedId])
  @@index([bedId])
}

// ─── PAYMENT ───────────────────────────────────────────
model Payment {
  id          String        @id @default(uuid()) @db.Uuid
  bookingId   String        @db.Uuid
  amount      Decimal       @db.Decimal(12, 0)
  currency    String        @default("VND")
  method      PaymentMethod
  status      PaymentStatus @default(PENDING)
  provider    String?       // sepay, vnpay
  providerRef String?       // external transaction ID
  refCode     String?       @unique
  paidAt      DateTime?
  metadata    Json          @default("{}")

  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  booking     Booking  @relation(fields: [bookingId], references: [id])

  @@index([bookingId])
  @@index([refCode])
  @@index([status])
}

enum PaymentMethod {
  VIETQR
  VNPAY_CARD
  VNPAY_WALLET
  BANK_TRANSFER
  CASH
}

enum PaymentStatus {
  PENDING
  PROCESSING
  COMPLETED
  FAILED
  REFUNDED
}

// ─── SUBSCRIPTION ──────────────────────────────────────
model Subscription {
  id          String   @id @default(uuid()) @db.Uuid
  tenantId    String   @db.Uuid
  plan        SubscriptionPlan
  status      SubscriptionStatus @default(ACTIVE)
  startDate   DateTime
  endDate     DateTime

  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  tenant      Tenant   @relation(fields: [tenantId], references: [id])

  @@index([tenantId])
  @@index([status])
}

enum SubscriptionPlan {
  BASIC
  PRO
  ENTERPRISE
}

enum SubscriptionStatus {
  ACTIVE
  PAST_DUE
  SUSPENDED
  CANCELLED
}

// ─── INVOICE ───────────────────────────────────────────
model Invoice {
  id          String   @id @default(uuid()) @db.Uuid
  tenantId    String   @db.Uuid
  amount      Decimal  @db.Decimal(12, 0)
  currency    String   @default("VND")
  status      InvoiceStatus @default(PENDING)
  dueDate     DateTime
  paidAt      DateTime?
  refCode     String   @unique

  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  tenant      Tenant   @relation(fields: [tenantId], references: [id])

  @@index([tenantId])
  @@index([status])
}

enum InvoiceStatus {
  PENDING
  PAID
  OVERDUE
  CANCELLED
}

// ─── REVIEW ────────────────────────────────────────────
model Review {
  id          String   @id @default(uuid()) @db.Uuid
  guestId     String   @db.Uuid
  propertyId  String   @db.Uuid
  rating      Int      // 1-5
  comment     String?
  isPublished Boolean  @default(false)

  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  guest       Guest    @relation(fields: [guestId], references: [id])

  @@index([propertyId])
  @@index([guestId])
}
```

### 2. RLS Policies (Day 2)

```sql
-- packages/database/prisma/rls.sql

-- Enable RLS on tenant-scoped tables
ALTER TABLE "Property" ENABLE ROW LEVEL SECURITY;
ALTER TABLE "Room" ENABLE ROW LEVEL SECURITY;
ALTER TABLE "Booking" ENABLE ROW LEVEL SECURITY;
ALTER TABLE "User" ENABLE ROW LEVEL SECURITY;

-- Tenant isolation policies
CREATE POLICY tenant_isolation_property ON "Property"
  USING ("tenantId" = current_setting('app.current_tenant', true)::UUID);

CREATE POLICY tenant_isolation_room ON "Room"
  USING ("propertyId" IN (
    SELECT id FROM "Property"
    WHERE "tenantId" = current_setting('app.current_tenant', true)::UUID
  ));

CREATE POLICY tenant_isolation_booking ON "Booking"
  USING ("tenantId" = current_setting('app.current_tenant', true)::UUID);

CREATE POLICY tenant_isolation_user ON "User"
  USING ("tenantId" = current_setting('app.current_tenant', true)::UUID);

-- Bypass for service role (migrations, admin)
CREATE ROLE service_role;
ALTER TABLE "Property" FORCE ROW LEVEL SECURITY;
CREATE POLICY service_bypass ON "Property" TO service_role USING (true);
```

### 3. Tenant Context Middleware (Day 2)

```typescript
// apps/api/src/database/tenant-context.middleware.ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import { PrismaService } from './prisma.service';

@Injectable()
export class TenantContextMiddleware implements NestMiddleware {
  constructor(private prisma: PrismaService) {}

  async use(req: any, res: any, next: () => void) {
    const tenantId = req.user?.tenantId;

    if (tenantId) {
      await this.prisma.$executeRawUnsafe(
        `SET app.current_tenant = '${tenantId}'`
      );
    }

    next();
  }
}
```

### 4. Indexes and Performance (Day 3)

```sql
-- Availability search optimization
CREATE INDEX idx_booking_availability
  ON "Booking" ("checkIn", "checkOut")
  WHERE status NOT IN ('CANCELLED', 'NO_SHOW');

-- Full-text search on properties
CREATE INDEX idx_property_search
  ON "Property" USING GIN (to_tsvector('english', name || ' ' || COALESCE(description, '') || ' ' || city));

-- Composite indexes for common queries
CREATE INDEX idx_room_property_active
  ON "Room" ("propertyId")
  WHERE "isActive" = true AND "deletedAt" IS NULL;
```

### 5. Seed Data (Day 3)

```typescript
// packages/database/prisma/seed.ts
import { PrismaClient } from '@prisma/client';
import { hash } from 'bcrypt';

const prisma = new PrismaClient();

async function main() {
  // Create demo tenant
  const tenant = await prisma.tenant.create({
    data: {
      name: 'Demo Hostel',
      slug: 'demo-hostel',
      email: 'demo@example.com',
    },
  });

  // Create owner user
  await prisma.user.create({
    data: {
      tenantId: tenant.id,
      email: 'owner@demo.com',
      password: await hash('password123', 10),
      name: 'Demo Owner',
      role: 'OWNER',
    },
  });

  // Create property with rooms and beds
  const property = await prisma.property.create({
    data: {
      tenantId: tenant.id,
      name: 'Backpacker Paradise',
      address: '123 Bui Vien, District 1',
      city: 'Ho Chi Minh City',
      rooms: {
        create: [
          {
            name: '8-Bed Mixed Dorm',
            type: 'DORM_MIXED',
            capacity: 8,
            basePrice: 150000,
            beds: {
              create: Array.from({ length: 8 }, (_, i) => ({
                name: `Bed ${i + 1}`,
                type: i % 2 === 0 ? 'BUNK_TOP' : 'BUNK_BOTTOM',
              })),
            },
          },
          {
            name: 'Private Double',
            type: 'PRIVATE_DOUBLE',
            capacity: 2,
            basePrice: 450000,
            beds: {
              create: [{ name: 'Queen Bed', type: 'DOUBLE' }],
            },
          },
        ],
      },
    },
  });

  console.log('Seed completed:', { tenant, property });
}

main()
  .catch(console.error)
  .finally(() => prisma.$disconnect());
```

### 6. Prisma Client Export (Day 4)

```typescript
// packages/database/src/index.ts
export * from '@prisma/client';
export { PrismaClient } from '@prisma/client';
```

### 7. Migration Scripts (Day 4)

```json
// packages/database/package.json scripts
{
  "scripts": {
    "db:generate": "prisma generate",
    "db:push": "prisma db push",
    "db:migrate": "prisma migrate dev",
    "db:migrate:deploy": "prisma migrate deploy",
    "db:seed": "ts-node prisma/seed.ts",
    "db:studio": "prisma studio"
  }
}
```

## Todo List

- [ ] Create Prisma schema with all entities
- [ ] Define enums for status fields
- [ ] Add JSONB columns for flexible data
- [ ] Configure indexes for common queries
- [ ] Write RLS policies SQL
- [ ] Create tenant context middleware
- [ ] Write seed script with demo data
- [ ] Run initial migration
- [ ] Test RLS isolation
- [ ] Document schema in README

## Success Criteria

- [ ] `prisma generate` produces client without errors
- [ ] `prisma migrate dev` creates tables successfully
- [ ] `prisma db seed` populates demo data
- [ ] RLS policies block cross-tenant queries
- [ ] Indexes visible in database
- [ ] Query performance <100ms for single tenant

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| RLS performance overhead | Medium | Benchmark, optimize policies |
| Migration conflicts | High | Test migrations in staging first |
| JSONB query complexity | Low | Define Zod schemas for validation |

## Security Considerations

- RLS enforced at database level (defense in depth)
- Passwords hashed with bcrypt (cost factor 10+)
- Sensitive fields (ID numbers) encrypt at application level
- Audit columns track who/when for changes

## Next Steps

After completion, proceed to [Phase 03: Auth System](./phase-03-auth-system-jwt-roles-guards.md)
