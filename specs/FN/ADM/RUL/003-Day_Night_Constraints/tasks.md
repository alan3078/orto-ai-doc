# Tasks - FN/ADM/RUL/003 - Day/Night Staffing Constraints

---

## JIRA Ticket Breakdown

### Epic: FN/ADM/RUL/003 - Day/Night Staffing Constraints

Implement staffing constraints for Day/Night shift mode including minimum staffing, female IC requirements, pattern blocking, and IC assignment tracking.

---

### FN/ADM/RUL/003-01: Create Documentation

**Story Points:** 1  
**Priority:** High  
**Assignee:** Development Team

**Description:**
Create specification documentation for Day/Night staffing constraints feature.

**Acceptance Criteria:**
- [ ] `plan.md` created with scope and success criteria
- [ ] `implementation.md` created with technical details
- [ ] `tasks.md` created with JIRA breakdown
- [ ] `BUSINESS_LOGIC.md` created with constraint rules

**Files:**
- `orto-ai-doc/specs/FN/ADM/RUL/003-Day_Night_Constraints/plan.md`
- `orto-ai-doc/specs/FN/ADM/RUL/003-Day_Night_Constraints/implementation.md`
- `orto-ai-doc/specs/FN/ADM/RUL/003-Day_Night_Constraints/tasks.md`
- `orto-ai-doc/specs/FN/ADM/RUL/003-Day_Night_Constraints/BUSINESS_LOGIC.md`

---

### FN/ADM/RUL/003-02: Add CompoundAttributeVerticalSumConstraint Schema

**Story Points:** 2  
**Priority:** High  
**Assignee:** Backend Developer

**Description:**
Add new Pydantic model for compound attribute filtering with AND logic.

**Acceptance Criteria:**
- [ ] `CompoundAttributeVerticalSumConstraint` model added to `schemas.py`
- [ ] Model includes `attribute_filters: Dict[str, List[str]]` field
- [ ] Model added to `ConstraintModel` union type
- [ ] Validation logic added in `SolveRequest` validator

**Files:**
- `orto-ai-engine/src/core/schemas.py`

**Code Sample:**
```python
class CompoundAttributeVerticalSumConstraint(BaseModel):
    type: Literal["compound_attribute_vertical_sum"] = "compound_attribute_vertical_sum"
    time_slot: Union[int, Literal["ALL"]]
    target_state: int = Field(..., ge=0)
    operator: Literal[">=", "<=", "=="]
    value: int = Field(..., ge=0)
    attribute_filters: Dict[str, List[str]]
```

---

### FN/ADM/RUL/003-03: Implement Compound Constraint Solver Logic

**Story Points:** 3  
**Priority:** High  
**Assignee:** Backend Developer

**Description:**
Implement solver handler for compound attribute vertical sum constraint.

**Acceptance Criteria:**
- [ ] `_apply_compound_attribute_vertical_sum_constraint()` method added
- [ ] Resources filtered by ALL attribute conditions (AND logic)
- [ ] Vertical sum applied to filtered resources
- [ ] Dispatch added in `_apply_constraint()` method
- [ ] Import added for new constraint type

**Files:**
- `orto-ai-engine/src/services/solver.py`

**Algorithm:**
1. Extract attribute_filters from constraint
2. Filter resources where ALL attributes match (AND)
3. For each time slot, count filtered resources in target_state
4. Apply operator comparison (>=, <=, ==)

---

### FN/ADM/RUL/003-04: Add Compound Constraint Validation

**Story Points:** 2  
**Priority:** High  
**Assignee:** Backend Developer

**Description:**
Implement validation handler for compound attribute vertical sum constraint.

**Acceptance Criteria:**
- [ ] `_validate_compound_attribute_vertical_sum()` method added
- [ ] Validates existing schedules against compound filter requirements
- [ ] Returns detailed violation information
- [ ] Dispatch added in `_validate_constraint()` method
- [ ] Import added for new constraint type

**Files:**
- `orto-ai-engine/src/services/validator.py`

---

### FN/ADM/RUL/003-05: Add TypeScript Zod Schema

**Story Points:** 1  
**Priority:** High  
**Assignee:** Frontend Developer

**Description:**
Add Zod validation schema for compound attribute vertical sum constraint.

**Acceptance Criteria:**
- [ ] `CompoundAttributeVerticalSumConstraintSchema` added
- [ ] Schema includes `attribute_filters` as `z.record(z.string(), z.array(z.string()))`
- [ ] Schema added to `SolverConstraintSchema` union
- [ ] Type exported

**Files:**
- `orto-ai-app/src/lib/validations/solver.ts`

---

