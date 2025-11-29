# FN/ADM/REQ/001 - Staff Requests

> **V1.0 Requirement #3**: Able to generate duty according to staff dayoff and no-Night requests

## Overview

Enable staff members to submit schedule preference requests (day off, no night shifts) that managers can approve and incorporate into roster generation.

## Goals

1. Define StaffRequest model
2. Build request submission UI
3. Add "No Night" request type
4. Build manager approval workflow
5. Integrate approved requests as solver constraints

## Scope

### In Scope
- Request types: DAY_OFF, NO_NIGHT, SHIFT_PREFERENCE
- Request model with date, type, status
- Staff self-service submission (Member role)
- Manager approval queue
- Auto-generate point constraints from approved requests
- Request history view

### Out of Scope
- Request quotas/limits
- Automatic approval rules
- Request conflicts detection
- Team request calendar

## Success Criteria

- [ ] Staff can submit day off request for specific date
- [ ] Staff can submit no-night request for date range
- [ ] Manager sees pending requests in queue
- [ ] Manager can approve/reject with comments
- [ ] Approved day-off creates point constraint (OFF)
- [ ] Approved no-night creates pattern block constraint
- [ ] Staff can view own request history

## Dependencies

- FN/ADM/AUTH/001 (User Account Types) - for Member role
- Partially works without auth (Manager creates on behalf)

## Data Model

```prisma
enum RequestType {
  DAY_OFF
  NO_NIGHT
  SHIFT_PREFERENCE
}

enum RequestStatus {
  PENDING
  APPROVED
  REJECTED
}

model StaffRequest {
  id           String        @id @default(cuid())
  staffId      String
  staff        Staff         @relation(fields: [staffId], references: [id])
  requestType  RequestType
  startDate    DateTime
  endDate      DateTime
  preferredShift String?     // For SHIFT_PREFERENCE
  reason       String?
  status       RequestStatus @default(PENDING)
  reviewedBy   String?
  reviewedAt   DateTime?
  reviewNotes  String?
  createdAt    DateTime      @default(now())
  updatedAt    DateTime      @updatedAt
}
```
