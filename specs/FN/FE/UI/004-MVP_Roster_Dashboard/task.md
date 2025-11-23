# Tasks - FN/FE/UI/004 - MVP Roster Dashboard

## Context

We are building the primary user interface for the scheduling system - a single-page dashboard that orchestrates staff management, constraint configuration, and roster generation with real-time visual feedback. This feature connects all backend services (database, solver engine) through an intuitive React interface using TanStack Query for state management.

**Prerequisites**: 
- Python engine must be running (`cd engine && uv run uvicorn src.main:app --reload`)
- PostgreSQL database must be accessible
- FN/BE/DATA/003 must be complete (database schema and integration service)
- Next.js dev server running (`cd web && npm run dev`)

---

## Jira Ticket Breakdown

### FN/FE/UI/004-01: Install Required Shadcn UI Components

**Goal**: Add all necessary UI components from Shadcn

- [ ] Navigate to `/web` directory
- [ ] Run `npx shadcn@latest add table` - For roster grid
- [ ] Run `npx shadcn@latest add dialog` - For add staff/constraint modals
- [ ] Run `npx shadcn@latest add badge` - For state indicators
- [ ] Run `npx shadcn@latest add skeleton` - For loading states
- [ ] Run `npx shadcn@latest add select` - For dropdowns
- [ ] Run `npx shadcn@latest add dropdown-menu` - For constraint type picker
- [ ] Run `npx shadcn@latest add sonner` - For toast notifications
- [ ] Run `npx shadcn@latest add alert` - For error/info messages
- [ ] Verify all components added to `/web/src/components/ui/`
- [ ] **Self-Check**: Run `npm run build` - no TypeScript errors
- [ ] **Self-Check**: Import each component in a test file to verify

**Estimated Time**: 1 hour

---

### FN/FE/UI/004-02: Create Feature Folder Structure

**Goal**: Set up organized folder structure following existing patterns

- [ ] Create `/web/src/features/dashboard/` directory
- [ ] Create subdirectories:
  - [ ] `components/` - React components
  - [ ] `hooks/` - React Query hooks
  - [ ] `services/` - Types, schemas, query keys
- [ ] Create placeholder files:
  - [ ] `services/dashboard.service.ts` - DTOs and schemas
  - [ ] `hooks/use-staff.ts` - Staff query hooks
  - [ ] `hooks/use-constraints.ts` - Constraint query hooks
  - [ ] `hooks/use-roster.ts` - Roster query hooks
  - [ ] `components/staff-list.tsx` - Staff management UI
  - [ ] `components/add-staff-dialog.tsx` - Add staff form
  - [ ] `components/constraint-builder.tsx` - Constraint management UI
  - [ ] `components/add-constraint-form.tsx` - Add constraint form
  - [ ] `components/roster-controls.tsx` - Generate button and controls
  - [ ] `components/roster-grid.tsx` - Schedule visualization
  - [ ] `components/state-badge.tsx` - State indicator badge
- [ ] **Self-Check**: Folder structure matches `/features/users` pattern
- [ ] **Self-Check**: All files export placeholder components/functions

**Estimated Time**: 30 minutes

---

### FN/FE/UI/004-03: Define TypeScript Schemas and Types

**Goal**: Create validation schemas and type definitions

- [ ] Edit `/web/src/features/dashboard/services/dashboard.service.ts`
- [ ] Import Zod: `import { z } from 'zod'`
- [ ] Define `createStaffSchema`:
  - [ ] `name`: string, min 2, max 100
  - [ ] `employeeId`: string, regex `/^[A-Z0-9]{3,20}$/`
  - [ ] `email`: optional email or empty string
- [ ] Export `CreateStaffDto` type from schema
- [ ] Define `pointConstraintFormSchema`:
  - [ ] `type`: literal 'point'
  - [ ] `name`: string, min 1, max 100
  - [ ] `staffId`: string, min 1
  - [ ] `timeSlot`: number, int, min 0
  - [ ] `state`: number, int, 0-2
  - [ ] `description`: optional string
