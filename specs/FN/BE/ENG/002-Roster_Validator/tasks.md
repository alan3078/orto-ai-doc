# Tasks - FN/BE/ENG/002 - Roster Validator (Audit Mode)

## Task Breakdown

### Backend Tasks (Python Engine)

#### FN/BE/ENG/002-01: Define Validation Schemas
**Estimated Time**: 2 hours  
**Priority**: High  
**Dependencies**: FN/BE/ENG/001 completed

**Acceptance Criteria**:
- [ ] Add `ValidateRequest` schema to `engine/src/core/schemas.py`
  - [ ] Includes `config: ConfigModel`
  - [ ] Includes `schedule: Dict[str, List[int]]`
  - [ ] Includes `constraints: List[ConstraintModel]`
  - [ ] Validator ensures schedule matches config (resources, time_slots, states)
- [ ] Add `ConstraintValidationResult` schema
  - [ ] Fields: `constraint_index`, `constraint_type`, `constraint_name`, `status`, `details`, `violations`
- [ ] Add `ValidateResponse` schema
  - [ ] Fields: `overall_status`, `total_constraints`, `passed_constraints`, `failed_constraints`, `results`, `validation_time_ms`
- [ ] All schemas have proper type hints and field descriptions
- [ ] Schemas validate edge cases (empty schedule, mismatched resources)

---

#### FN/BE/ENG/002-02: Implement Validator Service - Core Logic
**Estimated Time**: 4 hours  
**Priority**: High  
**Dependencies**: FN/BE/ENG/002-01

**Acceptance Criteria**:
- [ ] Create `engine/src/services/validator.py`
- [ ] Implement `ConstraintValidator` class with `__init__(config, schedule)`
- [ ] Implement `validate_constraint()` dispatcher method
- [ ] Implement `_check_operator()` helper (>=, <=, ==)
- [ ] Add proper error handling for unknown constraint types
- [ ] Code is well-commented and follows PEP 8 style guide

---

#### FN/BE/ENG/002-03: Implement Validator Service - Point & Sum Constraints
**Estimated Time**: 3 hours  
**Priority**: High  
**Dependencies**: FN/BE/ENG/002-02

**Acceptance Criteria**:
- [ ] Implement `_validate_point()` method
  - [ ] Checks if `schedule[resource][time_slot] == state`
  - [ ] Returns PASS/FAIL with clear details
- [ ] Implement `_validate_vertical_sum()` method
  - [ ] Handles `time_slot="ALL"` and specific time slots
  - [ ] Counts resources in `target_state` at each time slot
  - [ ] Applies operator comparison (>=, <=, ==)
  - [ ] Reports all failing time slots in violations array
- [ ] Implement `_validate_horizontal_sum()` method
  - [ ] Counts target_state occurrences for resource across time_slots
  - [ ] Applies operator comparison
- [ ] All methods return properly structured `ConstraintValidationResult`

---

#### FN/BE/ENG/002-04: Implement Validator Service - Advanced Constraints
**Estimated Time**: 4 hours  
**Priority**: High  
**Dependencies**: FN/BE/ENG/002-03

**Acceptance Criteria**:
- [ ] Implement `_validate_sliding_window()` method
  - [ ] Checks work-rest patterns across sliding windows
  - [ ] Reports violations with start/end time slots
- [ ] Implement `_validate_pattern_block()` method
  - [ ] Uses `state_mapping` to identify forbidden transitions
  - [ ] Checks all resources if `resources="ALL"`
  - [ ] Reports all forbidden transition occurrences
- [ ] Implement `_validate_attribute_vertical_sum()` method
  - [ ] Filters resources by `attribute` and `attribute_values`
  - [ ] Counts filtered resources in target_state
- [ ] Implement `_validate_resource_state_count()` method
  - [ ] Counts total occurrences of target_state for resource
- [ ] All methods handle edge cases (empty filters, single time slot)

---

#### FN/BE/ENG/002-05: Create Validation API Endpoint
**Estimated Time**: 2 hours  
**Priority**: High  
**Dependencies**: FN/BE/ENG/002-04

**Acceptance Criteria**:
- [ ] Add `POST /api/v1/validate` endpoint to `engine/src/api/v1/solve.py`
- [ ] Endpoint accepts `ValidateRequest` and returns `ValidateResponse`
- [ ] Measures validation time and includes in response
- [ ] Handles exceptions gracefully (returns 500 with error detail)
- [ ] Endpoint has proper docstring with example JSON
- [ ] Endpoint is registered in FastAPI router

---

