# FN/ADM/RUL/003 - Day/Night Constraints Business Logic

**Feature Type:** Business Rules / Constraint Logic  
**Status:** Implementation  
**Priority:** High  
**Dependencies:** FN/BE/ENG/001-Core_Universal_Solver, FN/ADM/RUL/002-Nurse_Patterns  

---

## 1. Overview

This document defines the business logic for Day/Night shift mode (DAY_NIGHT) staffing constraints used in ICU/HDU hospital units. These constraints ensure adequate staffing levels, female In-Charge coverage, and safe work patterns.

---

## 2. Shift Type: DAY_NIGHT

### 2.1 State Definitions

| State | Code | Time Window | Description |
|-------|------|-------------|-------------|
| 0 | O | - | Day Off |
| 1 | 7 | 07:00-19:00 | Day Shift (12 hours) |
| 2 | E | 19:00-07:00 | Night Shift (12 hours) |

### 2.2 State Mapping

```json
{
  "OFF": 0,
  "DAY": 1,
  "NIGHT": 2
}
```

---

## 3. Constraint Types for DAY_NIGHT Mode

### 3.1 Minimum Day Staffing

**Purpose:** Ensure adequate coverage during day shift

**Business Rule:**
- At least **3 staff members** must be assigned to Day shift (state=1) every day
- This is a **hard constraint** - roster is INFEASIBLE if not satisfied

**JSON Structure:**
```json
{
  "type": "vertical_sum",
  "name": "Minimum Day Staff",
  "config": {
    "time_slot": "ALL",
    "target_state": 1,
    "operator": ">=",
    "value": 3
  },
  "shiftType": "DAY_NIGHT",
  "priority": 100
}
```

**Rationale:**
- ICU/HDU requires minimum nursing coverage for patient safety
- 3 staff allows for break coverage and emergency response
- Matches observed pattern from Hong Kong Adventist Hospital roster

---

### 3.2 Minimum Night Staffing

**Purpose:** Ensure adequate coverage during night shift

**Business Rule:**
- At least **2 staff members** must be assigned to Night shift (state=2) every night
- This is a **hard constraint** - roster is INFEASIBLE if not satisfied

**JSON Structure:**
```json
{
  "type": "vertical_sum",
  "name": "Minimum Night Staff",
  "config": {
    "time_slot": "ALL",
    "target_state": 2,
    "operator": ">=",
    "value": 2
  },
  "shiftType": "DAY_NIGHT",
  "priority": 100
}
```

**Rationale:**
- Night shift has reduced patient activity but still requires coverage
- 2 staff minimum ensures one can respond to emergencies while other monitors
- Lower than day due to reduced admissions/procedures at night

---

### 3.3 Female IC Day Shift (Compound Constraint)

**Purpose:** Ensure at least one female In-Charge eligible staff on day shift

**Business Rule:**
- At least **1 staff member** who is both:
  - Female (gender = 'F')
  - IC-eligible (has 'IC' role)
- Must be assigned to Day shift (state=1) every day
- This is a **hard constraint**

**JSON Structure:**
```json
{
  "type": "compound_attribute_vertical_sum",
  "name": "Female IC Day",
  "config": {
    "time_slot": "ALL",
    "target_state": 1,
    "operator": ">=",
    "value": 1,
    "attribute_filters": {
      "gender": ["F"],
      "roles": ["IC"]
    }
  },
  "shiftType": "DAY_NIGHT",
  "priority": 90
}
```

**Rationale:**
- Hospital policy requires female IC for patient care considerations
- IC (In-Charge) role indicates senior/experienced staff capable of leading shift
- Gender-specific requirement may relate to patient comfort or regulatory compliance

---

### 3.4 Female IC Night Shift (Compound Constraint)

**Purpose:** Ensure at least one female In-Charge eligible staff on night shift

**Business Rule:**
- At least **1 staff member** who is both:
  - Female (gender = 'F')
  - IC-eligible (has 'IC' role)
- Must be assigned to Night shift (state=2) every night
- This is a **hard constraint**