- [ ] Define `verticalSumConstraintFormSchema`:
  - [ ] `type`: literal 'vertical_sum'
  - [ ] `name`: string, min 1, max 100
  - [ ] `timeSlot`: union of number or 'ALL'
  - [ ] `targetState`: number, int, 0-2
  - [ ] `operator`: enum ['>=', '<=', '==']
  - [ ] `value`: number, int, min 0
  - [ ] `description`: optional string
- [ ] Define query key factories:
  - [ ] `staffKeys.all: ['staff']`
  - [ ] `constraintKeys.all: ['constraints']`
  - [ ] `rosterKeys.month: (month) => ['roster', { month }]`
- [ ] **Self-Check**: All schemas validate correctly with test data
- [ ] **Self-Check**: TypeScript types are properly inferred

**Estimated Time**: 1 hour

---

### FN/FE/UI/004-04: Implement Staff Server Actions

**Goal**: Create CRUD Server Actions for staff management

- [ ] Create `/web/src/app/actions/staff.actions.ts`
- [ ] Add `'use server'` directive at top
- [ ] Implement `getStaffAction()`:
  - [ ] Query `prisma.staff.findMany({ where: { isActive: true } })`
  - [ ] Order by name ascending
  - [ ] Return `{ success: true, staff }` or error
- [ ] Implement `createStaffAction(data)`:
  - [ ] Check for duplicate `employeeId`
  - [ ] Return error if exists
  - [ ] Create staff with `prisma.staff.create()`
  - [ ] Call `revalidatePath('/dashboard')`
  - [ ] Return `{ success: true, staff }` or error
- [ ] Implement `deleteStaffAction(id)`:
  - [ ] Soft delete: `prisma.staff.update({ data: { isActive: false } })`
  - [ ] Call `revalidatePath('/dashboard')`
  - [ ] Return `{ success: true }` or error
- [ ] **Self-Check**: Test each action in a script or API route
- [ ] **Self-Check**: Prisma Studio shows correct data changes

**Estimated Time**: 1.5 hours

---

### FN/FE/UI/004-05: Implement Constraint Server Actions

**Goal**: Create CRUD Server Actions for constraint management

- [ ] Create `/web/src/app/actions/constraints.actions.ts`
- [ ] Add `'use server'` directive
- [ ] Implement `getConstraintsAction()`:
  - [ ] Query `prisma.constraint.findMany({ where: { isActive: true } })`
  - [ ] Order by priority descending
  - [ ] Return `{ success: true, constraints }` or error
- [ ] Implement `createConstraintAction(data)`:
  - [ ] Accept: name, type, config, description, priority
  - [ ] Create with `prisma.constraint.create()`
  - [ ] Call `revalidatePath('/dashboard')`
  - [ ] Return `{ success: true, constraint }` or error
- [ ] Implement `deleteConstraintAction(id)`:
  - [ ] Soft delete: `prisma.constraint.update({ data: { isActive: false } })`
  - [ ] Call `revalidatePath('/dashboard')`
  - [ ] Return `{ success: true }` or error
- [ ] **Self-Check**: Test creating point and vertical_sum constraints
- [ ] **Self-Check**: Verify JSON config is stored correctly

**Estimated Time**: 1.5 hours

---

### FN/FE/UI/004-06: Update Roster Generation Action

**Goal**: Modify existing action to accept parameters

- [ ] Edit `/web/src/app/actions/generate-roster.action.ts`
- [ ] Update `generateRosterAction` signature to accept object params:
  - [ ] `name: string` - Roster name
  - [ ] `startDate: string` - ISO date string
  - [ ] `timeSlots: number` - Number of days
  - [ ] `staffIds: string[]` - Array of staff IDs
  - [ ] `constraintIds: string[]` - Array of constraint IDs
- [ ] Remove FormData parsing logic
- [ ] Pass params to `service.generateRoster()`
- [ ] Return `{ success, rosterId, status }` or error
- [ ] Implement `getRosterAction({ month })`:
  - [ ] Parse month string to date range
  - [ ] Query roster with shifts and staff included
  - [ ] Order by creation date descending
  - [ ] Return first matching roster
