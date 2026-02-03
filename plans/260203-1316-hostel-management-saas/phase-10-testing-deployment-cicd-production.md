---
title: "Phase 10: Testing & Deployment"
status: pending
priority: P1
effort: 5d
---

# Phase 10: Testing & Deployment - CI/CD, Production

## Context Links

- [Phase 09: Monitoring & Logging](./phase-09-monitoring-logging-elk-prometheus-grafana.md)
- [Plan Overview](./plan.md)

## Overview

Implement comprehensive testing strategy (unit, integration, e2e) and production deployment pipeline. Configure CI/CD with GitHub Actions, containerize with Docker, deploy to cloud infrastructure.

## Key Insights

- Test pyramid: many unit tests, fewer integration, minimal e2e
- CI must complete <10 minutes for developer productivity
- Blue-green deployment for zero downtime
- Infrastructure as Code (Docker Compose or Kubernetes)

## Requirements

### Functional
- Unit tests for services and utils
- Integration tests for API endpoints
- E2E tests for critical user flows
- Database seeding for tests
- CI pipeline with all checks
- Staging environment
- Production deployment
- Rollback capability

### Non-Functional
- Test coverage >80%
- CI pipeline <10 minutes
- Zero-downtime deployments
- <5 minute rollback
- 99.9% uptime target

## Architecture

### Test Strategy

```
E2E Tests (Playwright)
├── Guest booking flow
├── Owner property management
└── Payment flow

Integration Tests (Supertest)
├── Auth endpoints
├── Booking API
├── Payment webhooks
└── tRPC routers

Unit Tests (Vitest)
├── Services
├── Utils
├── Validators
└── Business logic
```

### Deployment Architecture

```
GitHub Actions
    │
    ├── Build & Test
    │       │
    │       └── Docker Images
    │               │
    ├── Staging Deploy ──────> Staging Environment
    │                              │
    │                          Manual Approval
    │                              │
    └── Production Deploy ────> Production Environment
                                   │
                              ┌────┴────┐
                              │         │
                           App 1     App 2 (Blue-Green)
                              │         │
                              └────┬────┘
                                   │
                              Load Balancer
```

### Infrastructure

```
Cloud Provider (Railway/Render/AWS)
├── API Service (NestJS)
│   ├── Replicas: 2+
│   └── Auto-scaling
├── Web Service (Next.js)
│   ├── Vercel or container
│   └── Edge caching
├── PostgreSQL (Managed)
│   ├── Primary + Read replica
│   └── Daily backups
├── Redis (Managed)
│   └── Cluster mode
├── RabbitMQ (Managed)
│   └── CloudAMQP
└── Object Storage (S3/Cloudinary)
```

## Related Code Files

### Create
- `apps/api/test/setup.ts`
- `apps/api/test/helpers/*.ts`
- `apps/api/src/**/*.spec.ts`
- `apps/api/test/integration/*.test.ts`
- `apps/web/e2e/*.spec.ts`
- `apps/web/playwright.config.ts`
- `.github/workflows/ci.yml`
- `.github/workflows/deploy-staging.yml`
- `.github/workflows/deploy-production.yml`
- `Dockerfile.api`
- `Dockerfile.web`
- `docker-compose.prod.yml`
- `infra/terraform/*.tf` (optional)

## Implementation Steps

### 1. Test Setup (Day 1)

