# FN/ADM/RUL/004 - Fairness & Soft Constraints - Task Breakdown

**Feature:** FN/ADM/RUL/004 - Fairness & Soft Constraints  
**Estimated Total:** 8-10 hours  
**Dependencies:** FN/BE/ENG/001, FN/ADM/RUL/003

---

## Task List

### FN/ADM/RUL/004-01: Schema & Migration (1h)

**Description:** Add `isRequired` field to Constraint model in Prisma schema

**Subtasks:**
- [ ] Add `isRequired Boolean @default(true) @map("is_required")` to Constraint model in `schema.prisma`
- [ ] Run `npx prisma migrate dev --name add_is_required_to_constraint`
- [ ] Verify migration applied successfully
- [ ] Regenerate Prisma client

**Files:**
- `prisma/schema.prisma`

**Acceptance Criteria:**
- [ ] New `is_required` column exists in `constraint` table
- [ ] Default value is `true` for existing constraints
- [ ] Prisma types include `isRequired` property

---

### FN/ADM/RUL/004-02: Seed Data Update (1h)

**Description:** Update seed.ts to include `isRequired` for all constraints and add fairness_weights config

**Subtasks:**
- [ ] Add `isRequired: true` to all coverage constraints (Day/Night Coverage)
- [ ] Add `isRequired: true` to all attribute constraints (IC/Female Coverage)
- [ ] Add `isRequired: true` to all pattern_block constraints
- [ ] Change `night_distribution` system config to generate soft constraints
- [ ] Add `fairness_weights` system config item with weekday-only weights (Mon-Fri = 1, Sat/Sun = 0)
- [ ] Run seed and verify data

**Files:**
- `prisma/seed.ts`

**Acceptance Criteria:**
- [ ] All coverage/safety constraints have `isRequired: true`
- [ ] Fairness constraints identifiable for soft constraint treatment
- [ ] `fairness_weights` config excludes weekends from calculations

---

### FN/ADM/RUL/004-03: Python Schema Update (1h)

**Description:** Add `is_required` field to all constraint Pydantic models

**Subtasks:**
- [ ] Add `is_required: bool = True` to `PointConstraint`
- [ ] Add `is_required: bool = True` to `VerticalSumConstraint`
- [ ] Add `is_required: bool = True` to `HorizontalSumConstraint`
- [ ] Add `is_required: bool = True` to `SlidingWindowConstraint`
- [ ] Add `is_required: bool = True` to `PatternBlockConstraint`
- [ ] Add `is_required: bool = True` to `AttributeVerticalSumConstraint`
- [ ] Add `is_required: bool = True` to `ResourceStateCountConstraint`
- [ ] Add `is_required: bool = True` to `CompoundAttributeVerticalSumConstraint`
- [ ] Run Python tests to ensure no regressions

**Files:**
- `orto-ai-engine/src/core/schemas.py`

**Acceptance Criteria:**
- [ ] All constraint models accept `is_required` field
- [ ] Default is `True` (backward compatible)
- [ ] Existing tests pass

---

### FN/ADM/RUL/004-04: Solver Soft Constraint Implementation (2-3h)

**Description:** Implement soft constraint handling with slack variables and penalty minimization

**Subtasks:**
- [ ] Add `self.penalties = []` to SolverService `__init__`
- [ ] Add `self.soft_violations = {}` to track violations per constraint
- [ ] Modify `_apply_constraint()` to route soft constraints to separate handlers
- [ ] Implement `_apply_resource_state_count_soft()` with slack variables
- [ ] Implement `_apply_vertical_sum_soft()` with slack variables
- [ ] Add `model.Minimize(sum(self.penalties))` before `solver.Solve()`
- [ ] Include violation details in SolveResponse
- [ ] Write unit tests for soft constraint handling

**Files:**
- `orto-ai-engine/src/services/solver.py`
- `orto-ai-engine/src/core/schemas.py` (SolveResponse update)
- `orto-ai-engine/src/tests/test_solver.py`

**Acceptance Criteria:**
- [ ] Soft constraints allow violations with penalties
- [ ] Hard constraints remain strictly enforced
- [ ] Solver minimizes total penalty score
- [ ] Response includes penalty details

---

### FN/ADM/RUL/004-05: TypeScript Integration (1h)