- [ ] **Self-Check**: Action can be called from client component
- [ ] **Self-Check**: Returns proper types for React Query

**Estimated Time**: 1 hour

---

### FN/FE/UI/004-07: Build React Query Hooks for Staff

**Goal**: Create hooks for staff data fetching and mutations

- [ ] Edit `/web/src/features/dashboard/hooks/use-staff.ts`
- [ ] Add `'use client'` directive
- [ ] Import React Query, toast, and staff actions
- [ ] Implement `useStaff()`:
  - [ ] Use `useQuery` with key `staffKeys.all`
  - [ ] Call `getStaffAction()` in queryFn
  - [ ] Set `staleTime: 5 * 60 * 1000` (5 min)
  - [ ] Throw error if not success
  - [ ] Return query result
- [ ] Implement `useAddStaff()`:
  - [ ] Use `useMutation` with `createStaffAction`
  - [ ] Implement optimistic update in `onMutate`
  - [ ] Rollback on error in `onError`
  - [ ] Invalidate `staffKeys.all` on success
  - [ ] Show toast notifications
  - [ ] Return mutation
- [ ] Implement `useDeleteStaff()`:
  - [ ] Use `useMutation` with `deleteStaffAction`
  - [ ] Invalidate `staffKeys.all` on success
  - [ ] Show toast notifications
  - [ ] Return mutation
- [ ] **Self-Check**: Hooks work with test component
- [ ] **Self-Check**: Optimistic updates and rollbacks function correctly

**Estimated Time**: 2 hours

---

### FN/FE/UI/004-08: Build React Query Hooks for Constraints

**Goal**: Create hooks for constraint data fetching and mutations

- [ ] Edit `/web/src/features/dashboard/hooks/use-constraints.ts`
- [ ] Add `'use client'` directive
- [ ] Implement `useConstraints()`:
  - [ ] Use `useQuery` with key `constraintKeys.all`
  - [ ] Call `getConstraintsAction()`
  - [ ] Set `staleTime: 5 * 60 * 1000`
- [ ] Implement `useAddConstraint()`:
  - [ ] Use `useMutation` with `createConstraintAction`
  - [ ] Invalidate `constraintKeys.all` on success
  - [ ] Show toast notifications
  - [ ] Return mutation
- [ ] Implement `useDeleteConstraint()`:
  - [ ] Use `useMutation` with `deleteConstraintAction`
  - [ ] Invalidate `constraintKeys.all` on success
  - [ ] Show toast notifications
- [ ] **Self-Check**: Can add point and vertical_sum constraints
- [ ] **Self-Check**: List updates immediately after mutations

**Estimated Time**: 1.5 hours

---

### FN/FE/UI/004-09: Build React Query Hooks for Roster with Polling

**Goal**: Create hooks for roster generation and status polling

- [ ] Edit `/web/src/features/dashboard/hooks/use-roster.ts`
- [ ] Add `'use client'` directive
- [ ] Implement `useRoster({ month })`:
  - [ ] Use `useQuery` with key `rosterKeys.month(month)`
  - [ ] Call `getRosterAction({ month })`
  - [ ] Set `staleTime: 0` (always fresh)
  - [ ] Add `refetchInterval` function:
    - [ ] Check if `roster?.status === 'SOLVING'`
    - [ ] Return `2000` (2s) if solving, `false` otherwise
  - [ ] Return query result
- [ ] Implement `useGenerateRoster()`:
  - [ ] Use `useMutation` with `generateRosterAction`
  - [ ] Extract month from `startDate` param
  - [ ] Invalidate `rosterKeys.month(month)` on success
  - [ ] Show toast notifications
  - [ ] Return mutation
- [ ] **Self-Check**: Polling starts when status is SOLVING
- [ ] **Self-Check**: Polling stops when status changes to COMPLETED/FAILED

**Estimated Time**: 2 hours

---

### FN/FE/UI/004-10: Build StaffList Component

**Goal**: Create staff management UI component