```typescript
// apps/api/test/setup.ts
import { PrismaClient } from '@prisma/client';
import { execSync } from 'child_process';

const prisma = new PrismaClient();

beforeAll(async () => {
  // Create test database
  process.env.DATABASE_URL = process.env.TEST_DATABASE_URL;
  execSync('pnpm prisma migrate deploy', { stdio: 'inherit' });
});

beforeEach(async () => {
  // Clean database between tests
  const tables = await prisma.$queryRaw<{ tablename: string }[]>`
    SELECT tablename FROM pg_tables WHERE schemaname = 'public'
  `;

  for (const { tablename } of tables) {
    if (tablename !== '_prisma_migrations') {
      await prisma.$executeRawUnsafe(`TRUNCATE TABLE "${tablename}" CASCADE`);
    }
  }
});

afterAll(async () => {
  await prisma.$disconnect();
});

// apps/api/test/helpers/factories.ts
import { faker } from '@faker-js/faker';
import { PrismaClient } from '@prisma/client';
import { hash } from 'bcrypt';

export class TestFactory {
  constructor(private prisma: PrismaClient) {}

  async createTenant(overrides = {}) {
    return this.prisma.tenant.create({
      data: {
        name: faker.company.name(),
        slug: faker.helpers.slugify(faker.company.name()).toLowerCase(),
        email: faker.internet.email(),
        ...overrides,
      },
    });
  }

  async createUser(tenantId: string, overrides = {}) {
    return this.prisma.user.create({
      data: {
        tenantId,
        email: faker.internet.email(),
        password: await hash('password123', 10),
        name: faker.person.fullName(),
        role: 'STAFF',
        ...overrides,
      },
    });
  }

  async createProperty(tenantId: string, overrides = {}) {
    return this.prisma.property.create({
      data: {
        tenantId,
        name: faker.company.name() + ' Hostel',
        address: faker.location.streetAddress(),
        city: 'Ho Chi Minh City',
        ...overrides,
      },
    });
  }

  async createBooking(tenantId: string, guestId: string, bedIds: string[], overrides = {}) {
    return this.prisma.booking.create({
      data: {
        tenantId,
        guestId,
        checkIn: new Date(),
        checkOut: new Date(Date.now() + 86400000),
        totalPrice: 500000,
        status: 'PENDING',
        bookingBeds: {
          create: bedIds.map((bedId) => ({
            bedId,
            pricePerNight: 150000,
          })),
        },
        ...overrides,
      },
    });
  }
}
```

### 2. Unit Tests (Day 1-2)

```typescript
// apps/api/src/modules/booking/booking.service.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { BookingService } from './booking.service';
import { PrismaService } from '@/database/prisma.service';
import { TRPCError } from '@trpc/server';

describe('BookingService', () => {
  let service: BookingService;
  let prisma: PrismaService;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        BookingService,
        {
          provide: PrismaService,
          useValue: {
            booking: {
              create: jest.fn(),
              findMany: jest.fn(),
              update: jest.fn(),
            },
            bookingBed: {
              findMany: jest.fn(),
            },
            bed: {
              findMany: jest.fn(),
            },
            $transaction: jest.fn((fn) => fn(prisma)),
          },
        },
      ],
    }).compile();

    service = module.get<BookingService>(BookingService);
    prisma = module.get<PrismaService>(PrismaService);
  });

  describe('checkAvailability', () => {
    it('should return true when no conflicting bookings', async () => {
      jest.spyOn(prisma.bookingBed, 'findMany').mockResolvedValue([]);

      const result = await service.checkAvailability(
        ['bed-1', 'bed-2'],
        new Date('2024-01-01'),
        new Date('2024-01-03')
      );

      expect(result).toBe(true);
    });

    it('should return false when beds are booked', async () => {
      jest.spyOn(prisma.bookingBed, 'findMany').mockResolvedValue([
        { id: 'bb-1', bedId: 'bed-1', bookingId: 'booking-1' } as any,
      ]);

      const result = await service.checkAvailability(
        ['bed-1', 'bed-2'],
        new Date('2024-01-01'),
        new Date('2024-01-03')
      );

      expect(result).toBe(false);
    });
  });

  describe('createBooking', () => {
    it('should throw CONFLICT when beds not available', async () => {
      jest.spyOn(service, 'checkAvailability').mockResolvedValue(false);

      await expect(
        service.createBooking({
          tenantId: 'tenant-1',
          guestId: 'guest-1',
          bedIds: ['bed-1'],
          checkIn: new Date(),
          checkOut: new Date(),
        })
      ).rejects.toThrow(TRPCError);
    });

    it('should calculate correct total price', async () => {
      jest.spyOn(service, 'checkAvailability').mockResolvedValue(true);
      jest.spyOn(prisma.bed, 'findMany').mockResolvedValue([
        { id: 'bed-1', room: { basePrice: 150000 }, priceModifier: 0 } as any,
      ]);

      const mockBooking = { id: 'booking-1', totalPrice: 300000 };
      jest.spyOn(prisma.booking, 'create').mockResolvedValue(mockBooking as any);

      const result = await service.createBooking({
        tenantId: 'tenant-1',
        guestId: 'guest-1',
        bedIds: ['bed-1'],
        checkIn: new Date('2024-01-01'),
        checkOut: new Date('2024-01-03'), // 2 nights
      });

      expect(result.totalPrice).toBe(300000);
    });
  });
});
```

