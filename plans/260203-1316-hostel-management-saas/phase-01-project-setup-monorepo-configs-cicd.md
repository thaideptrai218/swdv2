---
title: "Phase 01: Project Setup"
status: pending
priority: P1
effort: 3d
---

# Phase 01: Project Setup - Monorepo, Configs, CI/CD

## Context Links

- [Tech Architecture Report](../reports/researcher-260203-1204-tech-architecture.md)
- [Plan Overview](./plan.md)

## Overview

Initialize Turborepo monorepo with Next.js 15 frontend, NestJS backend, and shared packages. Configure ESLint, Prettier, TypeScript, and GitHub Actions CI/CD.

## Key Insights

- Turborepo chosen over Nx for simplicity and Vercel integration
- Shared packages reduce code duplication (types, UI, database)
- Remote caching accelerates CI builds

## Requirements

### Functional
- Monorepo with apps/ and packages/ structure
- Next.js 15 app with App Router
- NestJS API with modular architecture
- Shared TypeScript types package
- Shared UI components package (shadcn/ui)
- Prisma database package

### Non-Functional
- TypeScript strict mode enabled
- ESLint + Prettier consistent formatting
- CI builds under 5 minutes
- Branch protection rules on main

## Architecture

```
hostel-saas/
├── apps/
│   ├── api/                 # NestJS backend
│   │   ├── src/
│   │   │   ├── modules/     # Feature modules
│   │   │   ├── trpc/        # tRPC router setup
│   │   │   └── main.ts
│   │   └── package.json
│   └── web/                 # Next.js frontend
│       ├── src/
│       │   ├── app/         # App Router pages
│       │   ├── components/
│       │   └── trpc/        # tRPC client
│       └── package.json
├── packages/
│   ├── database/            # Prisma schema + client
│   ├── shared/              # Zod schemas, types, utils
│   └── ui/                  # shadcn/ui components
├── turbo.json
├── package.json
├── .github/workflows/
└── docker-compose.yml       # Local dev (Postgres, Redis)
```

## Related Code Files

### Create
- `turbo.json` - Turborepo pipeline config
- `package.json` - Root workspace config
- `apps/api/*` - NestJS application
- `apps/web/*` - Next.js application
- `packages/database/*` - Prisma package
- `packages/shared/*` - Shared types/utils
- `packages/ui/*` - UI components
- `.github/workflows/ci.yml` - CI pipeline
- `docker-compose.yml` - Local dev services
- `.env.example` - Environment template

## Implementation Steps

### 1. Initialize Monorepo (Day 1)

1. Create root directory with pnpm workspaces
2. Initialize Turborepo config
3. Configure root package.json scripts
4. Setup .gitignore, .nvmrc (Node 20 LTS)

```bash
pnpm init
pnpm add -D turbo
```

### 2. Create NestJS API (Day 1)

1. Scaffold NestJS app in apps/api
2. Configure TypeScript paths for monorepo
3. Add health check endpoint
4. Setup environment config module

```bash
cd apps && nest new api --package-manager pnpm
```

### 3. Create Next.js Web (Day 1)

1. Scaffold Next.js 15 with App Router
2. Configure Tailwind CSS
3. Install shadcn/ui CLI
4. Setup tRPC client skeleton

```bash
cd apps && pnpm create next-app web --typescript --tailwind --app
```

### 4. Create Shared Packages (Day 2)

1. `packages/database`: Prisma schema placeholder
2. `packages/shared`: Zod schemas, type exports
3. `packages/ui`: shadcn/ui component library
4. Configure tsconfig references

### 5. Configure Tooling (Day 2)

1. ESLint with shared config
2. Prettier with consistent rules
3. Husky + lint-staged for pre-commit
4. TypeScript strict mode all packages

### 6. Docker Compose (Day 2)

1. PostgreSQL 16 service
2. Redis service
3. RabbitMQ service (for later)
4. Network configuration

```yaml
services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: hostel_saas
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports:
      - "5432:5432"
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
```

### 7. CI/CD Pipeline (Day 3)

1. GitHub Actions workflow
2. Lint, typecheck, build steps
3. Turborepo remote caching
4. Branch protection rules

```yaml
name: CI
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - run: pnpm install --frozen-lockfile
      - run: pnpm turbo lint typecheck build
```

### 8. Environment Setup (Day 3)

1. Create .env.example with all variables
2. Document local setup in README
3. Add VS Code workspace settings

## Todo List

- [ ] Initialize pnpm workspace + Turborepo
- [ ] Scaffold NestJS API app
- [ ] Scaffold Next.js web app
- [ ] Create packages/database with Prisma
- [ ] Create packages/shared with types
- [ ] Create packages/ui with shadcn
- [ ] Configure ESLint + Prettier
- [ ] Setup Husky pre-commit hooks
- [ ] Create docker-compose.yml
- [ ] Create GitHub Actions CI workflow
- [ ] Write .env.example template
- [ ] Update README with setup instructions

## Success Criteria

- [ ] `pnpm install` completes without errors
- [ ] `pnpm turbo build` builds all apps/packages
- [ ] `pnpm turbo lint` passes all checks
- [ ] `docker compose up` starts Postgres + Redis
- [ ] NestJS health endpoint returns 200
- [ ] Next.js dev server renders home page
- [ ] CI pipeline passes on main branch

## Risk Assessment

| Risk | Impact | Mitigation |
|------|--------|------------|
| pnpm workspace resolution issues | Medium | Pin exact versions, use catalog |
| Turborepo cache invalidation | Low | Clear .turbo folder, verify turbo.json |
| TypeScript path aliases conflict | Medium | Use consistent @repo/* prefix |

## Security Considerations

- Never commit .env files
- Use secrets manager for production credentials
- Enable GitHub branch protection
- Review dependency vulnerabilities (pnpm audit)

## Next Steps

After completion, proceed to [Phase 02: Database Design](./phase-02-database-design-prisma-schema-rls-migrations.md)

## Environment Variables Template

```env
# Database
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/hostel_saas

# Redis
REDIS_URL=redis://localhost:6379

# JWT
JWT_SECRET=your-secret-key-min-32-chars
JWT_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d

# App
NODE_ENV=development
API_PORT=3001
WEB_PORT=3000
```