- [ ] Edit `/web/src/features/dashboard/components/staff-list.tsx`
- [ ] Add `'use client'` directive
- [ ] Import UI components (Card, Button, etc.)
- [ ] Import hooks: `useStaff`, `useDeleteStaff`
- [ ] Add state for dialog: `useState(false)`
- [ ] Render Card with header "Staff Members"
- [ ] Add "Add" button in header that opens dialog
- [ ] Show loading state when `isLoading`
- [ ] Show empty state when `staff?.length === 0`
- [ ] Render staff list with:
  - [ ] Staff name and employeeId
  - [ ] Delete button (trash icon)
  - [ ] Confirmation dialog on delete
- [ ] Make list scrollable: `max-h-[400px] overflow-y-auto`
- [ ] Import and render `<AddStaffDialog>` component
- [ ] **Self-Check**: List displays correctly with sample data
- [ ] **Self-Check**: Delete functionality works

**Estimated Time**: 2 hours

---

### FN/FE/UI/004-11: Build AddStaffDialog Component

**Goal**: Create form dialog for adding staff

- [ ] Edit `/web/src/features/dashboard/components/add-staff-dialog.tsx`
- [ ] Add `'use client'` directive
- [ ] Import Dialog, Form components
- [ ] Import `useAddStaff` hook and `createStaffSchema`
- [ ] Set up react-hook-form with zodResolver
- [ ] Render Dialog with:
  - [ ] DialogTrigger (controlled by parent)
  - [ ] DialogContent with form
  - [ ] DialogHeader with title "Add Staff Member"
- [ ] Add form fields:
  - [ ] Name input (required)
  - [ ] Employee ID input (required, uppercase)
  - [ ] Email input (optional)
- [ ] Handle submit:
  - [ ] Call `addStaff.mutate(data)`
  - [ ] Close dialog on success
  - [ ] Reset form
- [ ] Show loading state on submit button
- [ ] **Self-Check**: Form validation works
- [ ] **Self-Check**: Staff appears in list after adding

**Estimated Time**: 2 hours

---

### FN/FE/UI/004-12: Build ConstraintBuilder Component

**Goal**: Create constraint management UI

- [ ] Edit `/web/src/features/dashboard/components/constraint-builder.tsx`
- [ ] Add `'use client'` directive
- [ ] Import UI components (Card, Badge, DropdownMenu, etc.)
- [ ] Import hooks: `useConstraints`, `useDeleteConstraint`
- [ ] Add state: `formType` (null | 'point' | 'vertical_sum')
- [ ] Render Card with header "Active Constraints"
- [ ] Add DropdownMenu in header:
  - [ ] Trigger: "Add Rule" button
  - [ ] Items: "Day Off (Point)", "Min/Max Workers (Sum)"
  - [ ] Set `formType` on item click
- [ ] Conditionally render `<AddConstraintForm type={formType} />`
- [ ] Show loading state when `isLoading`
- [ ] Show empty state when no constraints
- [ ] Render constraints list with:
  - [ ] Type badge (color-coded)
  - [ ] Constraint name
  - [ ] Delete button
- [ ] Make list scrollable: `max-h-[300px]`
- [ ] **Self-Check**: Dropdown menu works
- [ ] **Self-Check**: Form switches based on type selection

**Estimated Time**: 2 hours

---

### FN/FE/UI/004-13: Build AddConstraintForm Component

**Goal**: Create dynamic form for constraint types

- [ ] Edit `/web/src/features/dashboard/components/add-constraint-form.tsx`
- [ ] Add `'use client'` directive
- [ ] Accept props: `type`, `onCancel`, `onSuccess`
- [ ] Import `useStaff`, `useAddConstraint` hooks
- [ ] Set up react-hook-form with appropriate schema based on type
- [ ] **For Point Constraint**:
  - [ ] Name input
  - [ ] Staff select (from useStaff)
  - [ ] Day select (0-6)
  - [ ] State select (0=Off, 1=Work, 2=OnCall)
- [ ] **For Vertical Sum**:
  - [ ] Name input
  - [ ] Apply to select (Specific Day | All Days)
  - [ ] Day select (conditional, if specific)
  - [ ] Operator select (>=, <=, ==)
  - [ ] Value input (number)
  - [ ] Target state select (1=Work, 2=OnCall)
