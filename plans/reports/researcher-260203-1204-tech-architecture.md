# Technical Architecture Research: NestJS + Next.js SaaS

**Date:** 2026-02-03
**Context:** /home/welterial/projects/swdv2
**Focus:** Hostel/booking management SaaS platform

---

## 1. Monorepo Strategy

### Turborepo (Recommended)
**Pros:**
- Minimal config, faster setup vs Nx
- Incremental builds with intelligent caching
- Built-in remote caching (Vercel integration)
- Perfect for Next.js + NestJS stack
- Lighter learning curve

**Cons:**
- Fewer code generation tools vs Nx
- Less opinionated structure

### Nx
**Pros:**
- Comprehensive tooling ecosystem
- Built-in generators, dependency graph visualization
- Better for large teams (100+ developers)

**Cons:**
- Steeper learning curve
- More boilerplate, complex configuration

### Separate Repos
**Avoid for SaaS:** Creates versioning hell, duplicated types, slower iterations.

**Recommendation:** **Turborepo** for speed + simplicity. Structure:
```
apps/
  api/          # NestJS backend
  web/          # Next.js frontend
  admin/        # Admin dashboard (optional)
packages/
  shared/       # Shared types, utilities
  ui/           # Shared UI components
  database/     # Prisma schema, migrations
```

---

## 2. Authentication Strategy

### Analysis (from research)

**Better-Auth:**
- NestJS integration via `@thallesp/nestjs-better-auth`
- JWT cookie-based sessions, auto-refresh
- Quick setup but community-maintained package
- Good for rapid prototyping

**NextAuth (v5/Auth.js):**
- `@next-nest-auth/nextauth` (Dec 2025 release)
- Best Next.js integration, CSRF protection
- Frontend-focused, requires NestJS adapter layer
- Strong OAuth provider ecosystem

**Custom JWT (Passport.js):**
- `@nestjs/passport` + `@nestjs/jwt`
- Full control over auth flow
- Backend-agnostic, supports mobile/multiple clients
- More boilerplate but battle-tested

### Recommendation: **Custom JWT with Passport.js**
**Rationale:**
- Multi-tenant SaaS needs flexible auth (owner/staff/guest roles)
- Future mobile app support requires backend-agnostic solution
- Fine-grained permission control (room access, booking management)
- Industry standard, extensive NestJS docs

**Implementation:**
- Access tokens (15min expiry) + Refresh tokens (7 days)
- Role-based guards: `@Roles('owner', 'staff')`
- Tenant isolation via JWT claims: `{ userId, tenantId, role }`

---

## 3. Multi-Tenant Architecture

### Pattern Options

**1. Separate Database per Tenant**
- Strongest isolation, easiest compliance
- Expensive, complex backups/migrations

**2. Shared Database, Separate Schemas**
- Balanced isolation, PostgreSQL native support
- Moderate complexity

**3. Shared Schema with Tenant Column** (Recommended)
- Cost-effective, simpler migrations
- Requires strict Row-Level Security (RLS)

### Recommended: Shared Schema + RLS

**PostgreSQL Setup:**
```sql
-- All tables include tenant_id
CREATE TABLE bookings (
  id UUID PRIMARY KEY,
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  -- ...
);

-- Row-Level Security
ALTER TABLE bookings ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation ON bookings
  USING (tenant_id = current_setting('app.current_tenant')::UUID);
```

**NestJS Middleware:**
```typescript
// Inject tenant from JWT into connection
connection.query(`SET app.current_tenant = '${tenantId}'`);
```

**Benefits:**
- Automatic tenant isolation at DB level
- Prevents accidental cross-tenant queries
- Single database = simpler ops

---

## 4. Real-Time Features

### WebSocket vs SSE

**WebSocket (via `@nestjs/websockets`):**
- **Use for:** Bidirectional features (live chat, collaborative editing)
- Bidirectional, lower latency
- More complex: connection management, heartbeats
- Harder to scale (sticky sessions)

**Server-Sent Events (SSE):**
- **Use for:** Server→client updates (booking notifications, availability changes)
- Simpler, HTTP-based (auto-reconnect)
- Easier horizontal scaling
- One-way only

### Recommendation: **SSE for booking updates**
**Rationale:**
- Booking notifications = server-initiated events
- No need for client→server real-time (forms handle that)
- Better scalability for SaaS with many tenants

**NestJS SSE Example:**
```typescript
@Sse('bookings/stream')
bookingStream(@Request() req): Observable<MessageEvent> {
  return this.bookingService.getBookingUpdates(req.user.tenantId);
}
```

**Add WebSocket later if:**
- Live chat between staff/guests needed
- Real-time collaborative calendar editing

---

## 5. Database Schema (Hostel/Booking)

### Core Entities

**Tenants (Hostels):**
```typescript
tenant_id, name, subdomain, plan_tier, settings (JSONB)
```

**Properties:**
```typescript
property_id, tenant_id, name, address, total_rooms
```

