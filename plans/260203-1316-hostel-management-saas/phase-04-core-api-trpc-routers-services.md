---
title: "Phase 04: Core API"
status: pending
priority: P1
effort: 5d
---

# Phase 04: Core API - tRPC Routers, Services

## Context Links

- [Tech Architecture Report](../reports/researcher-260203-1204-tech-architecture.md)
- [Phase 03: Auth System](./phase-03-auth-system-jwt-roles-guards.md)

## Overview

Build tRPC API layer with type-safe routers for properties, rooms, beds, bookings, and guests. Implement business logic services. Add REST endpoints for webhooks.

## Key Insights

- tRPC for internal frontend-backend communication (type safety)
- REST for external integrations (payment webhooks)
- Service layer separates business logic from routers
- Tenant context automatically applied to all queries

## Requirements

### Functional
- Property CRUD with amenities, images
- Room CRUD with bed management
- Booking creation with availability check
- Guest management with booking history
- Availability search by date range
- Dashboard statistics

### Non-Functional
- API response <200ms for single resource
- Availability check <500ms
- Input validation on all endpoints
- Proper error responses

## Architecture

### tRPC Router Structure

```
appRouter
├── property
│   ├── list
│   ├── getById
│   ├── create
│   ├── update
│   └── delete
├── room
│   ├── list
│   ├── getById
│   ├── create
│   ├── update
│   └── delete
├── bed
│   ├── list
│   └── update
├── booking
│   ├── list
│   ├── getById
│   ├── create
│   ├── updateStatus
│   └── cancel
├── guest
│   ├── list
│   ├── getById
│   └── update
├── availability
│   ├── check
│   └── search
└── stats
    └── dashboard
```

### Service Pattern

```typescript
// Router → Service → Repository (Prisma)
// Router handles auth, validation
// Service contains business logic
// Prisma handles data access
```

## Related Code Files

### Create
- `apps/api/src/trpc/trpc.module.ts`
- `apps/api/src/trpc/trpc.router.ts`
- `apps/api/src/trpc/context.ts`
- `apps/api/src/modules/property/property.router.ts`
- `apps/api/src/modules/property/property.service.ts`
- `apps/api/src/modules/room/room.router.ts`
- `apps/api/src/modules/room/room.service.ts`
- `apps/api/src/modules/booking/booking.router.ts`
- `apps/api/src/modules/booking/booking.service.ts`
- `apps/api/src/modules/booking/availability.service.ts`
- `apps/api/src/modules/guest/guest.router.ts`
- `apps/api/src/modules/guest/guest.service.ts`
- `apps/api/src/modules/stats/stats.router.ts`
- `packages/shared/src/schemas/property.schema.ts`
- `packages/shared/src/schemas/booking.schema.ts`
- `apps/web/src/trpc/client.ts`
- `apps/web/src/trpc/server.ts`

## Implementation Steps

### 1. tRPC Setup (Day 1)

```bash
cd apps/api
pnpm add @trpc/server zod
cd ../web
pnpm add @trpc/client @trpc/react-query @tanstack/react-query
```

### 2. tRPC Context (Day 1)

```typescript
// apps/api/src/trpc/context.ts
import { inferAsyncReturnType } from '@trpc/server';
import { CreateExpressContextOptions } from '@trpc/server/adapters/express';
import { PrismaService } from '@/database/prisma.service';

export interface CreateContextOptions {
  prisma: PrismaService;
  user?: {
    id: string;
    email: string;
    role: string;
    tenantId?: string;
  };
}

export function createContext(prisma: PrismaService) {
  return async ({ req }: CreateExpressContextOptions): Promise<CreateContextOptions> => {
    return {
      prisma,
      user: req.user as any,
    };
  };
}

export type Context = inferAsyncReturnType<ReturnType<typeof createContext>>;
```

### 3. Base Router Setup (Day 1)