- [ ] Handle submit:
  - [ ] Build config object from form data
  - [ ] Call `addConstraint.mutate({ name, type, config })`
  - [ ] Call `onSuccess()` to close form
- [ ] Add Cancel button that calls `onCancel()`
- [ ] **Self-Check**: Point constraint creates with correct config
- [ ] **Self-Check**: Vertical sum constraint creates with correct config

**Estimated Time**: 3 hours

---

### FN/FE/UI/004-14: Build StateBadge Component

**Goal**: Create reusable state indicator badge

- [ ] Edit `/web/src/features/dashboard/components/state-badge.tsx`
- [ ] Import Badge component
- [ ] Accept prop: `state: number`
- [ ] Define `STATE_CONFIG` object:
  - [ ] `0`: grey background, "Off", ○ icon
  - [ ] `1`: blue background, "Work", ● icon
  - [ ] `2`: purple background, "OnCall", ◐ icon
- [ ] Render Badge with:
  - [ ] Dynamic className from config
  - [ ] Icon character
  - [ ] Optional tooltip with label
- [ ] **Self-Check**: Badge displays correctly for each state
- [ ] **Self-Check**: Colors meet WCAG contrast requirements

**Estimated Time**: 30 minutes

---

### FN/FE/UI/004-15: Build RosterGrid Component

**Goal**: Create schedule visualization table

- [ ] Edit `/web/src/features/dashboard/components/roster-grid.tsx`
- [ ] Add `'use client'` directive
- [ ] Accept prop: `month: string`
- [ ] Import `useRoster` hook, Skeleton, Alert, Card components
- [ ] Call `useRoster({ month })`
- [ ] **Loading State**: Show Skeleton when loading or status=SOLVING
- [ ] **Empty State**: Show Card with Calendar icon and message
- [ ] **Failed State**: Show Alert with error message
- [ ] **Infeasible State**: Show Alert with warning message
- [ ] **Success State**: Render table:
  - [ ] Group shifts by staffId
  - [ ] Header row: "Staff" + Day columns with dates
  - [ ] Body rows: Staff name + StateBadge for each time slot
  - [ ] Sticky first column for staff names
  - [ ] Horizontal scroll: `overflow-x-auto`
- [ ] Add metadata footer:
  - [ ] Solver status
  - [ ] Solve time in ms
- [ ] Use `date-fns` to format dates
- [ ] **Self-Check**: Grid displays correctly with test roster
- [ ] **Self-Check**: Horizontal scroll works on small screens

**Estimated Time**: 3 hours

---

### FN/FE/UI/004-16: Build RosterControls Component

**Goal**: Create generation controls and month selector

- [ ] Edit `/web/src/features/dashboard/components/roster-controls.tsx`
- [ ] Add `'use client'` directive
- [ ] Import Button, Select components
- [ ] Import `useStaff`, `useConstraints`, `useGenerateRoster` hooks
- [ ] Add state: `selectedMonth` (default: current month)
- [ ] Generate list of last 3 months for selector
- [ ] Render header with:
  - [ ] Title: "Roster: {Month Year}"
  - [ ] Month Select dropdown
  - [ ] Generate button
  - [ ] Export CSV button (disabled for MVP)
- [ ] Handle Generate click:
  - [ ] Validate: `staff.length > 0`
  - [ ] Show error toast if no staff
  - [ ] Calculate startDate from selectedMonth
  - [ ] Get timeSlots (days in month)
  - [ ] Get all active staff IDs
  - [ ] Get all active constraint IDs
  - [ ] Call `generateRoster.mutate({ name, startDate, timeSlots, staffIds, constraintIds })`
- [ ] Show loading state: "Generating..." with spinner
- [ ] Disable button when generating or no staff
- [ ] **Self-Check**: Generate button triggers roster creation
- [ ] **Self-Check**: Month selector updates grid

**Estimated Time**: 2 hours

---

### FN/FE/UI/004-17: Create Dashboard Page

**Goal**: Assemble all components into main page

