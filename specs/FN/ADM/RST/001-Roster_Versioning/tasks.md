# Tasks - FN/ADM/RST/001 - Roster Versioning

---

## JIRA Ticket

### FN/ADM/RST/001: Roster Versioning

**Story Points:** 13  
**Priority:** High  
**Assignee:** Development Team

**Description:**
Implement roster versioning allowing up to 3 versions per month with soft-delete support. When generating a new roster with existing versions, show a confirmation modal. Users can view, switch between, and delete versions from the UI.

**Acceptance Criteria:**
- [ ] Prisma schema extended with `version` (Int, default 1) and `deletedAt` (DateTime?) on Roster model
- [ ] Prisma schema extended with `deletedAt` (DateTime?) on Shift model
- [ ] Unique constraint `@@unique([startDate, endDate, shiftType, version])` added to Roster
- [ ] Migration created and applied successfully
- [ ] `getRosterVersionsAction(month)` returns list of non-deleted versions for a month
- [ ] `deleteRosterVersionAction(rosterId)` soft-deletes roster and its shifts (sets deletedAt)
- [ ] `generateRosterAction` no longer hard-deletes existing rosters
- [ ] `generateRosterAction` calculates next available version (1-3)
- [ ] `generateRosterAction` throws error if 3 active versions exist
- [ ] `getRosterAction` accepts optional `version` param, defaults to latest
- [ ] `getRosterAction` filters out soft-deleted rosters and shifts
- [ ] `RosterVersionConfirmModal` component displays when generating with existing versions
- [ ] `RosterVersionSelector` component displays version dropdown with status badges
- [ ] `RosterVersionSelector` includes delete button with confirmation
- [ ] `useRosterVersions(month)` hook fetches versions for a month
- [ ] `useDeleteRosterVersion` mutation handles soft-delete with cache invalidation
- [ ] `useRoster` hook supports optional version param
- [ ] Roster controls UI integrated with version selector and confirmation modal

**Files:**
- `orto-ai-app/prisma/schema.prisma`
- `orto-ai-app/prisma/migrations/YYYYMMDD_roster_versioning/migration.sql`
- `orto-ai-app/src/app/actions/roster.actions.ts`
- `orto-ai-app/src/features/dashboard/components/roster-version-confirm-modal.tsx`
- `orto-ai-app/src/features/dashboard/components/roster-version-selector.tsx`
- `orto-ai-app/src/features/dashboard/hooks/use-roster.ts`
- `orto-ai-app/src/features/dashboard/components/roster-controls.tsx`

---

## Implementation Checklist

### Phase 1: Database Schema
- [ ] Add `version Int @default(1)` to Roster model
- [ ] Add `deletedAt DateTime? @map("deleted_at")` to Roster model
- [ ] Add `deletedAt DateTime? @map("deleted_at")` to Shift model
- [ ] Add `@@unique([startDate, endDate, shiftType, version])` constraint
- [ ] Run `npx prisma migrate dev --name roster_versioning`
- [ ] Regenerate Prisma client

### Phase 2: Server Actions
- [ ] Create `getRosterVersionsAction(month)` function
- [ ] Create `deleteRosterVersionAction(rosterId)` function
- [ ] Modify `generateRosterAction` to remove hard-delete logic
- [ ] Modify `generateRosterAction` to calculate next version (1-3)
- [ ] Modify `generateRosterAction` to throw error at max versions
- [ ] Modify `getRosterAction` to accept optional version param
- [ ] Modify `getRosterAction` to filter deletedAt: null

### Phase 3: Frontend Components
- [ ] Create `RosterVersionConfirmModal` component
- [ ] Create `RosterVersionSelector` component
- [ ] Add delete confirmation within selector

### Phase 4: Hooks
- [ ] Update `rosterKeys` with versions query key
- [ ] Create `useRosterVersions(month)` hook
- [ ] Create `useDeleteRosterVersion` mutation hook
- [ ] Update `useRoster` to accept version param

### Phase 5: Integration
- [ ] Integrate version selector into roster-controls.tsx
- [ ] Wire confirmation modal to generate button
- [ ] Add selected version state management
- [ ] Test full flow: generate, switch, delete versions

---

## Testing Scenarios

1. **First Generation**: Generate roster for month with no existing versions → Creates V1
2. **Second Generation**: Generate with V1 existing → Modal shows, creates V2
3. **Third Generation**: Generate with V1, V2 existing → Modal shows, creates V3
4. **Max Versions**: Generate with V1, V2, V3 existing → Error message shown
5. **Delete Version**: Delete V2 → V2 soft-deleted, V1 and V3 remain
6. **Generate After Delete**: Generate after deleting V2 → Creates new V2
7. **Version Switch**: Switch between V1, V2, V3 → Correct roster displayed
8. **Soft Delete Cascade**: Delete roster → Both roster and shifts have deletedAt set
