# HostelViet - Code Standards and Codebase Structure

**Version:** 1.0
**Last Updated:** 2026-02-03

## Overview

This document defines coding conventions, architectural patterns, and best practices for the HostelViet codebase. All contributors must follow these standards to maintain code quality and consistency.

## Core Principles

### YAGNI (You Aren't Gonna Need It)
- Implement features only when required
- Avoid speculative generalization
- Delete unused code immediately

### KISS (Keep It Simple, Stupid)
- Prefer simple solutions over complex ones
- Maximum function length: 50 lines
- Maximum file length: 200 lines
- Split large files into focused modules

### DRY (Don't Repeat Yourself)
- Extract reusable logic into utilities
- Use composition over inheritance
- Shared code lives in `packages/` workspace

## Project Structure

### Monorepo Organization

```
swdv2/
├── apps/
│   ├── web/                    # Next.js guest booking portal
│   │   ├── app/                # App Router pages
│   │   ├── components/         # Page-specific components
│   │   ├── lib/                # App-specific utilities
│   │   └── public/             # Static assets
│   ├── dashboard/              # Next.js owner dashboard
│   │   ├── app/
│   │   ├── components/
│   │   ├── lib/
│   │   └── public/
│   └── api/                    # NestJS backend
│       ├── src/
│       │   ├── auth/           # Authentication module
│       │   ├── booking/        # Booking domain
│       │   ├── property/       # Property domain
│       │   ├── payment/        # Payment integration
│       │   ├── subscription/   # Subscription management
│       │   ├── user/           # User management
│       │   ├── common/         # Shared utilities
│       │   │   ├── guards/     # Auth guards
│       │   │   ├── decorators/ # Custom decorators
│       │   │   ├── filters/    # Exception filters
│       │   │   └── interceptors/ # Request/response interceptors
│       │   ├── config/         # Configuration modules
│       │   └── main.ts         # Application entry
│       └── test/               # E2E tests
├── packages/
│   ├── ui/                     # Shared UI components
│   │   ├── src/
│   │   │   ├── button.tsx
│   │   │   ├── card.tsx
│   │   │   ├── form.tsx
│   │   │   └── index.ts        # Barrel exports
│   │   └── package.json
│   ├── config/                 # Shared configs
│   │   ├── eslint/
│   │   ├── typescript/
│   │   └── tailwind/
│   ├── database/               # Prisma schema
│   │   ├── prisma/
│   │   │   ├── schema.prisma
│   │   │   ├── migrations/
│   │   │   └── seed.ts
│   │   └── src/
│   │       ├── client.ts       # Prisma client instance
│   │       └── types.ts        # Generated types
│   └── types/                  # Shared TypeScript types
│       ├── src/
│       │   ├── api/            # API response types
│       │   ├── models/         # Domain models
│       │   └── common/         # Utility types
│       └── package.json
├── docs/                       # Documentation
├── plans/                      # Implementation plans
├── .github/
│   └── workflows/              # CI/CD pipelines
├── turbo.json                  # Turborepo config
├── package.json                # Root package.json
└── pnpm-workspace.yaml         # Workspace configuration
```

## Naming Conventions

### Files and Directories

**Rule:** Use kebab-case with descriptive names

```
✅ GOOD
user-profile-form.tsx
booking-calendar-view.tsx
payment-webhook-handler.ts
database-connection-pool.ts

❌ BAD
UserProfile.tsx (PascalCase)
bookingcal.tsx (abbreviated)
payment.ts (too generic)
utils.ts (meaningless)
```

**Why:** When using tools like Grep or Glob, developers can understand file purpose without opening it.

### Components and Classes

**Rule:** PascalCase for React components and TypeScript classes

```typescript
// ✅ GOOD
export function UserProfileForm() {}
export class BookingService {}
export interface PropertyDetails {}

// ❌ BAD
export function userProfileForm() {}
export class bookingService {}
export interface propertydetails {}
```

### Variables and Functions

**Rule:** camelCase for variables, functions, and methods

