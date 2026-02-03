# HostelViet

> Vietnam's Modern Hostel Management SaaS Platform

HostelViet is a comprehensive B2B SaaS solution empowering hostel owners across Vietnam to manage properties, bookings, and operations while providing travelers with an intuitive booking platform.

## Overview

**Business Model:** B2B SaaS with dual-sided platform
- **Hostel Owners** (Customers): Pay subscription for management tools
- **Travelers** (End Users): Search and book hostels for free

**Market Focus:** Vietnam market with localized payments (VietQR, VNPay)

## Key Features

### For Hostel Owners
- Multi-property management with room/bed inventory
- Real-time booking management and availability tracking
- Staff roles and permissions (Manager, Receptionist, Cleaner)
- Revenue analytics and occupancy reports
- Automated guest communications via SSE
- Subscription management (Basic/Pro/Enterprise tiers)

### For Travelers
- Full-text search with filters (location, price, amenities)
- Hostel details with photos, reviews, and ratings
- Real-time availability and instant booking
- VietQR and VNPay payment options
- Booking history and profile management

## Tech Stack

```
┌─────────────────────────────────────────────────┐
│                   Frontend                       │
│  Next.js 15 (App Router) + shadcn/ui + Tailwind │
└────────────────┬────────────────────────────────┘
                 │ tRPC
┌────────────────┴────────────────────────────────┐
│                   Backend                        │
│        NestJS + tRPC + REST Webhooks            │
└────────────────┬────────────────────────────────┘
                 │
    ┌────────────┼────────────┬──────────┐
    │            │            │          │
┌───▼────┐  ┌───▼────┐  ┌───▼───┐  ┌──▼────┐
│ Postgres│  │ Redis  │  │RabbitMQ│  │  S3   │
│ + Prisma│  │ Cache  │  │ Queue  │  │Files  │
└─────────┘  └────────┘  └────────┘  └───────┘
```

### Core Technologies
- **Monorepo:** Turborepo for unified builds
- **Frontend:** Next.js 15, shadcn/ui, Tailwind CSS
- **Backend:** NestJS, tRPC, Passport.js JWT
- **Database:** PostgreSQL with Prisma ORM and RLS
- **Real-time:** Server-Sent Events (SSE)
- **Cache:** Redis for sessions and queries
- **Queue:** RabbitMQ for async jobs
- **Payments:** SePay (VietQR) + VNPay
- **Search:** PostgreSQL Full-Text Search
- **Storage:** AWS S3 or Cloudinary
- **Monitoring:** ELK Stack, Prometheus, Grafana

## Project Structure

```
swdv2/
├── apps/
│   ├── web/                 # Next.js public portal (travelers)
│   ├── dashboard/           # Next.js owner dashboard
│   └── api/                 # NestJS backend API
├── packages/
│   ├── ui/                  # Shared UI components (shadcn/ui)
│   ├── config/              # Shared configs (ESLint, TS, Tailwind)
│   ├── database/            # Prisma schema and migrations
│   └── types/               # Shared TypeScript types
├── docs/                    # Documentation
├── plans/                   # Implementation plans
└── turbo.json               # Turborepo config
```

## Quick Start

### Prerequisites
- Node.js 20+ and pnpm 9+
- PostgreSQL 16+
- Redis 7+
- RabbitMQ 3.13+

### Installation

```bash
# Clone repository
git clone https://github.com/your-org/swdv2.git
cd swdv2

# Install dependencies
pnpm install

# Setup environment variables
cp .env.example .env
# Edit .env with your credentials

# Run database migrations
pnpm db:migrate

# Seed development data
pnpm db:seed
```

### Development

```bash
# Start all apps in development mode
pnpm dev

# Start specific app
pnpm dev --filter=web
pnpm dev --filter=dashboard
pnpm dev --filter=api

# Access applications
# - Public Portal: http://localhost:3000
# - Owner Dashboard: http://localhost:3001
# - API: http://localhost:4000
```

### Build

```bash
# Build all packages and apps
pnpm build

# Build specific app
pnpm build --filter=web

# Type checking
pnpm typecheck

# Linting
pnpm lint

# Format code
pnpm format
```

### Testing

```bash
# Run all tests
pnpm test

# Unit tests only
pnpm test:unit

# Integration tests
pnpm test:integration

# E2E tests
pnpm test:e2e

# Test coverage
pnpm test:coverage
```

## Environment Variables

Create `.env` file in project root with the following variables:

