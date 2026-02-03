---
title: "Hostel Management SaaS Platform"
description: "B2B SaaS for Vietnam hostel owners with guest booking portal"
status: pending
priority: P1
effort: 12-16 weeks
branch: main
tags: [saas, hospitality, vietnam, nestjs, nextjs, turborepo]
created: 2026-02-03
---

# Hostel Management SaaS - Vietnam Market

## Overview

Multi-tenant B2B SaaS platform enabling hostel owners to manage properties, rooms, beds, bookings, and staff. Guests search/book hostels via public portal.

**Monetization:** Subscription tiers (Basic/Pro/Enterprise) via SePay (VietQR) + VNPay

## Tech Stack

| Layer | Technology |
|-------|------------|
| Monorepo | Turborepo |
| Frontend | Next.js 15 (App Router), shadcn/ui, Tailwind |
| Backend | NestJS, tRPC + REST webhooks |
| Database | PostgreSQL + Prisma + RLS |
| Auth | Custom JWT (Passport.js) |
| Real-time | SSE |
| Cache | Redis |
| Queue | RabbitMQ |
| Payments | SePay (VietQR), VNPay |
| Search | PostgreSQL Full-Text |
| Files | S3/Cloudinary |
| Monitoring | ELK, Prometheus, Grafana |

## Phase Overview

| Phase | Name | Effort | Status |
|-------|------|--------|--------|
| 01 | [Project Setup](phase-01-project-setup-monorepo-configs-cicd.md) | 3d | Pending |
| 02 | [Database Design](phase-02-database-design-prisma-schema-rls-migrations.md) | 4d | Pending |
| 03 | [Auth System](phase-03-auth-system-jwt-roles-guards.md) | 3d | Pending |
| 04 | [Core API](phase-04-core-api-trpc-routers-services.md) | 5d | Pending |
| 05 | [Owner Dashboard](phase-05-owner-dashboard-property-booking-reports-ui.md) | 8d | Pending |
| 06 | [Guest Platform](phase-06-guest-platform-search-listing-booking-ui.md) | 6d | Pending |
| 07 | [Payment Integration](phase-07-payment-integration-sepay-vnpay-subscriptions.md) | 5d | Pending |
| 08 | [Real-time & Notifications](phase-08-real-time-sse-notifications-alerts.md) | 3d | Pending |
| 09 | [Monitoring & Logging](phase-09-monitoring-logging-elk-prometheus-grafana.md) | 3d | Pending |
| 10 | [Testing & Deployment](phase-10-testing-deployment-cicd-production.md) | 5d | Pending |

## Key Dependencies

- SePay/VNPay merchant accounts (2-4 weeks onboarding)
- S3/Cloudinary account setup
- Domain + SSL certificates
- PostgreSQL + Redis hosting

## Research References

- [Tech Architecture](../reports/researcher-260203-1204-tech-architecture.md)
- [Vietnam Payments](../reports/researcher-260203-1204-vietnam-payments.md)
- [Competitor Analysis](../reports/researcher-260203-1204-hostel-saas-competitors.md)

## Critical Path

1. Phase 01-03: Foundation (auth, db, monorepo)
2. Phase 04-05: Owner MVP (property/booking management)
3. Phase 07: Payment integration (revenue enablement)
4. Phase 06: Guest portal (public booking)
5. Phase 08-10: Polish, monitoring, deployment