```typescript
// ✅ GOOD
const userId = 123;
function calculateTotalPrice() {}
const isAvailable = true;

// ❌ BAD
const UserId = 123;
const user_id = 123;
function CalculateTotalPrice() {}
```

### Constants

**Rule:** SCREAMING_SNAKE_CASE for true constants

```typescript
// ✅ GOOD
const MAX_UPLOAD_SIZE = 5 * 1024 * 1024; // 5MB
const DEFAULT_PAGE_SIZE = 20;
const JWT_EXPIRY_SECONDS = 900; // 15 minutes

// ❌ BAD
const maxUploadSize = 5 * 1024 * 1024;
const PageSize = 20;
```

### Database Entities

**Rule:** PascalCase singular for Prisma models

```prisma
// ✅ GOOD
model User {}
model Property {}
model Booking {}

// ❌ BAD
model users {}
model Properties {}
model booking_table {}
```

## TypeScript Standards

### Strict Mode Configuration

```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "forceConsistentCasingInFileNames": true
  }
}
```

### Type Definitions

**Rule:** Prefer interfaces for object shapes, types for unions/intersections

```typescript
// ✅ GOOD - Interface for object shapes
interface UserProfile {
  id: string;
  email: string;
  name: string;
}

// ✅ GOOD - Type for unions
type UserRole = 'owner' | 'manager' | 'receptionist' | 'guest';
type PaymentStatus = 'pending' | 'completed' | 'failed';

// ✅ GOOD - Type for complex combinations
type AuthenticatedUser = UserProfile & {
  role: UserRole;
  tenantId: string;
};

// ❌ BAD - Type for simple objects (use interface)
type UserProfile = {
  id: string;
  email: string;
};
```

### Avoid `any` Type

```typescript
// ✅ GOOD
function processBooking(data: BookingInput): BookingResult {
  return { success: true, bookingId: data.id };
}

// ❌ BAD
function processBooking(data: any): any {
  return data;
}

// ✅ ACCEPTABLE - When type is truly unknown
function parseJson(json: string): unknown {
  return JSON.parse(json);
}
```

### Null Safety

```typescript
// ✅ GOOD - Explicit null handling
function getUserEmail(userId: string): string | null {
  const user = findUser(userId);
  return user?.email ?? null;
}

// ✅ GOOD - Optional chaining
const propertyName = booking?.property?.name;

// ❌ BAD - Non-null assertion (avoid unless absolutely necessary)
const email = user!.email;
```

## React and Next.js Standards

### Component Structure

**Rule:** Functional components with TypeScript

```typescript
// ✅ GOOD - Typed props with interface
interface BookingCardProps {
  bookingId: string;
  propertyName: string;
  checkIn: Date;
  checkOut: Date;
  totalPrice: number;
  onCancel?: () => void;
}

export function BookingCard({
  bookingId,
  propertyName,
  checkIn,
  checkOut,
  totalPrice,
  onCancel,
}: BookingCardProps) {
  return (
    <div className="rounded-lg border p-4">
      <h3>{propertyName}</h3>
      <p>{formatDateRange(checkIn, checkOut)}</p>
      <p>${totalPrice}</p>
      {onCancel && <button onClick={onCancel}>Cancel</button>}
    </div>
  );
}

// ❌ BAD - Class components, inline types
export class BookingCard extends React.Component<{
  bookingId: string;
  propertyName: string;
}> {}
```

### File Organization

```typescript
// booking-card.tsx

// 1. Imports - external first, then internal
import { useState } from 'react';
import { formatDateRange } from '@/lib/utils';
import { Button } from '@/components/ui/button';

// 2. Type definitions
interface BookingCardProps {
  // ...
}

// 3. Component
export function BookingCard(props: BookingCardProps) {
  // State hooks
  const [isLoading, setIsLoading] = useState(false);

  // Event handlers
  const handleCancel = async () => {
    setIsLoading(true);
    await cancelBooking(props.bookingId);
    setIsLoading(false);
  };

  // Render
  return (/* JSX */);
}

// 4. Helper functions (if only used in this file)
function formatPrice(amount: number): string {
  return `$${amount.toFixed(2)}`;
}
```