### 3. Integration Tests (Day 2)

```typescript
// apps/api/test/integration/booking.test.ts
import { Test, TestingModule } from '@nestjs/testing';
import { INestApplication } from '@nestjs/common';
import * as request from 'supertest';
import { AppModule } from '@/app.module';
import { PrismaService } from '@/database/prisma.service';
import { TestFactory } from '../helpers/factories';

describe('Booking API (Integration)', () => {
  let app: INestApplication;
  let prisma: PrismaService;
  let factory: TestFactory;
  let authToken: string;
  let tenant: any;
  let property: any;
  let room: any;
  let bed: any;
  let guest: any;

  beforeAll(async () => {
    const module: TestingModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = module.createNestApplication();
    prisma = module.get<PrismaService>(PrismaService);
    factory = new TestFactory(prisma);
    await app.init();
  });

  beforeEach(async () => {
    // Setup test data
    tenant = await factory.createTenant();
    const user = await factory.createUser(tenant.id, { role: 'OWNER' });
    property = await factory.createProperty(tenant.id);
    room = await prisma.room.create({
      data: {
        propertyId: property.id,
        name: 'Test Room',
        type: 'DORM_MIXED',
        capacity: 4,
        basePrice: 150000,
      },
    });
    bed = await prisma.bed.create({
      data: { roomId: room.id, name: 'Bed 1', type: 'SINGLE' },
    });
    guest = await prisma.guest.create({
      data: { email: 'guest@test.com', name: 'Test Guest' },
    });

    // Get auth token
    const loginRes = await request(app.getHttpServer())
      .post('/auth/staff/login')
      .send({ email: user.email, password: 'password123' });
    authToken = loginRes.body.accessToken;
  });

  afterAll(async () => {
    await app.close();
  });

  describe('POST /trpc/booking.create', () => {
    it('should create a booking successfully', async () => {
      const response = await request(app.getHttpServer())
        .post('/trpc/booking.create')
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          guestId: guest.id,
          bedIds: [bed.id],
          checkIn: '2024-06-01',
          checkOut: '2024-06-03',
        });

      expect(response.status).toBe(200);
      expect(response.body.result.data).toHaveProperty('id');
      expect(response.body.result.data.status).toBe('PENDING');
      expect(response.body.result.data.totalPrice).toBe(300000); // 2 nights
    });

    it('should reject booking for unavailable dates', async () => {
      // Create existing booking
      await factory.createBooking(tenant.id, guest.id, [bed.id], {
        checkIn: new Date('2024-06-01'),
        checkOut: new Date('2024-06-03'),
        status: 'CONFIRMED',
      });

      const response = await request(app.getHttpServer())
        .post('/trpc/booking.create')
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          guestId: guest.id,
          bedIds: [bed.id],
          checkIn: '2024-06-02',
          checkOut: '2024-06-04',
        });

      expect(response.status).toBe(409);
    });
  });

  describe('PATCH /trpc/booking.updateStatus', () => {
    it('should update booking status to CONFIRMED', async () => {
      const booking = await factory.createBooking(tenant.id, guest.id, [bed.id]);

      const response = await request(app.getHttpServer())
        .post('/trpc/booking.updateStatus')
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          id: booking.id,
          status: 'CONFIRMED',
        });

      expect(response.status).toBe(200);
      expect(response.body.result.data.status).toBe('CONFIRMED');
    });
  });
});
```