#### FN/BE/ENG/002-06: Write Backend Unit Tests
**Estimated Time**: 4 hours  
**Priority**: High  
**Dependencies**: FN/BE/ENG/002-05

**Acceptance Criteria**:
- [ ] Create `engine/src/tests/test_validator.py`
- [ ] Test `_validate_point()` - PASS and FAIL cases
- [ ] Test `_validate_vertical_sum()` - ALL and specific time slots
- [ ] Test `_validate_horizontal_sum()` - various operators
- [ ] Test `_validate_sliding_window()` - pattern violations
- [ ] Test `_validate_pattern_block()` - forbidden transitions
- [ ] Test `_validate_attribute_vertical_sum()` - filtered resources
- [ ] Test `_validate_resource_state_count()` - total counts
- [ ] Test edge cases (empty schedule, no constraints, all pass, all fail)
- [ ] All tests pass with `pytest`
- [ ] Code coverage > 90% for validator module

---

### Frontend Tasks (Next.js Web App)

#### FN/BE/ENG/002-07: Create Validation TypeScript Schemas
**Estimated Time**: 1 hour  
**Priority**: High  
**Dependencies**: FN/BE/ENG/002-01

**Acceptance Criteria**:
- [ ] Create `web/src/lib/validations/validator.ts`
- [ ] Define `ConstraintValidationResultSchema` using Zod
- [ ] Define `ValidateResponseSchema` using Zod
- [ ] Export types for TypeScript usage
- [ ] Schemas match Python backend schemas exactly

---

#### FN/BE/ENG/002-08: Extend Solver Integration Service
**Estimated Time**: 3 hours  
**Priority**: High  
**Dependencies**: FN/BE/ENG/002-05, FN/BE/ENG/002-07

**Acceptance Criteria**:
- [ ] Add `validateRoster(rosterId: string)` method to `SolverIntegrationService`
- [ ] Method fetches roster with shifts, constraints, and staff from Prisma
- [ ] Rebuilds schedule matrix from Shift records
- [ ] Fetches resource_attributes (gender, roles) for staff
- [ ] Transforms DB constraints to solver format
- [ ] Calls `callValidationApi()` with request payload
- [ ] Add `callValidationApi()` private method
  - [ ] POSTs to `${engineUrl}/api/v1/validate`
  - [ ] Validates response with `ValidateResponseSchema`
  - [ ] Handles API errors gracefully
- [ ] Method returns typed `ValidateResponse`

---

#### FN/BE/ENG/002-09: Create Validation Server Action
**Estimated Time**: 1 hour  
**Priority**: High  
**Dependencies**: FN/BE/ENG/002-08

**Acceptance Criteria**:
- [ ] Create `web/src/app/actions/validate-roster.action.ts`
- [ ] Implement `validateRosterAction(rosterId: string)`
- [ ] Action instantiates `SolverIntegrationService`
- [ ] Calls `service.validateRoster(rosterId)`
- [ ] Returns `{ success: true, data: ValidateResponse }` on success
- [ ] Returns `{ success: false, error: string, data: null }` on failure
- [ ] Proper error logging with `console.error`

---

#### FN/BE/ENG/002-10: Build ValidationDrawer Component
**Estimated Time**: 4 hours  
**Priority**: High  
**Dependencies**: FN/BE/ENG/002-09

**Acceptance Criteria**:
- [ ] Create `web/src/features/dashboard/components/validation-drawer.tsx`
- [ ] Component uses Shadcn Sheet (side drawer) component
- [ ] Props: `rosterId: string`, optional `children` for trigger button
- [ ] "Run Validation" button calls `validateRosterAction`
- [ ] Shows loading state during validation (spinner, disabled button)
- [ ] Displays error state if validation fails (red alert card)
- [ ] Displays validation summary card:
  - [ ] Overall status badge (PASS=green, FAIL=red)
  - [ ] Total/Passed/Failed constraint counts
  - [ ] Validation time in milliseconds
- [ ] Lists individual constraint results using `ConstraintReportCard`
- [ ] Drawer is scrollable for long constraint lists
- [ ] Drawer is responsive (full width on mobile, max-w-2xl on desktop)

---

#### FN/BE/ENG/002-11: Build ConstraintReportCard Component
**Estimated Time**: 2 hours  
**Priority**: High  
**Dependencies**: FN/BE/ENG/002-10