```bash
# Database
DATABASE_URL="postgresql://user:password@localhost:5432/hostelviet"

# Redis
REDIS_URL="redis://localhost:6379"

# RabbitMQ
RABBITMQ_URL="amqp://localhost:5672"

# JWT Secrets
JWT_ACCESS_SECRET="your-access-secret"
JWT_REFRESH_SECRET="your-refresh-secret"

# AWS S3 (or Cloudinary)
AWS_ACCESS_KEY_ID="your-key"
AWS_SECRET_ACCESS_KEY="your-secret"
AWS_S3_BUCKET="hostelviet-uploads"
AWS_REGION="ap-southeast-1"

# Payment Providers
SEPAY_API_KEY="your-sepay-key"
VNPAY_TMN_CODE="your-vnpay-code"
VNPAY_HASH_SECRET="your-vnpay-secret"

# App URLs
NEXT_PUBLIC_WEB_URL="http://localhost:3000"
NEXT_PUBLIC_DASHBOARD_URL="http://localhost:3001"
API_URL="http://localhost:4000"
```

## Development Workflow

### Code Standards
- TypeScript strict mode enabled
- ESLint + Prettier for code formatting
- File naming: kebab-case (e.g., `user-profile.tsx`)
- Component naming: PascalCase (e.g., `UserProfile`)
- Maximum file size: 200 lines (split larger files)

### Git Workflow
```bash
# Create feature branch
git checkout -b feature/booking-calendar

# Make changes and commit
git add .
git commit -m "feat(booking): add calendar view component"

# Run pre-push checks
pnpm lint && pnpm typecheck && pnpm test

# Push and create PR
git push origin feature/booking-calendar
```

### Commit Convention
Follow [Conventional Commits](https://www.conventionalcommits.org/):
- `feat:` New feature
- `fix:` Bug fix
- `docs:` Documentation changes
- `style:` Code formatting
- `refactor:` Code restructuring
- `test:` Test additions/changes
- `chore:` Maintenance tasks

## API Documentation

### tRPC Endpoints
- **Auth:** `/trpc/auth.login`, `/trpc/auth.register`, `/trpc/auth.refresh`
- **Properties:** `/trpc/property.list`, `/trpc/property.create`, `/trpc/property.update`
- **Bookings:** `/trpc/booking.search`, `/trpc/booking.create`, `/trpc/booking.cancel`
- **Subscriptions:** `/trpc/subscription.plans`, `/trpc/subscription.subscribe`

### REST Webhooks
- `POST /webhooks/sepay` - SePay payment notifications
- `POST /webhooks/vnpay` - VNPay payment notifications

Full API documentation: [docs/api-reference.md](docs/api-reference.md)

## Database Schema

Key entities:
- **Tenant:** Hostel owner organization (multi-tenant isolation)
- **Property:** Hostel/hotel managed by tenant
- **Room:** Physical rooms in property
- **Bed:** Individual beds in dorm rooms
- **Booking:** Guest reservations
- **User:** System users (owners, staff, guests)
- **Subscription:** Tenant subscription plans

See detailed schema: [docs/database-schema.md](docs/database-schema.md)

## Security

- **Authentication:** JWT with access (15min) and refresh (7d) tokens
- **Authorization:** Role-based access control (RBAC)
- **Multi-tenancy:** Row-level security (RLS) in PostgreSQL
- **Data Protection:** Encrypted sensitive fields, HTTPS only
- **Rate Limiting:** Redis-based API throttling
- **Input Validation:** Zod schemas on all inputs

## Deployment

### Production Requirements
- Node.js 20+ runtime
- PostgreSQL 16+ with 2+ vCPU, 4GB+ RAM
- Redis 7+ with persistence
- RabbitMQ 3.13+ cluster
- CDN for static assets
- SSL certificates

### CI/CD Pipeline
- GitHub Actions for automated testing
- Docker images for all services
- Kubernetes deployment manifests
- Automated database migrations
- Blue-green deployment strategy

See deployment guide: [docs/deployment-guide.md](docs/deployment-guide.md)

## Contributing

1. Read [docs/code-standards.md](docs/code-standards.md)
2. Fork the repository
3. Create feature branch from `main`
4. Make changes following code standards
5. Write/update tests for changes
6. Ensure all tests pass and no linting errors
7. Submit pull request with clear description

## Documentation

- [Project Overview & PDR](docs/project-overview-pdr.md)
- [System Architecture](docs/system-architecture.md)
- [Code Standards](docs/code-standards.md)
- [Design Guidelines](docs/design-guidelines.md)
- [Development Roadmap](docs/development-roadmap.md)

## License

Proprietary - All rights reserved

## Support

- **Documentation:** [docs/](docs/)
- **Issues:** [GitHub Issues](https://github.com/your-org/swdv2/issues)
- **Email:** support@hostelviet.com

---

Built with ❤️ for Vietnam's hospitality industry