### Server vs Client Components

```typescript
// ✅ GOOD - Server component (default in Next.js 15)
// app/properties/page.tsx
import { db } from '@/lib/database';

export default async function PropertiesPage() {
  const properties = await db.property.findMany();
  return <PropertyList properties={properties} />;
}

// ✅ GOOD - Client component (interactive)
// components/property-search-form.tsx
'use client';

import { useState } from 'react';

export function PropertySearchForm() {
  const [query, setQuery] = useState('');
  return <input value={query} onChange={(e) => setQuery(e.target.value)} />;
}
```

### Hooks Best Practices

```typescript
// ✅ GOOD - Custom hook with clear naming
export function useBookingForm(initialData?: BookingInput) {
  const [formData, setFormData] = useState(initialData ?? {});
  const [errors, setErrors] = useState<Record<string, string>>({});

  const validate = () => {
    // Validation logic
  };

  return { formData, errors, validate, setFormData };
}

// ✅ GOOD - Cleanup in useEffect
useEffect(() => {
  const subscription = subscribeToBookings(propertyId);
  return () => subscription.unsubscribe();
}, [propertyId]);

// ❌ BAD - Missing dependency array
useEffect(() => {
  fetchData();
}); // Runs on every render
```

## NestJS Backend Standards

### Module Structure

```typescript
// booking/booking.module.ts
import { Module } from '@nestjs/common';
import { BookingController } from './booking.controller';
import { BookingService } from './booking.service';
import { BookingRepository } from './booking.repository';

@Module({
  controllers: [BookingController],
  providers: [BookingService, BookingRepository],
  exports: [BookingService], // Export if used by other modules
})
export class BookingModule {}
```

### Service Pattern

```typescript
// booking/booking.service.ts
import { Injectable, NotFoundException } from '@nestjs/common';
import { BookingRepository } from './booking.repository';
import { CreateBookingDto } from './dto/create-booking.dto';

@Injectable()
export class BookingService {
  constructor(private readonly bookingRepo: BookingRepository) {}

  async createBooking(dto: CreateBookingDto, userId: string) {
    // Validate availability
    const isAvailable = await this.checkAvailability(dto);
    if (!isAvailable) {
      throw new ConflictException('Room not available');
    }

    // Create booking
    return this.bookingRepo.create({
      ...dto,
      userId,
      status: 'pending',
    });
  }

  async getBooking(id: string, tenantId: string) {
    const booking = await this.bookingRepo.findOne(id, tenantId);
    if (!booking) {
      throw new NotFoundException(`Booking ${id} not found`);
    }
    return booking;
  }

  private async checkAvailability(dto: CreateBookingDto): Promise<boolean> {
    // Implementation
    return true;
  }
}
```

### DTO Validation

```typescript
// booking/dto/create-booking.dto.ts
import { IsString, IsDate, IsNumber, Min } from 'class-validator';
import { Type } from 'class-transformer';

export class CreateBookingDto {
  @IsString()
  propertyId: string;

  @IsString()
  roomId: string;

  @Type(() => Date)
  @IsDate()
  checkIn: Date;

  @Type(() => Date)
  @IsDate()
  checkOut: Date;

  @IsNumber()
  @Min(1)
  guestCount: number;
}
```

### Repository Pattern

```typescript
// booking/booking.repository.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '@/database/prisma.service';

@Injectable()
export class BookingRepository {
  constructor(private readonly prisma: PrismaService) {}

  async create(data: CreateBookingData) {
    return this.prisma.booking.create({
      data,
      include: {
        property: true,
        room: true,
      },
    });
  }

  async findOne(id: string, tenantId: string) {
    return this.prisma.booking.findFirst({
      where: { id, property: { tenantId } }, // RLS enforcement
      include: { property: true, room: true },
    });
  }

  async findMany(filters: BookingFilters, tenantId: string) {
    return this.prisma.booking.findMany({
      where: {
        property: { tenantId },
        ...filters,
      },
      orderBy: { createdAt: 'desc' },
    });
  }
}
```