```typescript
// apps/api/src/trpc/trpc.router.ts
import { initTRPC, TRPCError } from '@trpc/server';
import { Context } from './context';

const t = initTRPC.context<Context>().create();

export const router = t.router;
export const publicProcedure = t.procedure;

export const protectedProcedure = t.procedure.use(({ ctx, next }) => {
  if (!ctx.user) {
    throw new TRPCError({ code: 'UNAUTHORIZED' });
  }
  return next({
    ctx: {
      ...ctx,
      user: ctx.user,
    },
  });
});

export const ownerProcedure = protectedProcedure.use(({ ctx, next }) => {
  if (ctx.user.role !== 'OWNER' && ctx.user.role !== 'MANAGER') {
    throw new TRPCError({ code: 'FORBIDDEN' });
  }
  return next({ ctx });
});
```

### 4. Property Router (Day 2)

```typescript
// apps/api/src/modules/property/property.router.ts
import { z } from 'zod';
import { router, protectedProcedure, ownerProcedure } from '@/trpc/trpc.router';
import { propertyCreateSchema, propertyUpdateSchema } from '@repo/shared';

export const propertyRouter = router({
  list: protectedProcedure.query(async ({ ctx }) => {
    return ctx.prisma.property.findMany({
      where: {
        tenantId: ctx.user.tenantId,
        deletedAt: null,
      },
      include: {
        rooms: {
          where: { deletedAt: null },
          include: { beds: true },
        },
      },
      orderBy: { createdAt: 'desc' },
    });
  }),

  getById: protectedProcedure
    .input(z.object({ id: z.string().uuid() }))
    .query(async ({ ctx, input }) => {
      const property = await ctx.prisma.property.findFirst({
        where: {
          id: input.id,
          tenantId: ctx.user.tenantId,
          deletedAt: null,
        },
        include: {
          rooms: {
            where: { deletedAt: null },
            include: { beds: true },
          },
        },
      });

      if (!property) {
        throw new TRPCError({ code: 'NOT_FOUND' });
      }

      return property;
    }),

  create: ownerProcedure
    .input(propertyCreateSchema)
    .mutation(async ({ ctx, input }) => {
      return ctx.prisma.property.create({
        data: {
          ...input,
          tenantId: ctx.user.tenantId,
        },
      });
    }),

  update: ownerProcedure
    .input(propertyUpdateSchema)
    .mutation(async ({ ctx, input }) => {
      const { id, ...data } = input;
      return ctx.prisma.property.update({
        where: { id },
        data,
      });
    }),

  delete: ownerProcedure
    .input(z.object({ id: z.string().uuid() }))
    .mutation(async ({ ctx, input }) => {
      return ctx.prisma.property.update({
        where: { id: input.id },
        data: { deletedAt: new Date() },
      });
    }),
});
```

### 5. Booking Service (Day 2-3)

```typescript
// apps/api/src/modules/booking/booking.service.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '@/database/prisma.service';
import { TRPCError } from '@trpc/server';

@Injectable()
export class BookingService {
  constructor(private prisma: PrismaService) {}

  async checkAvailability(
    bedIds: string[],
    checkIn: Date,
    checkOut: Date,
  ): Promise<boolean> {
    const conflictingBookings = await this.prisma.bookingBed.findMany({
      where: {
        bedId: { in: bedIds },
        booking: {
          status: { notIn: ['CANCELLED', 'NO_SHOW'] },
          OR: [
            {
              checkIn: { lt: checkOut },
              checkOut: { gt: checkIn },
            },
          ],
        },
      },
    });

    return conflictingBookings.length === 0;
  }

  async createBooking(data: {
    tenantId: string;
    guestId: string;
    bedIds: string[];
    checkIn: Date;
    checkOut: Date;
  }) {
    // Check availability first
    const isAvailable = await this.checkAvailability(
      data.bedIds,
      data.checkIn,
      data.checkOut,
    );

    if (!isAvailable) {
      throw new TRPCError({
        code: 'CONFLICT',
        message: 'Selected beds are not available for these dates',
      });
    }

    // Get bed prices
    const beds = await this.prisma.bed.findMany({
      where: { id: { in: data.bedIds } },
      include: { room: true },
    });

    // Calculate total price
    const nights = Math.ceil(
      (data.checkOut.getTime() - data.checkIn.getTime()) / (1000 * 60 * 60 * 24)
    );

    const totalPrice = beds.reduce((sum, bed) => {
      const pricePerNight = Number(bed.room.basePrice) + Number(bed.priceModifier);
      return sum + pricePerNight * nights;
    }, 0);

    // Create booking with beds
    return this.prisma.$transaction(async (tx) => {
      const booking = await tx.booking.create({
        data: {
          tenantId: data.tenantId,
          guestId: data.guestId,
          checkIn: data.checkIn,
          checkOut: data.checkOut,
          totalPrice,
          status: 'PENDING',
          bookingBeds: {
            create: beds.map((bed) => ({
              bedId: bed.id,
              pricePerNight: Number(bed.room.basePrice) + Number(bed.priceModifier),
            })),
          },
        },
        include: {
          bookingBeds: { include: { bed: true } },
          guest: true,
        },
      });

      return booking;
    });
  }

  async updateStatus(bookingId: string, status: string) {
    const updates: any = { status };

    if (status === 'CANCELLED') {
      updates.cancelledAt = new Date();
    }

    return this.prisma.booking.update({
      where: { id: bookingId },
      data: updates,
    });
  }
}
```

