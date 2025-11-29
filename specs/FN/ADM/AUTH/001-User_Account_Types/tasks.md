# FN/ADM/AUTH/001 - Tasks

## Subtasks

### FN/ADM/AUTH/001-01: Define User and UserRole Prisma models
- [ ] Add UserRole enum (MANAGER, MEMBER) to schema
- [ ] Add User model with fields:
  - id, email, passwordHash, role, createdAt, updatedAt
  - isActive, lastLoginAt, mustResetPassword
- [ ] Add unique constraint on email
- [ ] Create migration
- [ ] Add seed data for default admin user

### FN/ADM/AUTH/001-02: Setup NextAuth.js with credentials provider
- [ ] Install next-auth and @auth/prisma-adapter
- [ ] Configure NextAuth route handler
- [ ] Implement credentials provider with bcrypt password verification
- [ ] Configure JWT strategy with custom session callback
- [ ] Add auth configuration file

### FN/ADM/AUTH/001-03: Create login/logout UI
- [ ] Create `/login` page with email/password form
- [ ] Add form validation with zod
- [ ] Implement signIn action
- [ ] Create logout button component
- [ ] Add error handling and loading states

### FN/ADM/AUTH/001-04: Add session management
- [ ] Create useSession hook wrapper
- [ ] Add session provider to layout
- [ ] Display current user info in header
- [ ] Implement session refresh logic
- [ ] Add session expiry handling

### FN/ADM/AUTH/001-05: Create user management admin UI
- [ ] Create `/admin/users` page
- [ ] Add user list table with columns: email, role, lastLogin, status
- [ ] Implement create user dialog
- [ ] Implement edit user dialog
- [ ] Add toggle active/inactive
- [ ] Add password reset function (admin-initiated)