**JSON Structure:**
```json
{
  "type": "compound_attribute_vertical_sum",
  "name": "Female IC Night",
  "config": {
    "time_slot": "ALL",
    "target_state": 2,
    "operator": ">=",
    "value": 1,
    "attribute_filters": {
      "gender": ["F"],
      "roles": ["IC"]
    }
  },
  "shiftType": "DAY_NIGHT",
  "priority": 90
}
```

---

### 3.5 No Night-to-Day Transition (Pattern Block)

**Purpose:** Prevent dangerous work patterns that cause fatigue

**Business Rule:**
- A staff member **cannot** work Day shift immediately after Night shift
- Night ends at 07:00, Day starts at 07:00 - no rest period
- This is a **hard constraint**

**JSON Structure:**
```json
{
  "type": "pattern_block",
  "name": "No Night-to-Day",
  "config": {
    "pattern": ["NIGHT", "DAY"],
    "resources": "ALL",
    "state_mapping": {
      "OFF": 0,
      "DAY": 1,
      "NIGHT": 2
    }
  },
  "shiftType": "DAY_NIGHT",
  "priority": 80
}
```

**Rationale:**
- Working day shift after night shift is extremely dangerous
- No recovery time between 12-hour shifts
- Violates most healthcare labor regulations
- Common cause of medical errors and accidents

**Allowed Transitions:**
| From | To | Allowed |
|------|----|---------|
| OFF → DAY | ✅ |
| OFF → NIGHT | ✅ |
| DAY → OFF | ✅ |
| DAY → NIGHT | ✅ (with care) |
| NIGHT → OFF | ✅ |
| NIGHT → DAY | ❌ **BLOCKED** |

---

### 3.6 Maximum Consecutive Work Days (Optional)

**Purpose:** Prevent burnout from extended work periods

**Business Rule:**
- A staff member should not work more than **5 consecutive days**
- Applies to any combination of Day and Night shifts
- This can be a **soft constraint** (advisory) or hard constraint

**JSON Structure:**
```json
{
  "type": "horizontal_sum",
  "name": "Max Consecutive Work",
  "config": {
    "resource": "ALL",
    "time_slots": [0, 1, 2, 3, 4, 5, 6],
    "target_state": 1,
    "operator": "<=",
    "value": 5
  },
  "shiftType": "DAY_NIGHT",
  "priority": 70
}
```

**Note:** This requires per-resource constraints. See implementation for how to apply to all resources.

---

## 4. Compound Attribute Constraint Logic

### 4.1 AND Logic

The `compound_attribute_vertical_sum` constraint uses **AND logic** for multiple attributes:

```
Resource matches IF:
  attribute_1 matches ANY of [values_1]
  AND
  attribute_2 matches ANY of [values_2]
  AND
  ...
```

**Example: Female IC**
```json
{
  "attribute_filters": {
    "gender": ["F"],
    "roles": ["IC"]
  }
}
```

A resource matches if:
- `gender` == 'F' (exact match)
- AND `roles` contains 'IC' (list contains match)

### 4.2 Attribute Types

| Attribute | Type | Example Values |
|-----------|------|----------------|
| gender | string | "F", "M" |
| roles | string[] | ["IC"], ["IC", "RN"] |
| rank | string | "SNO", "SRN", "RN" |

### 4.3 Matching Rules

**String attribute (e.g., gender):**
```python
match = str(resource_attributes[attr]) in attribute_values
```

**List attribute (e.g., roles):**
```python
match = any(str(v) in attribute_values for v in resource_attributes[attr])
```

---

## 5. IC Assignment Logic

### 5.1 IC Eligibility vs IC Assignment

| Concept | Stored In | Meaning |
|---------|-----------|---------|
| IC Eligible | Staff.roles contains 'IC' | Staff CAN act as IC |
| IC Assigned | Shift.isIC = true | Staff IS acting as IC on this shift |

### 5.2 Assignment Rules

1. Only IC-eligible staff can be assigned as IC
2. Exactly **1 IC per shift** (per time slot + state combination)
3. IC assignment is tracked per shift record

### 5.3 Visual Indicator