### FN/ADM/RUL/003-06: Extend Shift Model with isIC Field

**Story Points:** 2  
**Priority:** Medium  
**Assignee:** Backend Developer

**Description:**
Add `isIC` boolean field to Shift model for IC assignment tracking.

**Acceptance Criteria:**
- [ ] `isIC Boolean @default(false) @map("is_ic")` added to Shift model
- [ ] Prisma migration created and applied
- [ ] Prisma client regenerated

**Files:**
- `orto-ai-app/prisma/schema.prisma`

**Commands:**
```bash
npx prisma migrate dev --name add_shift_is_ic
npx prisma generate
```

---

### FN/ADM/RUL/003-07: Update Roster Grid UI for IC Highlighting

**Story Points:** 2  
**Priority:** Medium  
**Assignee:** Frontend Developer

**Description:**
Apply green background (#e2efda) to roster grid cells where staff is IC.

**Acceptance Criteria:**
- [ ] Import `ROSTER_GRID_COLORS` from constants
- [ ] Apply `backgroundColor: ROSTER_GRID_COLORS.LIGHT_GREEN` when `shift.isIC === true`
- [ ] Visual appears in completed roster grid
- [ ] Legend updated to show IC indicator (optional)

**Files:**
- `orto-ai-app/src/features/dashboard/components/roster-grid.tsx`

---

### FN/ADM/RUL/003-08: Add Day/Night Seed Constraints

**Story Points:** 2  
**Priority:** Medium  
**Assignee:** Backend Developer

**Description:**
Add default constraints for Day/Night shift mode in seed data.

**Acceptance Criteria:**
- [ ] Minimum Day Staff (>=3) constraint added
- [ ] Minimum Night Staff (>=2) constraint added
- [ ] Female IC Day (>=1 compound) constraint added
- [ ] Female IC Night (>=1 compound) constraint added
- [ ] No Night-to-Day pattern block constraint added
- [ ] All constraints have `shiftType: 'DAY_NIGHT'`

**Files:**
- `orto-ai-app/prisma/seed.ts`

---

### FN/ADM/RUL/003-09: Implement Round-Robin IC Assignment (Future)

**Story Points:** 3  
**Priority:** Low  
**Assignee:** TBD

**Description:**
Implement automatic IC assignment using round-robin distribution among eligible female IC staff.

**Acceptance Criteria:**
- [ ] IC assignment logic added to solver integration service
- [ ] Round-robin ensures fair distribution across roster period
- [ ] IC assignments saved with `isIC: true` after roster solve
- [ ] Unit tests verify fair distribution

**Status:** 🔮 Future Implementation

**Notes:**
- Track last IC assignment per staff to ensure rotation
- Consider seniority/rank for tie-breaking
- May need UI for manual override

---

## Task Dependencies

```
003-01 (Docs) ─────────────────────────────────────────┐
                                                        │
003-02 (Schema) ─────┬─────────────────────────────────┤
                     │                                  │
003-03 (Solver) ─────┤                                  │
                     │                                  ▼
003-04 (Validator) ──┴─────────────────────────► 003-08 (Seed)
                                                        │
003-05 (Zod) ──────────────────────────────────────────┤
                                                        │
003-06 (Prisma) ─────┬─────────────────────────────────┤
                     │                                  │
003-07 (UI) ─────────┴─────────────────────────────────┘
                                                        │
                                                        ▼
                                               003-09 (IC Assignment)
                                                  [Future]
```

---

## Testing Checklist

### Unit Tests
- [ ] Compound constraint filters correctly (gender AND role)
- [ ] Solver returns OPTIMAL/FEASIBLE with compound constraints
- [ ] Validator detects compound constraint violations
- [ ] Zod schema validates compound constraint JSON

### Integration Tests
- [ ] Full solve with Day/Night constraints produces valid roster
- [ ] At least 3 staff on each Day shift
- [ ] At least 2 staff on each Night shift
- [ ] At least 1 female IC on each Day and Night shift
- [ ] No Night→Day transitions in output

### UI Tests
- [ ] IC cells display green background
- [ ] Non-IC cells have default background
- [ ] Legend shows IC indicator (if added)

---

## Estimated Total

| Task | Story Points |
|------|--------------|
| 003-01 Documentation | 1 |
| 003-02 Python Schema | 2 |
| 003-03 Solver Logic | 3 |
| 003-04 Validator | 2 |
| 003-05 Zod Schema | 1 |
| 003-06 Prisma Migration | 2 |
| 003-07 UI Highlighting | 2 |
| 003-08 Seed Data | 2 |
| **Total** | **15** |

---

**Document Version:** 1.0  
**Created:** November 27, 2025  
