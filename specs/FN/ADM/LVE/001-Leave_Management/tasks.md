# FN/ADM/LVE/001 - Tasks

## Subtasks

### FN/ADM/LVE/001-01: Define LeaveType enum and Leave Prisma model
- [ ] Add LeaveType enum to schema.prisma
- [ ] Add LeaveStatus enum to schema.prisma
- [ ] Create Leave model with staff relation
- [ ] Add indexes on staffId and date range
- [ ] Create migration
- [ ] Generate Prisma client

### FN/ADM/LVE/001-02: Create PublicHoliday table and seed data
- [ ] Create PublicHoliday model in schema
- [ ] Add seed data for 2025/2026 Hong Kong public holidays
- [ ] Create utility to check if date is PH
- [ ] Add API to fetch PH for date range

### FN/ADM/LVE/001-03: Build leave management UI
- [ ] Create /admin/leaves page
- [ ] Add leave calendar component (month view)
- [ ] Build create leave dialog with:
  - Staff selector
  - Date range picker
  - Leave type dropdown
  - Notes field
- [ ] Add leave list table view
- [ ] Implement edit/delete leave actions

### FN/ADM/LVE/001-04: Integrate leaves as fixed OFF in solver
- [ ] Query leaves for roster date range
- [ ] Generate point constraints for each leave day
- [ ] Map leave to state "OFF" in solver payload
- [ ] Update solver-integration.service.ts
- [ ] Test with existing roster generation

### FN/ADM/LVE/001-05: Display leaves on roster grid
- [ ] Modify RosterGrid cell component
- [ ] Show leave type abbreviation (SL, AL, etc.)
- [ ] Add distinct color/style for leave cells
- [ ] Add tooltip showing leave details
- [ ] Mark cells as non-editable when leave exists
