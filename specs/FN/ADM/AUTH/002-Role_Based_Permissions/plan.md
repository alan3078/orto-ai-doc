# FN/ADM/AUTH/002 - Role-Based Permissions

> **V1.0 Requirement #9**: Manager: all client permissions; Member: Duty Viewer, duty request, password reset

## Overview

Implement role-based access control (RBAC) to differentiate Manager and Member capabilities.

## Goals

1. Define permission matrix for each role
2. Create authentication middleware
3. Protect admin routes for Manager only
4. Implement Member-only views
5. Add duty request submission for Members

## Scope

### In Scope
- Permission definitions (view, edit, delete, manage)
- Route-level protection middleware
- Component-level permission checks
- Member dashboard with limited functionality
- Duty request submission for Members

### Out of Scope
- Granular per-resource permissions
- Custom role creation
- Audit log for permission changes

## Success Criteria

- [ ] Manager can access all admin routes
- [ ] Member redirected from admin routes
- [ ] Member can view roster but not edit
- [ ] Member can submit duty requests
- [ ] Member can reset own password
- [ ] UI elements hidden based on role

## Dependencies

- FN/ADM/AUTH/001 (User Account Types) - **REQUIRED**

## Permission Matrix

| Feature | Manager | Member |
|---------|---------|--------|
| View Roster | ✅ | ✅ |
| Edit Roster | ✅ | ❌ |
| Generate Roster | ✅ | ❌ |
| Manage Staff | ✅ | ❌ |
| Manage Constraints | ✅ | ❌ |
| System Config | ✅ | ❌ |
| Submit Duty Request | ✅ | ✅ |
| Approve Requests | ✅ | ❌ |
| Manage Users | ✅ | ❌ |
| View Own Schedule | ✅ | ✅ |
| Reset Own Password | ✅ | ✅ |
