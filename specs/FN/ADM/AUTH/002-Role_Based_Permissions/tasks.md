# FN/ADM/AUTH/002 - Tasks

## Subtasks

### FN/ADM/AUTH/002-01: Define permission matrix
- [ ] Create permissions.ts with Permission enum
- [ ] Define role-to-permission mappings
- [ ] Add hasPermission(role, permission) utility
- [ ] Document permission matrix in codebase

### FN/ADM/AUTH/002-02: Create auth middleware
- [ ] Create withAuth middleware for API routes
- [ ] Create withRole higher-order component
- [ ] Add getServerSession utility function
- [ ] Implement unauthorized response handling

### FN/ADM/AUTH/002-03: Protect admin routes (Manager only)
- [ ] Add middleware.ts with route matcher
- [ ] Protect /admin/* routes
- [ ] Add redirect to login for unauthenticated
- [ ] Add redirect to home for unauthorized role
- [ ] Show access denied toast message

### FN/ADM/AUTH/002-04: Implement Member-only views
- [ ] Create /my-roster page (Member dashboard)
- [ ] Show personal schedule calendar view
- [ ] Display upcoming shifts
- [ ] Show request status

### FN/ADM/AUTH/002-05: Add duty request submission for Members
- [ ] Create DutyRequest model in Prisma
- [ ] Build request submission form
- [ ] Add request types: DAY_OFF, NO_NIGHT, PREFERENCE
- [ ] Implement request list view
- [ ] Add request cancellation