**Acceptance Criteria**:
- [ ] Component within `validation-drawer.tsx` or separate file
- [ ] Props: `result: ConstraintValidationResult`
- [ ] Displays constraint type badge
- [ ] Shows PASS/FAIL status with appropriate icon (CheckCircle/XCircle)
- [ ] Displays constraint name (if available)
- [ ] Shows details text in muted color
- [ ] Renders violations array (if present):
  - [ ] Shows up to 5 violations in formatted boxes
  - [ ] Shows "...and X more violations" if > 5
  - [ ] Violations displayed as formatted JSON or structured text
- [ ] Card border color changes based on status (green for PASS, red for FAIL)

---

#### FN/BE/ENG/002-12: Integrate Validation Button - Roster Management Page
**Estimated Time**: 1 hour  
**Priority**: Medium  
**Dependencies**: FN/BE/ENG/002-10

**Acceptance Criteria**:
- [ ] Add `ValidationDrawer` import to `web/src/app/(admin)/roster-management/page.tsx`
- [ ] Add "Check Rules" button in page header or near RosterControls
- [ ] Button triggers ValidationDrawer with current roster ID
- [ ] Button is disabled if no roster is loaded
- [ ] Button has proper icon (AlertCircle) and label

---

#### FN/BE/ENG/002-13: Integrate Validation Button - Test Roster Detail Page
**Estimated Time**: 1 hour  
**Priority**: Medium  
**Dependencies**: FN/BE/ENG/002-10

**Acceptance Criteria**:
- [ ] Add `ValidationDrawer` to `web/src/app/test-roster/[id]/page.tsx`
- [ ] Add "Check Rules" button in page header actions
- [ ] Button uses roster ID from `params.id`
- [ ] Drawer appears on right side when triggered

---

#### FN/BE/ENG/002-14: End-to-End Testing
**Estimated Time**: 3 hours  
**Priority**: High  
**Dependencies**: FN/BE/ENG/002-13

**Acceptance Criteria**:
- [ ] Test validation on a passing roster (all constraints met)
  - [ ] Drawer shows green overall status
  - [ ] All constraint cards show PASS badges
  - [ ] No violations displayed
- [ ] Test validation on a failing roster (some violations)
  - [ ] Drawer shows red overall status
  - [ ] Failed constraints show detailed violation info
  - [ ] Violation counts are accurate
- [ ] Test validation with no constraints
  - [ ] Shows "0 constraints" message or appropriate state
- [ ] Test validation with roster that has no shifts
  - [ ] Shows error message
- [ ] Test validation with invalid roster ID
  - [ ] Shows "Roster not found" error
- [ ] Test validation with Python engine down
  - [ ] Shows API error message
- [ ] Test UI responsiveness (mobile, tablet, desktop)
- [ ] Test keyboard navigation and accessibility

---

#### FN/BE/ENG/002-15: Documentation and Cleanup
**Estimated Time**: 2 hours  
**Priority**: Medium  
**Dependencies**: FN/BE/ENG/002-14

**Acceptance Criteria**:
- [ ] Update API documentation (Swagger/OpenAPI) with `/validate` endpoint
- [ ] Add feature documentation to `engine/README.md`
- [ ] Add user guide section with validation workflow screenshots
- [ ] Document validation response format for API consumers
- [ ] Add code comments for complex validation logic
- [ ] Update `Phase.md` to mark feature as completed
- [ ] Update `JIRA_MAP.md` with completion status

---

## Summary

**Total Estimated Time**: 37 hours (4-5 days)

### Task Dependencies Flowchart
```
002-01 (Schemas)
  ↓
002-02 (Validator Core)
  ↓
002-03 (Point & Sum) → 002-04 (Advanced Constraints)
  ↓
002-05 (API Endpoint)
  ↓
002-06 (Backend Tests) ← Can run in parallel with frontend
  
002-07 (TS Schemas) → 002-08 (Service) → 002-09 (Action)
  ↓
002-10 (Drawer) → 002-11 (ReportCard)
  ↓
002-12 (Roster Mgmt) + 002-13 (Test Roster)
  ↓
002-14 (E2E Testing)
  ↓
002-15 (Docs)
```

### Priority Sequence
1. **Critical Path** (Backend): 001 → 002 → 003 → 004 → 005
2. **Critical Path** (Frontend): 007 → 008 → 009 → 010 → 011
3. **Integration**: 012 → 013 → 014
4. **Testing & Docs**: 006, 015

### Recommended Workflow
- **Day 1-2**: Backend schemas + validator service (001-004)
- **Day 3**: Backend API + tests (005-006)
- **Day 4**: Frontend schemas + service + action (007-009)
- **Day 5**: Frontend UI components (010-011)
- **Day 6**: Integration + E2E testing (012-014)
- **Buffer**: Documentation + cleanup (015)