**Description:** Pass `is_required` field from database to Python solver

**Subtasks:**
- [ ] Update `buildSolverRequest()` to include `is_required: c.isRequired ?? true` for user constraints
- [ ] Update system constraint mapping to include `is_required` based on constraint type
- [ ] Add TypeScript types for `is_required` field in SolverConstraint type
- [ ] Test with mixed hard/soft constraints

**Files:**
- `src/services/solver-integration.service.ts`
- `src/types/solver.ts` (if exists)

**Acceptance Criteria:**
- [ ] Solver payload includes `is_required` for all constraints
- [ ] Night distribution constraints passed as soft (`is_required: false`)
- [ ] Coverage constraints passed as hard (`is_required: true`)

---

### FN/ADM/RUL/004-06: UI - Required Column & Badge (1h)

**Description:** Add Required column to constraint table with Hard/Soft badge

**Subtasks:**
- [ ] Add `<TableHead>Required</TableHead>` to table header
- [ ] Add `<Badge>` cell showing "Hard" (default) or "Soft" based on `c.isRequired`
- [ ] Style Hard badge with solid/default variant
- [ ] Style Soft badge with secondary/outline variant
- [ ] Verify display for existing constraints

**Files:**
- `src/features/dashboard/components/constraint-builder.tsx`

**Acceptance Criteria:**
- [ ] Required column visible in constraint table
- [ ] Badge correctly shows Hard/Soft status
- [ ] Visual distinction clear between hard and soft

---

### FN/ADM/RUL/004-07: UI - Edit Dialog Toggle (1h)

**Description:** Add isRequired toggle switch to edit constraint dialog

**Subtasks:**
- [ ] Add `isRequired` to `editFormData` state
- [ ] Initialize `isRequired` from `editingConstraint.isRequired` in `handleEditClick`
- [ ] Add Switch component with Label in edit dialog
- [ ] Update `handleEditSubmit` to include `isRequired` in update payload
- [ ] Update `useUpdateConstraint` hook to accept `isRequired` field
- [ ] Test toggle functionality

**Files:**
- `src/features/dashboard/components/constraint-builder.tsx`
- `src/features/dashboard/hooks/use-constraints.ts`
- `src/app/actions/constraints.ts`

**Acceptance Criteria:**
- [ ] Toggle switch visible in edit dialog
- [ ] Toggle state reflects current constraint setting
- [ ] Changes persist to database on save

---

### FN/ADM/RUL/004-08: Integration Testing (1h)

**Description:** End-to-end testing of soft constraint functionality

**Subtasks:**
- [ ] Generate roster with all hard constraints - verify OPTIMAL/FEASIBLE
- [ ] Modify night_distribution to soft - verify more balanced distribution
- [ ] Test infeasibility with hard constraint violation
- [ ] Test feasibility with soft constraint violation (penalized)
- [ ] Verify UI correctly displays all constraint states

**Files:**
- Manual testing / test scripts

**Acceptance Criteria:**
- [ ] Hard constraints strictly enforced
- [ ] Soft constraints relaxed when necessary
- [ ] Night shift distribution more balanced than before
- [ ] No regressions in existing functionality

---

## Summary

| Task | Estimate | Status |
|------|----------|--------|
| FN/ADM/RUL/004-01 | 1h | ✅ Completed |
| FN/ADM/RUL/004-02 | 1h | ✅ Completed |
| FN/ADM/RUL/004-03 | 1h | ✅ Completed |
| FN/ADM/RUL/004-04 | 2-3h | ✅ Completed |
| FN/ADM/RUL/004-05 | 1h | ✅ Completed |
| FN/ADM/RUL/004-06 | 1h | ✅ Completed |
| FN/ADM/RUL/004-07 | 1h | ✅ Completed |
| FN/ADM/RUL/004-08 | 1h | 🔄 Ready for Testing |
| **Total** | **8-10h** | |

---

## Dependencies Graph

```
004-01 (Schema) ──┬──> 004-02 (Seed)
                  │
                  ├──> 004-03 (Python Schema) ──> 004-04 (Solver)
                  │
                  └──> 004-05 (TS Integration) ──> 004-06 (UI Column)
                                                         │
                                                         └──> 004-07 (UI Toggle)
                                                                    │
                                                                    └──> 004-08 (Testing)
```
