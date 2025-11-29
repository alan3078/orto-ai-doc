# FN/ADM/RUL/003 - Day/Night Staffing Constraints

**Feature Type:** Business Rules / Constraint Logic  
**Status:** Planning  
**Priority:** High  
**Dependencies:** FN/BE/ENG/001-Core_Universal_Solver, FN/ADM/RUL/002-Nurse_Patterns  

---

## 1. Overview

This feature implements staffing constraints specific to Day/Night shift mode (DAY_NIGHT), including minimum staffing levels, female In-Charge (IC) requirements, pattern blocking, and IC assignment tracking with visual highlighting.

---

## 2. Problem Statement

Hospital ICU/HDU units require:
- Minimum staffing levels per shift (3 for day, 2 for night)
- At least one female IC-eligible staff member acting as In-Charge per shift
- Prevention of dangerous Night→Day transitions (fatigue management)
- Maximum consecutive work day limits
- Visual tracking of who is assigned as IC each shift

---

## 3. Scope

### In Scope
1. **Minimum Staffing Constraints** - Vertical sum constraints for Day (state=1) ≥3 and Night (state=2) ≥2
2. **Compound Attribute Constraint** - New constraint type filtering by multiple attributes (gender + role)
3. **Female IC Requirement** - At least 1 female IC-eligible staff per Day shift and Night shift
4. **IC Assignment Tracking** - Database field to mark which staff is IC on each shift
5. **IC Visual Highlighting** - Green background (#e2efda) for IC assignments in roster grid
6. **Pattern Block Constraint** - Prevent Night→Day transitions
7. **Max Consecutive Work Days** - Limit consecutive work days (horizontal sum)

### Out of Scope (Future)
- Round-robin IC assignment algorithm (tracked for future implementation)
- Manual IC assignment UI
- IC reporting/statistics dashboard

---

## 4. Success Criteria

1. ✅ Solver enforces minimum 3 staff on Day shift, 2 on Night shift
2. ✅ Solver enforces at least 1 female IC-eligible staff on each Day and Night shift
3. ✅ Solver blocks Night→Day transitions
4. ✅ Shift model stores `isIC` boolean flag
5. ✅ Roster grid displays IC assignments with green (#e2efda) background
6. ✅ Seed data includes Day/Night constraints for immediate use

---

## 5. Technical Approach

### 5.1 New Constraint Type: compound_attribute_vertical_sum

Filter resources by multiple attributes (AND logic) before counting:

```json
{
  "type": "compound_attribute_vertical_sum",
  "time_slot": "ALL",
  "target_state": 1,
  "operator": ">=",
  "value": 1,
  "attribute_filters": {
    "gender": ["F"],
    "roles": ["IC"]
  }
}
```

### 5.2 Data Model Extension

Add `isIC` boolean to `Shift` model:

```prisma
model Shift {
  // ... existing fields
  isIC       Boolean  @default(false) @map("is_ic")
}
```

### 5.3 UI Enhancement

Apply conditional styling in roster grid:
- IC shift cells get `backgroundColor: '#e2efda'`

---

## 6. Dependencies

| Component | Dependency | Status |
|-----------|------------|--------|
| Python Engine | OR-Tools CP-SAT | ✅ Installed |
| Schemas | Pydantic models | ✅ Base exists |
| Validation | Zod schemas | ✅ Base exists |
| Database | Prisma ORM | ✅ Connected |
| UI | roster-grid.tsx | ✅ Exists |

---

## 7. Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Compound constraint makes roster INFEASIBLE | High | Ensure sufficient female IC staff in seed data |
| Performance degradation with compound filters | Medium | Pre-filter resources before constraint application |
| Migration breaks existing data | Low | Default `isIC` to false, no data loss |

---

## 8. Timeline

| Phase | Duration | Deliverable |
|-------|----------|-------------|
| Documentation | 1 hour | plan.md, implementation.md, tasks.md, BUSINESS_LOGIC.md |
| Backend (Python) | 2 hours | Compound constraint schema + solver + validator |
| Backend (TypeScript) | 1 hour | Zod schema + Prisma migration |
| Frontend | 1 hour | Roster grid IC highlighting |
| Testing | 1 hour | Integration tests |
| **Total** | **6 hours** | |

---

## 9. References

- [FN/ADM/RUL/002 - Nurse Patterns](../002-Nurse_Patterns/BUSINESS_LOGIC.md)
- [Hospital Roster Sample](attached image - Feb 2025)
- OR-Tools CP-SAT Documentation

---

**Document Version:** 1.0  
**Created:** November 27, 2025  
**Author:** Development Team  