## Error Handling

### Frontend Error Handling

```typescript
// ✅ GOOD - Try/catch with user feedback
async function handleBookingSubmit() {
  try {
    setIsLoading(true);
    const result = await trpc.booking.create.mutate(formData);
    toast.success('Booking created successfully!');
    router.push(`/bookings/${result.id}`);
  } catch (error) {
    if (error instanceof TRPCError) {
      toast.error(error.message);
    } else {
      toast.error('An unexpected error occurred');
      console.error('Booking error:', error);
    }
  } finally {
    setIsLoading(false);
  }
}

// ❌ BAD - Unhandled errors
async function handleBookingSubmit() {
  const result = await trpc.booking.create.mutate(formData); // No error handling
  router.push(`/bookings/${result.id}`);
}
```

### Backend Error Handling

```typescript
// ✅ GOOD - Specific exception types
import { NotFoundException, ConflictException, BadRequestException } from '@nestjs/common';

async checkAvailability(propertyId: string, dates: DateRange) {
  const property = await this.propertyRepo.findOne(propertyId);
  if (!property) {
    throw new NotFoundException('Property not found');
  }

  if (!this.isValidDateRange(dates)) {
    throw new BadRequestException('Invalid date range');
  }

  const hasConflict = await this.hasBookingConflict(propertyId, dates);
  if (hasConflict) {
    throw new ConflictException('Room already booked for these dates');
  }
}

// ✅ GOOD - Global exception filter
@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();

    const status = exception instanceof HttpException
      ? exception.getStatus()
      : 500;

    const message = exception instanceof HttpException
      ? exception.message
      : 'Internal server error';

    logger.error('Exception caught', { exception, request });

    response.status(status).json({
      statusCode: status,
      message,
      timestamp: new Date().toISOString(),
      path: request.url,
    });
  }
}
```

## Database and Prisma Standards

### Schema Organization

```prisma
// schema.prisma

// Core entities
model Tenant {
  id        String   @id @default(cuid())
  name      String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  properties Property[]
  users      User[]
  subscription Subscription?

  @@map("tenants")
}

model Property {
  id          String   @id @default(cuid())
  tenantId    String
  name        String
  address     String
  city        String
  description String?
  amenities   String[] // JSON array
  photos      String[] // S3 URLs
  rating      Float    @default(0)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  tenant   Tenant    @relation(fields: [tenantId], references: [id], onDelete: Cascade)
  rooms    Room[]
  bookings Booking[]

  @@index([tenantId])
  @@index([city])
  @@map("properties")
}

// Use singular names, clear relationships, proper indexes
```

### Query Optimization

```typescript
// ✅ GOOD - Selective field loading
const properties = await prisma.property.findMany({
  select: {
    id: true,
    name: true,
    city: true,
    rating: true,
    photos: true,
  },
  where: { city: 'Hanoi' },
  take: 20,
});

// ✅ GOOD - Batched queries
const [properties, totalCount] = await Promise.all([
  prisma.property.findMany({ where, take: 20, skip: page * 20 }),
  prisma.property.count({ where }),
]);

// ❌ BAD - N+1 queries
const properties = await prisma.property.findMany();
for (const property of properties) {
  const rooms = await prisma.room.findMany({ where: { propertyId: property.id } });
}

// ✅ GOOD - Use include instead
const properties = await prisma.property.findMany({
  include: { rooms: true },
});
```

## Testing Standards

### Unit Tests