- [ ] Create `/web/src/app/dashboard/page.tsx`
- [ ] Leave as Server Component (do not add 'use client')
- [ ] Import all dashboard components
- [ ] Set up page metadata: title, description
- [ ] Render layout structure:
  - [ ] Container with padding
  - [ ] Header section
  - [ ] Grid layout: 12 columns
  - [ ] Sidebar (col-span-3): StaffList + ConstraintBuilder
  - [ ] Main (col-span-9): RosterControls + RosterGrid
- [ ] Add responsive classes:
  - [ ] `col-span-12` on mobile
  - [ ] `lg:col-span-3` and `lg:col-span-9` on desktop
- [ ] Add spacing between sections: `space-y-6`
- [ ] **Self-Check**: Page renders without errors
- [ ] **Self-Check**: Layout is responsive

**Estimated Time**: 1 hour

---

### FN/FE/UI/004-18: Add Toast Notifications

**Goal**: Set up Sonner toast system

- [ ] Edit `/web/src/app/layout.tsx`
- [ ] Import `Toaster` from `@/components/ui/sonner`
- [ ] Add `<Toaster />` component inside body (after children)
- [ ] Set toast position: `position="top-center"`
- [ ] Verify toast styles match app theme
- [ ] Test all toast scenarios:
  - [ ] Success: Staff added, Constraint added, Roster generated
  - [ ] Error: Duplicate employeeId, Generation failed
  - [ ] Info: No staff to generate roster
- [ ] **Self-Check**: Toasts appear and dismiss correctly
- [ ] **Self-Check**: Multiple toasts stack properly

**Estimated Time**: 30 minutes

---

### FN/FE/UI/004-19: End-to-End Manual Testing

**Goal**: Validate complete workflow

- [ ] Start Python engine: `cd engine && uv run uvicorn src.main:app --reload`
- [ ] Start Next.js: `cd web && npm run dev`
- [ ] Run seed: `npx prisma db seed` (if needed)
- [ ] Open browser: `http://localhost:3000/dashboard`
- [ ] **Test Staff Management**:
  - [ ] Click "Add Staff"
  - [ ] Fill form: Name="Alice", EmployeeId="EMP001", Email="alice@test.com"
  - [ ] Verify staff appears in list
  - [ ] Add 4 more staff (EMP002-EMP005)
  - [ ] Delete one staff, verify removal
- [ ] **Test Constraint Management**:
  - [ ] Click "Add Rule" → "Day Off (Point)"
  - [ ] Select Alice, Day 0 (Monday), State Off
  - [ ] Name: "Alice Monday Off"
  - [ ] Verify constraint appears in list
  - [ ] Click "Add Rule" → "Min/Max Workers"
  - [ ] Select "All Days", >=, Value: 3, State: Work
  - [ ] Name: "Min 3 Workers Daily"
  - [ ] Verify constraint appears
- [ ] **Test Roster Generation**:
  - [ ] Click "Generate Roster"
  - [ ] Observe SOLVING state (skeleton loader)
  - [ ] Wait for completion (~2-5 seconds)
  - [ ] Verify grid appears with staff × days
  - [ ] Check Alice is Off on Day 0
  - [ ] Check each day has ≥3 workers
- [ ] **Test Error Scenarios**:
  - [ ] Delete all staff, try to generate → Error toast
  - [ ] Stop Python engine, try to generate → Failed status
  - [ ] Add conflicting constraints → Infeasible status
- [ ] **Test Responsive Design**:
  - [ ] Resize browser to tablet size (768px)
  - [ ] Verify layout stacks vertically
  - [ ] Check grid horizontal scroll works
- [ ] **Self-Check**: All scenarios pass
- [ ] **Self-Check**: No console errors or warnings

**Estimated Time**: 2 hours

---

### FN/FE/UI/004-20: Accessibility Audit

**Goal**: Ensure dashboard is accessible

- [ ] Install axe DevTools browser extension
- [ ] Run accessibility scan on dashboard page
- [ ] Fix any critical violations:
  - [ ] Missing alt text on icons
  - [ ] Missing ARIA labels on buttons
  - [ ] Color contrast issues
  - [ ] Missing form labels
