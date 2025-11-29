# FN/ADM/LVE/001 - Leave Management

> **V1.0 Requirement #2**: Able to identify SL/PH/AL or other leaves and generate appropriate duty

## Overview

Implement leave management system to track sick leave, public holidays, annual leave, and other leave types. Leaves are treated as fixed OFF assignments in the solver.

## Goals

1. Define LeaveType enum and Leave model
2. Create Public Holiday calendar
3. Build leave management UI
4. Integrate leaves into solver as constraints
5. Display leaves on roster grid

## Scope

### In Scope
- Leave types: SL (Sick), PH (Public Holiday), AL (Annual), UL (Unpaid), ML (Maternity), CL (Compassionate)
- Leave model with staff, date range, type, status
- Public holiday table for Hong Kong
- Leave calendar view
- Manager approval workflow for AL
- Auto-apply PH to all staff
- Solver treats leaves as fixed OFF

### Out of Scope
- Leave balance tracking
- Accrual rules
- Half-day leaves
- Leave reports/analytics

## Success Criteria

- [ ] Manager can create leave entries for staff
- [ ] Public holidays auto-populated for roster period
- [ ] Solver respects leaves as fixed OFF
- [ ] Roster grid shows leave type in cell
- [ ] Leave calendar shows all leaves for month

## Dependencies

- None

## Data Model

```prisma
enum LeaveType {
  SL  // Sick Leave
  PH  // Public Holiday
  AL  // Annual Leave
  UL  // Unpaid Leave
  ML  // Maternity Leave
  CL  // Compassionate Leave
}

enum LeaveStatus {
  PENDING
  APPROVED
  REJECTED
}

model Leave {
  id        String      @id @default(cuid())
  staffId   String
  staff     Staff       @relation(fields: [staffId], references: [id])
  startDate DateTime
  endDate   DateTime
  leaveType LeaveType
  status    LeaveStatus @default(APPROVED)
  notes     String?
  createdAt DateTime    @default(now())
  updatedAt DateTime    @updatedAt
}

model PublicHoliday {
  id          String   @id @default(cuid())
  date        DateTime
  name        String
  year        Int
  isRecurring Boolean  @default(false)
}
```
