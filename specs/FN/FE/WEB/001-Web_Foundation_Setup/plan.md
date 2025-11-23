# Plan - FN/FE/WEB/001 - Web Foundation Setup

## 1. Overview

This feature establishes the foundational Next.js web application with TypeScript, PostgreSQL database, and UI component library. This forms the base infrastructure for the Admin CMS and User Frontend. Authentication will be added later in a separate feature.

## 2. Goals

**Production-Ready Foundation**: Set up a scalable, type-safe web application architecture that can handle database operations and UI rendering.

**Developer Experience**: Configure tooling for fast iteration, type safety, and code quality (ESLint, Prettier, TypeScript strict mode).

**Database-First Design**: Establish Prisma ORM with PostgreSQL for type-safe database access and automated migrations.

**Modern UI System**: Integrate shadcn/ui with Tailwind CSS for accessible, customizable components.

## 3. Scope (MVP)

For this first version, we will set up:

- **Next.js 15 with App Router**: Full-stack React framework with TypeScript
- **Node.js 22**: Latest LTS version for optimal performance
- **PostgreSQL Database**: Running in local Docker container for development
- **Prisma ORM**: Type-safe database client with migration system
- **shadcn/ui + Tailwind**: Component library with utility-first CSS
- **Basic Project Structure**: Organized folders for app routes, components, lib utilities

## 4. Out of Scope

- Authentication (NextAuth.js will be added later as FN/FE/AUTH/001)
- Actual feature implementation (rules, staff management, scheduling UI)
- Production deployment configuration
- CI/CD pipeline setup
- Advanced Prisma schema (only basic User table for testing)

## 5. Success Criteria

- Next.js dev server runs successfully on `localhost:3000`
- PostgreSQL container runs and accepts connections on `localhost:5432`
- Prisma migrations execute without errors
- Can install and use shadcn/ui components (e.g., Button)
- TypeScript compilation has zero errors
- Can create a "Hello World" page that reads from database
- Can perform basic CRUD operations on User table via Prisma

## 6. Dependencies

- **Requires**: Docker Desktop installed on local machine
- **Requires**: Node.js v22.11.0 (managed via nvm)
- **Blocks**: All FN/FE/* and FN/ADM/* features (need web foundation first)

---

**Estimated Effort**: 1-2 days (simplified without auth)  
**Priority**: P0 (Critical - Foundation for all web features)