**Rooms:**
```typescript
room_id, property_id, room_number, type (dorm|private),
capacity, beds[], amenities (JSONB), base_price
```

**Bookings:**
```typescript
booking_id, tenant_id, room_id, guest_id,
check_in, check_out, total_price, status, metadata (JSONB)
```

**Guests:**
```typescript
guest_id, tenant_id, email, phone, id_document (encrypted),
booking_history
```

**Users (Staff):**
```typescript
user_id, tenant_id, email, role (owner|manager|receptionist),
permissions (JSONB)
```

### Key Design Decisions

**1. Soft Deletes:**
```typescript
deleted_at TIMESTAMP NULL
```
Critical for financial records, booking history.

**2. Audit Trail:**
```typescript
created_by, updated_by, created_at, updated_at
```

**3. JSONB for Flexibility:**
- Room amenities (varies by property)
- Tenant settings (custom fields per hostel)
- Booking metadata (special requests)

**4. Indexes:**
```sql
-- Tenant isolation queries
CREATE INDEX idx_bookings_tenant ON bookings(tenant_id);
CREATE INDEX idx_bookings_dates ON bookings(check_in, check_out);

-- Availability searches
CREATE INDEX idx_rooms_property ON rooms(property_id)
  WHERE deleted_at IS NULL;
```

---

## 6. API Design

### REST vs GraphQL vs tRPC

**REST:**
- **Pros:** Universal, cacheable (HTTP), well-understood
- **Cons:** Over/under-fetching, versioning challenges
- **Best for:** Public API, third-party integrations

**GraphQL:**
- **Pros:** Flexible queries, single endpoint, schema-driven
- **Cons:** Complex caching, N+1 query issues, learning curve
- **Best for:** Complex data relationships, mobile apps with varied needs

**tRPC:**
- **Pros:** End-to-end type safety (TS only), zero code-gen, small bundle
- **Cons:** TypeScript-only, less mature ecosystem
- **Best for:** Monorepo TypeScript stacks (Next.js + NestJS)

### Recommendation: **tRPC for internal API + REST for external**

**Rationale:**
- Monorepo = shared types between frontend/backend
- tRPC eliminates API contract drift
- Faster development (no manual type definitions)
- Add REST endpoints for webhook integrations (payment providers)

**Implementation:**
```typescript
// Backend (NestJS tRPC adapter)
export const appRouter = router({
  booking: {
    create: protectedProcedure
      .input(createBookingSchema)
      .mutation(({ ctx, input }) => ctx.bookingService.create(input)),
    list: protectedProcedure.query(({ ctx }) =>
      ctx.bookingService.findByTenant(ctx.user.tenantId)
    ),
  },
});

// Frontend (Next.js) - full type safety
const { data } = trpc.booking.list.useQuery();
//    ^? Booking[] - no manual typing!
```

**REST subset:**
- `/webhooks/stripe` - Payment confirmations
- `/api/public/availability` - Public room search (if needed)

---

## Architecture Summary

### Recommended Stack

| Layer | Technology | Rationale |
|-------|-----------|-----------|
| **Monorepo** | Turborepo | Simplicity, Vercel integration |
| **Auth** | Custom JWT (Passport.js) | Flexibility, multi-client support |
| **Multi-tenancy** | Shared schema + RLS | Cost-effective, secure |
| **Real-time** | SSE (WebSocket later) | Simpler, better scaling |
| **API** | tRPC + REST subset | Type safety, external integrations |
| **Database** | PostgreSQL + Prisma | RLS support, type-safe ORM |
| **Caching** | Redis | Session storage, availability cache |

### Data Flow

1. **Auth:** Next.js → tRPC → NestJS → JWT validation → Tenant context
2. **Booking:** User request → RLS policy enforced → Query scoped to tenant
3. **Real-time:** NestJS event → SSE stream → Next.js updates UI

### File Structure
```
apps/
  api/              # NestJS
    src/
      modules/
        auth/       # Passport JWT strategy
        booking/    # Booking service + tRPC routers
        tenant/     # Tenant context middleware
      trpc/         # tRPC server setup
  web/              # Next.js
    src/
      app/          # App router (Next.js 15)
      trpc/         # tRPC client
packages/
  database/         # Prisma schema with RLS
  shared/           # Zod schemas, types
  ui/               # Shadcn components
```

---

## Unresolved Questions

1. **Payment integration:** Stripe tenant-connected accounts vs platform model?
2. **File storage:** S3 vs Cloudinary for ID documents, property images?
3. **Email provider:** Resend, SendGrid, or AWS SES for booking confirmations?
4. **Deployment:** Single Kubernetes cluster vs separate services (Vercel + Railway)?
5. **Rate limiting:** Per-tenant or global? Redis-based or NestJS throttler?

---

## Next Steps

1. Create detailed database schema with Prisma models
2. Setup Turborepo workspace with shared packages
3. Implement tenant context middleware + RLS policies
4. Build tRPC router skeleton with auth guards
5. Design booking availability algorithm (consider overbooking rules)
