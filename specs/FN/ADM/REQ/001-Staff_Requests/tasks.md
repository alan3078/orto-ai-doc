# FN/ADM/REQ/001 - Tasks

## Subtasks

### FN/ADM/REQ/001-01: Define StaffRequest Prisma model
- [ ] Add RequestType enum
- [ ] Add RequestStatus enum
- [ ] Create StaffRequest model with staff relation
- [ ] Add indexes on staffId, status, dates
- [ ] Create migration
- [ ] Generate Prisma client

### FN/ADM/REQ/001-02: Build staff request submission UI
- [ ] Create /requests page (for Members)
- [ ] Build request form component with:
  - Request type selector
  - Date/date range picker
  - Reason text area
- [ ] Add submit request action
- [ ] Show pending/history tabs
- [ ] Allow cancel for pending requests

### FN/ADM/REQ/001-03: Add "No Night" request type
- [ ] Handle NO_NIGHT in form (date range instead of single date)
- [ ] Validate date range (max 1 month)
- [ ] Display differently in list view
- [ ] Generate constraint for night shifts only

### FN/ADM/REQ/001-04: Build manager approval workflow
- [ ] Create /admin/requests page
- [ ] Show pending requests queue
- [ ] Display request details with staff info
- [ ] Add approve/reject buttons
- [ ] Add review notes input
- [ ] Update request status on action
- [ ] Send notification to staff (future)

### FN/ADM/REQ/001-05: Integrate approved requests as solver constraints
- [ ] Query approved requests for roster period
- [ ] Generate point constraint for DAY_OFF (state: OFF)
- [ ] Generate pattern_block for NO_NIGHT (forbidden: [N, E])
- [ ] Handle SHIFT_PREFERENCE as soft constraint
- [ ] Update solver-integration.service.ts
- [ ] Test with existing roster generation
