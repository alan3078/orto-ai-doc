# FN/ADM/AUTH/001 - User Account Types

> **V1.0 Requirement #8**: Generate 2 types of accounts: Manager / Member

## Overview

Implement user authentication with two account types (Manager/Member) using NextAuth.js with credentials provider.

## Goals

1. Define User and UserRole Prisma models
2. Setup NextAuth.js authentication
3. Create login/logout UI
4. Implement session management
5. Build user management admin interface

## Scope

### In Scope
- User model with email, password (hashed), role
- UserRole enum: MANAGER, MEMBER
- Credentials-based authentication
- JWT + Database session hybrid
- Login page with email/password form
- User profile display
- Admin panel to create/manage users

### Out of Scope
- OAuth providers (Google, GitHub)
- Magic link authentication
- 2FA (Version 1.1)

## Success Criteria

- [ ] User can register with email/password
- [ ] User can login/logout
- [ ] Session persists across page reloads
- [ ] Manager can view user management page
- [ ] Member cannot access admin routes

## Dependencies

- None (foundation feature)

## Blocks

- FN/ADM/AUTH/002 (Role-Based Permissions)
- FN/FE/ONB/001 (First-Time User Experience)
