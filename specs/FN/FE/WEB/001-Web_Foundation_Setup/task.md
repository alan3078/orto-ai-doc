# Tasks - FN/FE/WEB/001 - Web Foundation Setup

## Context

We are building the foundational Next.js web application that will serve as both the Admin CMS and User Frontend. This setup includes TypeScript, PostgreSQL, Prisma ORM, and shadcn/ui. **Authentication is excluded** and will be added later.

**Goal**: Have a running Next.js app that can read/write to PostgreSQL with a modern UI component system.

---

## Jira Ticket Breakdown

### FN/FE/WEB/001-01: Initialize Next.js Project

- [ ] Verify Node.js v22.11.0 is installed (`node -v`)
  - If not: `nvm install 22.11.0 && nvm use 22.11.0`
- [ ] Create `.nvmrc` file in project root with `v22.11.0`
- [ ] Run `npx create-next-app@latest web` with options:
  - ✅ TypeScript
  - ✅ ESLint
  - ✅ Tailwind CSS
  - ✅ `src/` directory
  - ✅ App Router
  - ✅ Import alias: `@/*`
- [ ] `cd web` and verify project structure created
- [ ] Run `npm run dev` and verify starts successfully
- [ ] **Self-Check**: Visit `http://localhost:3000` and see Next.js welcome page

---

### FN/FE/WEB/001-02: Setup PostgreSQL Database

- [ ] Verify Docker Desktop is running
- [ ] Start PostgreSQL container:
  ```bash
  docker run --name orto-postgres \
    -e POSTGRES_PASSWORD=dev_password \
    -e POSTGRES_DB=orto_scheduler \
    -p 5432:5432 \
    -d postgres:16
  ```
- [ ] Verify container is running: `docker ps | grep orto-postgres`
- [ ] Test connection:
  ```bash
  docker exec -it orto-postgres psql -U postgres -d orto_scheduler
  ```
- [ ] In psql, type `\l` to list databases, then `\q` to exit
- [ ] **Self-Check**: Container restarts successfully after `docker restart orto-postgres`

---

### FN/FE/WEB/001-03: Configure Prisma ORM

- [ ] Install Prisma packages:
  ```bash
  npm install prisma @prisma/client
  ```
- [ ] Initialize Prisma:
  ```bash
  npx prisma init
  ```
- [ ] Create `.env.local` file in `/web` with:
  ```bash
  DATABASE_URL="postgresql://postgres:dev_password@localhost:5432/orto_scheduler?schema=public"
  NEXT_PUBLIC_APP_NAME="Universal Scheduler"
  ```
- [ ] Update `prisma/schema.prisma` with User model (copy from `implementation.md`)
- [ ] Run first migration:
  ```bash
  npx prisma migrate dev --name init
  ```
- [ ] Generate Prisma Client:
  ```bash
  npx prisma generate
  ```
- [ ] Create `/src/lib/prisma.ts` with singleton pattern (copy from `implementation.md`)
- [ ] **Self-Check**: `npx prisma studio` opens at `http://localhost:5555` and shows User table

---

### FN/FE/WEB/001-04: Initialize shadcn/ui

- [ ] Run shadcn/ui init:
  ```bash
  npx shadcn-ui@latest init
  ```
  - Style: **Default**
  - Base color: **Slate**
  - CSS variables: **Yes**
- [ ] Verify `components.json` was created
- [ ] Verify `tailwind.config.ts` was updated with theme
- [ ] Create `/src/lib/utils.ts` with `cn()` helper (copy from `implementation.md`)
- [ ] Install Button component:
  ```bash
  npx shadcn-ui@latest add button
  ```
- [ ] Verify `/src/components/ui/button.tsx` was created
- [ ] Test Button in `app/page.tsx`:
  ```tsx
  import { Button } from '@/components/ui/button'
  
  <Button>Test Button</Button>
  ```
- [ ] **Self-Check**: Button renders with proper styling on homepage

---

### FN/FE/WEB/001-05: Create Database Test Pages & API

- [ ] Create `/src/app/test-db/page.tsx` (copy from `implementation.md`)
- [ ] Create `/src/app/api/users/route.ts` with GET and POST handlers (copy from `implementation.md`)
- [ ] Add test users via Prisma Studio:
  - Open `npx prisma studio`
  - Navigate to User model
  - Add 2-3 test users with email, name, and role
- [ ] Visit `http://localhost:3000/test-db`
- [ ] **Self-Check**: Page shows user count and list without errors
- [ ] Test API endpoint:
  ```bash
  curl http://localhost:3000/api/users
  ```
- [ ] **Self-Check**: Returns JSON array of users

---

### FN/FE/WEB/001-06: Create Homepage

- [ ] Update `/src/app/page.tsx` with starter content (copy from `implementation.md`)
- [ ] Add navigation to test-db page using shadcn Button
- [ ] Update `/src/app/layout.tsx` with proper metadata:
  ```tsx
  export const metadata = {
    title: 'Universal Scheduler',
    description: 'Nurse Edition - Roster Management System',
  }
  ```
- [ ] **Self-Check**: Homepage renders with proper layout and navigation works

---

### FN/FE/WEB/001-07: Final Integration Test

- [ ] Run TypeScript check:
  ```bash
  npm run build
  ```
  - Should complete with **0 errors**
- [ ] Run linting:
  ```bash
  npm run lint
  ```
  - Fix any issues that appear
- [ ] Verify `.env.local` is in `.gitignore`
- [ ] Test full user journey:
  1. Visit homepage
  2. Click "Test Database Connection" button
  3. See list of users
  4. Open Prisma Studio and add a new user
  5. Refresh page and see new user appear
- [ ] Stop dev server, clear `.next` folder, restart
- [ ] **Self-Check**: Everything still works after fresh restart

---

## 🎯 Definition of Done

- ✅ Next.js app runs on `localhost:3000` without errors
- ✅ Using Node.js v22.11.0
- ✅ PostgreSQL container runs and persists data across restarts
- ✅ Prisma migrations execute successfully
- ✅ Prisma Studio can connect and show User table
- ✅ shadcn/ui Button component renders correctly
- ✅ Test DB page shows database connectivity working
- ✅ API endpoints (`/api/users`) return proper JSON responses
- ✅ `npm run build` completes with zero TypeScript errors
- ✅ `.env.local` is added to `.gitignore`
- ✅ Can create, read users via both UI and API

---

## 📝 Notes

**Estimated Time**: 1.5-2 hours if following steps sequentially

**Common Issues**:
- If Prisma can't connect: Check Docker container is running (`docker ps`)
- If Next.js won't start: Delete `.next` folder and restart (`rm -rf .next && npm run dev`)
- If shadcn/ui fails: Ensure `components.json` has correct paths and Tailwind is configured
- If Node version mismatch: Run `nvm use 22` to switch to Node 22

**Next Steps After Completion**:
- ✅ **Recommended**: Start with FN/BE/ENG/001 (Engine MVP) - no dependencies
- Or start building first web feature (FN/ADM/STF/003 - Staff Import)
- Authentication can be added later as FN/FE/AUTH/001 when needed