- [ ] Test keyboard navigation:
  - [ ] Tab through all interactive elements
  - [ ] Open/close dialogs with Enter/Escape
  - [ ] Navigate dropdowns with arrow keys
- [ ] Test with screen reader (VoiceOver on Mac):
  - [ ] Verify table headers are announced
  - [ ] Verify form labels are read correctly
  - [ ] Verify toast messages are announced
- [ ] Add `aria-live="polite"` to roster status region
- [ ] Add `aria-busy="true"` during SOLVING state
- [ ] **Self-Check**: 0 critical accessibility violations
- [ ] **Self-Check**: All features usable with keyboard only

**Estimated Time**: 1.5 hours

---

### FN/FE/UI/004-21: Code Quality and Documentation

**Goal**: Polish code and add documentation

- [ ] Run TypeScript type check: `npm run type-check`
- [ ] Fix any TypeScript errors
- [ ] Run linter: `npm run lint`
- [ ] Fix linting warnings
- [ ] Add JSDoc comments to:
  - [ ] All hook functions
  - [ ] All Server Actions
  - [ ] Complex component logic
- [ ] Update `/web/README.md`:
  - [ ] Add "Dashboard Features" section
  - [ ] Document staff/constraint workflows
  - [ ] Add screenshot or ASCII diagram
- [ ] Update `.env.example` (if needed)
- [ ] Commit all changes with clear message
- [ ] **Self-Check**: `npm run build` succeeds
- [ ] **Self-Check**: Documentation is clear and complete

**Estimated Time**: 1 hour

---

## Definition of Done

- [ ] All 21 subtasks completed
- [ ] Dashboard page accessible at `/dashboard`
- [ ] Staff CRUD operations work without page refresh
- [ ] Constraint creation works for both types
- [ ] Roster generation displays grid with color-coded states
- [ ] Polling updates status automatically during solving
- [ ] Error states (FAILED, INFEASIBLE) display clearly
- [ ] Responsive design works on tablet+ screens
- [ ] Toast notifications work for all actions
- [ ] Accessibility audit passes with 0 critical issues
- [ ] TypeScript build completes with no errors
- [ ] Manual testing checklist 100% complete

## Implementation Summary

**Expected Deliverables**:
1. ✅ 8 Shadcn UI components installed
2. ✅ Feature folder structure with 11 files
3. ✅ 3 Server Actions files (staff, constraints, roster)
4. ✅ 3 React Query hook files with 9 total hooks
5. ✅ 7 React components (StaffList, AddStaffDialog, ConstraintBuilder, AddConstraintForm, RosterControls, RosterGrid, StateBadge)
6. ✅ 1 dashboard page at `/dashboard`
7. ✅ Toast notification system integrated
8. ✅ Comprehensive manual testing completed
9. ✅ Accessibility audit passed

**Key Features**:
- Real-time staff and constraint management
- Dynamic constraint forms (point vs. vertical sum)
- Automated roster generation with visual feedback
- Color-coded schedule grid (Off=grey, Work=blue, OnCall=purple)
- Status polling during solver execution
- Optimistic UI updates with rollback on errors
- Responsive layout (desktop: 3-9 split, tablet: stacked)
- Toast notifications for all user actions
- Keyboard-accessible interface

**Files Modified/Created**: ~25 files
- 9 new component files
- 3 hook files
- 1 service file
- 3 Server Action files
- 1 page file
- 8 UI component additions
- 1 layout update (Toaster)

---

## Next Steps After Completion

Once this feature is complete, you can:
1. **NLP Constraint Input** (FN/BE/NLP/xxx): Replace manual forms with text parsing
2. **Schedule Editor** (FN/FE/SCD/xxx): Allow drag-and-drop manual overrides
3. **Advanced Exports** (FN/FE/EXP/xxx): Add PDF, iCal, Google Calendar exports
4. **Staff Preferences** (FN/ADM/STF/xxx): Import availability and shift preferences
5. **Multi-Tenant Auth** (FN/BE/AUTH/xxx): Add organizations and role-based access

---

**Estimated Total Time**: ~32 hours (~4 days)