### 6. Availability Search (Day 3)

```typescript
// apps/api/src/modules/booking/availability.service.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '@/database/prisma.service';

interface AvailabilitySearchParams {
  city?: string;
  checkIn: Date;
  checkOut: Date;
  guests: number;
  roomType?: string;
  minPrice?: number;
  maxPrice?: number;
}

@Injectable()
export class AvailabilityService {
  constructor(private prisma: PrismaService) {}

  async searchAvailable(params: AvailabilitySearchParams) {
    const { city, checkIn, checkOut, guests, roomType, minPrice, maxPrice } = params;

    // Find properties with available beds
    const properties = await this.prisma.property.findMany({
      where: {
        isActive: true,
        deletedAt: null,
        ...(city && { city: { contains: city, mode: 'insensitive' } }),
      },
      include: {
        rooms: {
          where: {
            isActive: true,
            deletedAt: null,
            ...(roomType && { type: roomType as any }),
            ...(minPrice && { basePrice: { gte: minPrice } }),
            ...(maxPrice && { basePrice: { lte: maxPrice } }),
          },
          include: {
            beds: {
              where: { isActive: true },
            },
          },
        },
      },
    });

    // Get booked bed IDs for date range
    const bookedBeds = await this.prisma.bookingBed.findMany({
      where: {
        booking: {
          status: { notIn: ['CANCELLED', 'NO_SHOW'] },
          checkIn: { lt: checkOut },
          checkOut: { gt: checkIn },
        },
      },
      select: { bedId: true },
    });

    const bookedBedIds = new Set(bookedBeds.map((b) => b.bedId));

    // Filter to properties with enough available beds
    return properties
      .map((property) => {
        const roomsWithAvailability = property.rooms.map((room) => {
          const availableBeds = room.beds.filter(
            (bed) => !bookedBedIds.has(bed.id)
          );
          return {
            ...room,
            availableBeds,
            availableCount: availableBeds.length,
          };
        });

        const totalAvailable = roomsWithAvailability.reduce(
          (sum, r) => sum + r.availableCount,
          0
        );

        return {
          ...property,
          rooms: roomsWithAvailability,
          totalAvailable,
        };
      })
      .filter((p) => p.totalAvailable >= guests);
  }
}
```

### 7. App Router Assembly (Day 4)

```typescript
// apps/api/src/trpc/app.router.ts
import { router } from './trpc.router';
import { propertyRouter } from '@/modules/property/property.router';
import { roomRouter } from '@/modules/room/room.router';
import { bookingRouter } from '@/modules/booking/booking.router';
import { guestRouter } from '@/modules/guest/guest.router';
import { availabilityRouter } from '@/modules/booking/availability.router';
import { statsRouter } from '@/modules/stats/stats.router';

export const appRouter = router({
  property: propertyRouter,
  room: roomRouter,
  booking: bookingRouter,
  guest: guestRouter,
  availability: availabilityRouter,
  stats: statsRouter,
});

export type AppRouter = typeof appRouter;
```

### 8. tRPC Client Setup (Day 4)