```typescript
// booking.service.spec.ts
import { Test } from '@nestjs/testing';
import { BookingService } from './booking.service';
import { BookingRepository } from './booking.repository';

describe('BookingService', () => {
  let service: BookingService;
  let repository: BookingRepository;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        BookingService,
        {
          provide: BookingRepository,
          useValue: {
            create: jest.fn(),
            findOne: jest.fn(),
          },
        },
      ],
    }).compile();

    service = module.get(BookingService);
    repository = module.get(BookingRepository);
  });

  describe('createBooking', () => {
    it('should create booking when room is available', async () => {
      const dto = {
        propertyId: 'prop-1',
        roomId: 'room-1',
        checkIn: new Date('2026-03-01'),
        checkOut: new Date('2026-03-05'),
        guestCount: 2,
      };

      jest.spyOn(repository, 'create').mockResolvedValue(mockBooking);

      const result = await service.createBooking(dto, 'user-1');

      expect(result).toEqual(mockBooking);
      expect(repository.create).toHaveBeenCalledWith(
        expect.objectContaining({ userId: 'user-1', status: 'pending' })
      );
    });

    it('should throw ConflictException when room is unavailable', async () => {
      // Test implementation
    });
  });
});
```

### Coverage Requirements

- **Overall Coverage:** 80% minimum
- **Critical Paths:** 95% (payment, booking, auth)
- **UI Components:** 70% (focus on business logic)

## Security Standards

### Authentication

```typescript
// ✅ GOOD - JWT guard on sensitive routes
@UseGuards(JwtAuthGuard)
@Get('bookings')
async getBookings(@CurrentUser() user: AuthUser) {
  return this.bookingService.findByUser(user.id, user.tenantId);
}

// ✅ GOOD - Role-based authorization
@UseGuards(JwtAuthGuard, RolesGuard)
@Roles('owner', 'manager')
@Post('properties')
async createProperty(@Body() dto: CreatePropertyDto, @CurrentUser() user: AuthUser) {
  return this.propertyService.create(dto, user.tenantId);
}
```

### Input Validation

```typescript
// ✅ GOOD - Validate and sanitize inputs
import { IsEmail, IsString, MinLength, Matches } from 'class-validator';

export class RegisterDto {
  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/, {
    message: 'Password must contain uppercase, lowercase, and number',
  })
  password: string;
}

// ✅ GOOD - SQL injection prevention (Prisma handles this)
const users = await prisma.user.findMany({
  where: { email: userInput }, // Parameterized query, safe
});
```

### Multi-Tenancy Enforcement

```typescript
// ✅ GOOD - Always filter by tenantId
async getProperty(id: string, tenantId: string) {
  return this.prisma.property.findFirst({
    where: { id, tenantId }, // Prevents cross-tenant access
  });
}

// ❌ BAD - Missing tenant isolation
async getProperty(id: string) {
  return this.prisma.property.findUnique({ where: { id } }); // Security risk!
}
```

## Performance Standards

### Response Time Targets
- **API Endpoints:** p95 <200ms, p99 <500ms
- **Database Queries:** p95 <100ms
- **Page Loads:** <3 seconds on 3G

### Optimization Techniques

```typescript
// ✅ GOOD - Pagination
async getBookings(page: number = 1, pageSize: number = 20) {
  return prisma.booking.findMany({
    take: pageSize,
    skip: (page - 1) * pageSize,
  });
}

// ✅ GOOD - Caching with Redis
async getCachedProperty(id: string) {
  const cached = await redis.get(`property:${id}`);
  if (cached) return JSON.parse(cached);

  const property = await prisma.property.findUnique({ where: { id } });
  await redis.setex(`property:${id}`, 3600, JSON.stringify(property));
  return property;
}

// ✅ GOOD - Debounce search inputs
const debouncedSearch = useDebouncedCallback((query: string) => {
  searchProperties(query);
}, 300);
```

## Code Review Checklist

Before submitting PR, verify:

- [ ] No syntax errors, code compiles
- [ ] All tests pass (unit, integration)
- [ ] No ESLint warnings or errors
- [ ] TypeScript strict mode compliance
- [ ] File naming follows kebab-case convention
- [ ] Components under 200 lines (split if larger)
- [ ] Error handling implemented
- [ ] Multi-tenancy enforced (where applicable)
- [ ] Security considerations addressed
- [ ] Performance optimized (queries indexed, cached)
- [ ] Documentation updated
- [ ] Commit messages follow conventional commits

---

**Document Owner:** Technical Lead
**Last Review:** 2026-02-03
**Next Review:** 2026-03-03