### 4. E2E Tests (Day 3)

```typescript
// apps/web/e2e/booking-flow.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Guest Booking Flow', () => {
  test.beforeEach(async ({ page }) => {
    // Seed test data via API
    await page.request.post('/api/test/seed');
  });

  test('complete booking flow', async ({ page }) => {
    // 1. Visit homepage
    await page.goto('/');
    await expect(page.getByRole('heading', { name: /find your perfect hostel/i })).toBeVisible();

    // 2. Search for hostels
    await page.getByPlaceholder('Where are you going?').fill('Ho Chi Minh');
    await page.getByRole('button', { name: /search/i }).click();

    // 3. View search results
    await expect(page).toHaveURL(/\/search/);
    await expect(page.getByTestId('property-card')).toHaveCount.greaterThan(0);

    // 4. Click on a property
    await page.getByTestId('property-card').first().click();
    await expect(page).toHaveURL(/\/hostels\//);

    // 5. Start booking
    await page.getByRole('button', { name: /book now/i }).click();
    await expect(page).toHaveURL(/\/book/);

    // 6. Select beds
    await page.getByTestId('bed-button').first().click();
    await page.getByRole('button', { name: /continue/i }).click();

    // 7. Fill guest info
    await page.getByLabel('Full Name').fill('Test Guest');
    await page.getByLabel('Email').fill('test@example.com');
    await page.getByLabel('Phone').fill('0901234567');
    await page.getByRole('button', { name: /continue/i }).click();

    // 8. Confirm payment
    await expect(page.getByText(/scan to pay/i)).toBeVisible();
    await expect(page.getByTestId('qr-code')).toBeVisible();

    // 9. Simulate payment webhook
    await page.request.post('/api/test/simulate-payment');

    // 10. Verify confirmation
    await expect(page.getByText(/booking confirmed/i)).toBeVisible({ timeout: 30000 });
  });

  test('owner check-in flow', async ({ page }) => {
    // Login as owner
    await page.goto('/login');
    await page.getByLabel('Email').fill('owner@test.com');
    await page.getByLabel('Password').fill('password123');
    await page.getByRole('button', { name: /sign in/i }).click();

    // Navigate to bookings
    await page.goto('/dashboard/bookings');
    await expect(page.getByTestId('booking-row')).toHaveCount.greaterThan(0);

    // Check in guest
    await page.getByTestId('booking-row').first().getByRole('button', { name: /check in/i }).click();
    await expect(page.getByText(/checked in/i)).toBeVisible();
  });
});
```

### 5. CI Pipeline (Day 4)

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '20'
  PNPM_VERSION: '8'

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      - run: pnpm install --frozen-lockfile
      - run: pnpm turbo lint

  typecheck:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      - run: pnpm install --frozen-lockfile
      - run: pnpm turbo typecheck

  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
      redis:
        image: redis:7
        ports:
          - 6379:6379
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      - run: pnpm install --frozen-lockfile
      - run: pnpm turbo test -- --coverage
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/test
          REDIS_URL: redis://localhost:6379
      - uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info

  build:
    runs-on: ubuntu-latest
    needs: [lint, typecheck, test]
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      - run: pnpm install --frozen-lockfile
      - run: pnpm turbo build
      - uses: actions/upload-artifact@v3
        with:
          name: build
          path: |
            apps/api/dist
            apps/web/.next

  e2e:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      - run: pnpm install --frozen-lockfile
      - run: npx playwright install --with-deps
      - uses: actions/download-artifact@v3
        with:
          name: build
      - run: pnpm e2e
        env:
          BASE_URL: http://localhost:3000
      - uses: actions/upload-artifact@v3
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/
```

### 6. Dockerfile (Day 4)

```dockerfile
# Dockerfile.api
FROM node:20-alpine AS base
RUN npm install -g pnpm