```typescript
// apps/web/src/trpc/client.ts
import { createTRPCReact } from '@trpc/react-query';
import type { AppRouter } from '@repo/api/trpc';

export const trpc = createTRPCReact<AppRouter>();

// apps/web/src/trpc/provider.tsx
'use client';

import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { httpBatchLink } from '@trpc/client';
import { useState } from 'react';
import { trpc } from './client';

export function TRPCProvider({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(() => new QueryClient());
  const [trpcClient] = useState(() =>
    trpc.createClient({
      links: [
        httpBatchLink({
          url: `${process.env.NEXT_PUBLIC_API_URL}/trpc`,
          headers() {
            return {
              authorization: `Bearer ${getAccessToken()}`,
            };
          },
        }),
      ],
    })
  );

  return (
    <trpc.Provider client={trpcClient} queryClient={queryClient}>
      <QueryClientProvider client={queryClient}>
        {children}
      </QueryClientProvider>
    </trpc.Provider>
  );
}
```

### 9. Webhook Endpoints (Day 5)

```typescript
// apps/api/src/modules/webhooks/webhooks.controller.ts
import { Controller, Post, Body, Headers, RawBody } from '@nestjs/common';
import { Public } from '@/modules/auth/decorators/public.decorator';
import { PaymentService } from '@/modules/payment/payment.service';

@Controller('webhooks')
export class WebhooksController {
  constructor(private paymentService: PaymentService) {}

  @Public()
  @Post('sepay')
  async handleSepayWebhook(
    @Body() payload: any,
    @Headers('x-sepay-signature') signature: string,
  ) {
    await this.paymentService.handleSepayWebhook(payload, signature);
    return { received: true };
  }

  @Public()
  @Post('vnpay')
  async handleVnpayWebhook(@Body() payload: any) {
    await this.paymentService.handleVnpayWebhook(payload);
    return { received: true };
  }
}
```

### 10. Shared Schemas (Day 5)

```typescript
// packages/shared/src/schemas/property.schema.ts
import { z } from 'zod';

export const propertyCreateSchema = z.object({
  name: z.string().min(2).max(100),
  description: z.string().optional(),
  address: z.string().min(5),
  city: z.string().min(2),
  district: z.string().optional(),
  latitude: z.number().optional(),
  longitude: z.number().optional(),
  amenities: z.array(z.string()).default([]),
  images: z.array(z.string().url()).default([]),
});

export const propertyUpdateSchema = propertyCreateSchema
  .partial()
  .extend({ id: z.string().uuid() });

// packages/shared/src/schemas/booking.schema.ts
export const bookingCreateSchema = z.object({
  guestId: z.string().uuid(),
  bedIds: z.array(z.string().uuid()).min(1),
  checkIn: z.coerce.date(),
  checkOut: z.coerce.date(),
  notes: z.string().optional(),
}).refine(
  (data) => data.checkOut > data.checkIn,
  { message: 'Check-out must be after check-in' }
);

export const availabilitySearchSchema = z.object({
  city: z.string().optional(),
  checkIn: z.coerce.date(),
  checkOut: z.coerce.date(),
  guests: z.number().int().min(1).default(1),
  roomType: z.string().optional(),
  minPrice: z.number().optional(),
  maxPrice: z.number().optional(),
});
```

## Todo List

- [ ] Install tRPC dependencies (api + web)
- [ ] Create tRPC context with user/prisma
- [ ] Define base procedures (public, protected, owner)
- [ ] Implement property router + service
- [ ] Implement room router + service
- [ ] Implement booking router + service
- [ ] Implement availability service
- [ ] Implement guest router + service
- [ ] Implement stats router
- [ ] Create app router assembly
- [ ] Setup tRPC client in Next.js
- [ ] Create webhook controller (REST)
- [ ] Define Zod schemas in shared package
- [ ] Write API tests

## Success Criteria

- [ ] All tRPC endpoints respond with correct data
- [ ] Type safety works end-to-end (no TypeScript errors)
- [ ] Availability search returns correct results
- [ ] Booking creates with correct price calculation
- [ ] Webhook endpoints receive and process payloads
- [ ] Tenant isolation enforced on all queries

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Availability race conditions | High | Use database transactions |
| N+1 query performance | Medium | Use Prisma includes, DataLoader |
| Type drift between packages | Medium | Turborepo type checking |

## Security Considerations

- All mutations require authentication
- Owner/Manager role required for destructive operations
- Input validation on all endpoints
- Webhook signature verification

## Next Steps

After completion, proceed to [Phase 05: Owner Dashboard](./phase-05-owner-dashboard-property-booking-reports-ui.md)