- IC shifts are highlighted with **#e2efda** (light green) background
- Non-IC shifts have default background

### 5.4 Future: Round-Robin Assignment

When implementing automatic IC assignment:

```
For each (time_slot, state) where state in [1, 2]:  # Day, Night
  eligible = staff where:
    - scheduled for this state at this time_slot
    - has IC role
    - gender == 'F'
  
  Select IC based on:
    1. Least recent IC assignment (round-robin fairness)
    2. Seniority (rank) as tie-breaker
    3. Random if still tied
  
  Mark selected shift as isIC = true
```

---

## 6. Constraint Priority Hierarchy

| Priority | Constraint Type | Reason |
|----------|----------------|--------|
| 100 | Minimum Staffing (Day/Night) | Patient safety - non-negotiable |
| 90 | Female IC Requirements | Policy compliance |
| 80 | Pattern Block (Night→Day) | Safety regulation |
| 70 | Max Consecutive Work | Wellness/advisory |

Higher priority constraints are satisfied first. If constraints conflict, higher priority wins.

---

## 7. Validation Examples

### 7.1 Valid Roster (Day)

| Staff | Gender | IC | Day 1 | Day 2 | Day 3 |
|-------|--------|-----|-------|-------|-------|
| Amy | F | ✅ | **7** | O | **7** |
| Bob | M | ✅ | **7** | **7** | O |
| Carol | F | ❌ | **7** | **7** | **7** |
| Dan | M | ❌ | O | O | **7** |

**Check:** Day 1
- Day staff count: 3 (Amy, Bob, Carol) ✅ >= 3
- Female IC on Day: 1 (Amy) ✅ >= 1

### 7.2 Invalid Roster (Day)

| Staff | Gender | IC | Day 1 |
|-------|--------|-----|-------|
| Amy | F | ✅ | O |
| Bob | M | ✅ | **7** |
| Carol | F | ❌ | **7** |
| Dan | M | ❌ | **7** |

**Check:** Day 1
- Day staff count: 3 (Bob, Carol, Dan) ✅ >= 3
- Female IC on Day: 0 ❌ >= 1 (Amy is off, Carol is not IC)

### 7.3 Invalid Transition

| Staff | Day 1 | Day 2 |
|-------|-------|-------|
| Amy | E (Night) | 7 (Day) |

❌ **INVALID:** Night→Day transition blocked

---

## 8. Edge Cases

### 8.1 Insufficient Female IC Staff

**Problem:** Only 1 female IC in staff pool, needs to cover both Day and Night

**Solution:** 
- Solver returns INFEASIBLE
- Error message: "Cannot satisfy 'Female IC Day' and 'Female IC Night' constraints with available staff"
- Recommendation: Add more female IC-eligible staff

### 8.2 Staff Has Multiple Roles

**Problem:** Staff has roles: ["IC", "Trainer"]

**Solution:**
- List matching: If ANY role matches filter, staff qualifies
- `roles: ["IC"]` filter matches staff with ["IC", "Trainer"]

### 8.3 Gender Not Set

**Problem:** Staff has `gender: null`

**Solution:**
- Staff excluded from gender-filtered constraints
- Warning logged: "Staff X has no gender attribute, excluded from gender filters"

---

## 9. Summary Table: DAY_NIGHT Constraints

| # | Constraint | Type | Target State | Operator | Value | Attributes |
|---|------------|------|--------------|----------|-------|------------|
| 1 | Min Day Staff | vertical_sum | 1 (Day) | >= | 3 | - |
| 2 | Min Night Staff | vertical_sum | 2 (Night) | >= | 2 | - |
| 3 | Female IC Day | compound_attribute_vertical_sum | 1 (Day) | >= | 1 | gender=F, roles=IC |
| 4 | Female IC Night | compound_attribute_vertical_sum | 2 (Night) | >= | 1 | gender=F, roles=IC |
| 5 | No Night→Day | pattern_block | - | - | - | pattern=[NIGHT, DAY] |

---

**Document Version:** 1.0  
**Last Updated:** November 27, 2025  
**Author:** Development Team  
**Status:** Implementation Ready