FROM base AS deps
WORKDIR /app
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml ./
COPY apps/api/package.json ./apps/api/
COPY packages/database/package.json ./packages/database/
COPY packages/shared/package.json ./packages/shared/
RUN pnpm install --frozen-lockfile --prod

FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN pnpm turbo build --filter=api

FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nestjs

COPY --from=builder --chown=nestjs:nodejs /app/apps/api/dist ./dist
COPY --from=builder --chown=nestjs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nestjs:nodejs /app/packages/database/prisma ./prisma

USER nestjs

EXPOSE 3001
CMD ["node", "dist/main.js"]
```

### 7. Production Deployment (Day 5)

```yaml
# .github/workflows/deploy-production.yml
name: Deploy Production

on:
  push:
    tags:
      - 'v*'

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push API
        uses: docker/build-push-action@v5
        with:
          context: .
          file: ./Dockerfile.api
          push: true
          tags: ghcr.io/${{ github.repository }}/api:${{ github.ref_name }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - name: Deploy to Railway
        uses: bervProject/railway-deploy@v1.0.0
        with:
          railway_token: ${{ secrets.RAILWAY_TOKEN }}
          service: hostel-api

      - name: Run database migrations
        run: |
          railway run pnpm prisma migrate deploy

      - name: Notify Discord
        uses: sarisia/actions-status-discord@v1
        with:
          webhook: ${{ secrets.DISCORD_WEBHOOK }}
          title: 'Production Deployment'
          description: 'Deployed version ${{ github.ref_name }}'
          color: 0x00ff00
```

## Todo List

- [ ] Create test setup and helpers
- [ ] Write unit tests for all services
- [ ] Write integration tests for API
- [ ] Create Playwright E2E tests
- [ ] Configure test coverage reporting
- [ ] Create CI workflow
- [ ] Create Dockerfiles
- [ ] Configure staging deployment
- [ ] Configure production deployment
- [ ] Set up database migrations in CI
- [ ] Configure monitoring alerts
- [ ] Write deployment runbook
- [ ] Set up rollback procedures
- [ ] Load testing with k6

## Success Criteria

- [ ] Test coverage >80%
- [ ] CI pipeline completes <10 minutes
- [ ] All E2E tests pass
- [ ] Staging deployment works
- [ ] Production deployment with zero downtime
- [ ] Rollback completes <5 minutes
- [ ] Monitoring dashboards show healthy metrics

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| Flaky tests | Medium | Retry logic, test isolation |
| Long CI times | Medium | Parallel jobs, caching |
| Failed deployment | High | Staging gate, rollback plan |
| Data loss | Critical | Backup before migration |

## Security Considerations

- No secrets in code (use GitHub secrets)
- Separate staging/production credentials
- Database backup before migrations
- Audit log for deployments
- Restrict production access

## Next Steps

After Phase 10, the platform is ready for:
1. Beta testing with real hostels
2. Performance optimization based on metrics
3. Feature iteration based on feedback
4. Scale infrastructure as needed

## Rollback Procedure

```bash
# 1. Identify last working version
git tag -l 'v*' --sort=-v:refname | head -5

# 2. Deploy previous version
railway deploy --service hostel-api --image ghcr.io/org/api:v1.0.0

# 3. Rollback database if needed
railway run pnpm prisma migrate resolve --rolled-back <migration_name>

# 4. Verify health
curl https://api.hostelhub.vn/health

# 5. Notify team
curl -X POST $DISCORD_WEBHOOK -d '{"content": "Rollback completed"}'
```
