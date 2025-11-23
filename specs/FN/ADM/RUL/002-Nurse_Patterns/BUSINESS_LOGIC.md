# FN/ADM/RUL/002 - Nurse Patterns Business Logic

**Feature Type:** Business Rules / Constraint Logic  
**Status:** Implementation  
**Priority:** High  
**Dependencies:** FN/BE/ENG/001-Core_Universal_Solver  

---

## 1. Overview

This document defines the business logic for advanced nurse rostering patterns including horizontal (consecutive) constraints and sliding window patterns. These rules enable complex scheduling requirements common in healthcare environments.

---

## 2. Constraint Types

### 2.1 Existing Constraints (Baseline)

#### A. Point Constraint
**Purpose:** Force a specific resource to a specific state at a specific time slot

**Business Rules:**
- **Hard Constraint:** Must be satisfied (solver fails if impossible)
- **Single Cell:** Affects only one cell in the schedule matrix
- **Priority:** Always enforced before soft constraints

**Use Cases:**
- Scheduled time off (vacation, medical appointments)
- Mandatory shifts (seniority, contractual obligations)
- Training days (must attend/must not work)

**Examples:**
- "Amy must have Monday off" (Requested PTO)
- "Bob must work Friday morning" (Coverage agreement)
- "Carol cannot work December 25th" (Holiday request)

---

#### B. Vertical Sum Constraint
**Purpose:** Count how many resources are in a target state at specific time slot(s)

**Business Rules:**
- **Coverage Requirements:** Ensure minimum staffing levels
- **Overtime Prevention:** Limit maximum staff per shift
- **Fairness:** Equal distribution of shifts (when applied across time)

**Use Cases:**
- Minimum coverage requirements (e.g., "At least 3 nurses on every shift")
- Maximum capacity (e.g., "No more than 5 staff on night shift")
- Exact staffing (e.g., "Exactly 2 nurses on-call")

**Examples:**
- "At least 3 nurses working every day" (Safety requirement)
- "Maximum 5 nurses on night shift" (Budget constraint)
- "Exactly 2 on-call nurses on weekends" (Operational policy)

---

### 2.2 New Constraints (This Implementation)

#### C. Horizontal Sum Constraint
**Purpose:** Count consecutive occurrences of a state for a single resource across time

**Business Rules:**
- **Fatigue Prevention:** Limit consecutive work days to prevent burnout
- **Recovery Time:** Ensure adequate rest between shift blocks
- **Pattern Detection:** Identify and constrain work sequences

**JSON Structure:**
```json
{
  "type": "horizontal_sum",
  "resource": "STAFF_ID",
  "time_slots": [0, 1, 2, 3, 4, 5, 6],
  "target_state": 1,
  "operator": "<=",
  "value": 3
}
```

**Parameters:**
- `resource` (string): Staff member ID (e.g., "EMP001", "Amy")
- `time_slots` (array): Consecutive time slots to evaluate (e.g., [0,1,2,3,4,5,6] for full week)
- `target_state` (int): State to count (0=Off, 1=Work, 2=OnCall)
- `operator` (string): Comparison operator ("<=", ">=", "==")
- `value` (int): Threshold for consecutive occurrences

**Use Cases:**
1. **Max Consecutive Work Days**
   - Example: "Bob cannot work more than 3 consecutive days"
   - JSON: `{"operator": "<=", "value": 3, "target_state": 1}`
   - Rationale: Prevent fatigue and maintain work-life balance

2. **Max Consecutive Night Shifts**
   - Example: "No nurse can work more than 2 consecutive night shifts"
   - JSON: `{"operator": "<=", "value": 2, "target_state": 1}`
   - Rationale: Night work is physically demanding, requires more recovery

3. **Min Consecutive Days Off**
   - Example: "Staff must have at least 2 consecutive days off per week"
   - JSON: `{"operator": ">=", "value": 2, "target_state": 0}`
   - Rationale: Single day off insufficient for recovery

4. **No Isolated Work Days**
   - Example: "Avoid single work days surrounded by days off"
   - JSON: Complex - requires multiple constraints or sliding window
   - Rationale: Inefficient commute, poor work pattern

**OR-Tools Implementation:**
```python
# Pseudocode for max 3 consecutive work days
for start in range(len(time_slots) - 3):
    window = time_slots[start:start+4]  # 4-day window
    work_vars = [shifts[(resource, t)] == target_state for t in window]
    model.Add(sum(work_vars) <= 3)
```

**Edge Cases:**
- **Week Boundaries:** Does "consecutive" span across roster periods?
  - **Decision:** Treat each roster period independently (no cross-period constraints)
- **Partial Windows:** What if roster is 5 days but constraint checks 7-day window?
  - **Decision:** Apply constraint only to available time slots
- **Multiple States:** Can constraint check for "not working" (state 0)?
  - **Decision:** Yes, use `target_state: 0` for consecutive days off

---

#### D. Sliding Window Constraint
**Purpose:** Enforce work-rest patterns with guaranteed recovery periods

**Business Rules:**
- **Fixed Patterns:** "Work X days, then rest Y days" cycles
- **Mandatory Rest:** Automatically enforce recovery after work blocks
- **Shift Rotation:** Support common nursing schedules (e.g., 5-on-2-off, 4-on-3-off)

**JSON Structure:**
```json
{
  "type": "sliding_window",
  "resource": "STAFF_ID",
  "work_days": 5,
  "rest_days": 2,
  "target_state": 1
}
```

**Parameters:**
- `resource` (string): Staff member ID
- `work_days` (int): Number of consecutive work days in pattern
- `rest_days` (int): Number of required rest days after work block
- `target_state` (int): State representing "work" (typically 1)

**Use Cases:**
1. **Standard 5-2 Pattern**
   - Example: "Alice works 5 days, then has 2 days off"
   - JSON: `{"work_days": 5, "rest_days": 2, "target_state": 1}`
   - Rationale: Most common full-time nursing schedule

2. **Intensive Care 3-4 Pattern**
   - Example: "ICU nurses work 3 consecutive 12-hour shifts, then 4 days off"
   - JSON: `{"work_days": 3, "rest_days": 4, "target_state": 1}`
   - Rationale: High-stress environment requires extended recovery

3. **Part-Time 2-5 Pattern**
   - Example: "Part-time staff work 2 days, then 5 days off"
   - JSON: `{"work_days": 2, "rest_days": 5, "target_state": 1}`
   - Rationale: Accommodate staff with other commitments

4. **No Pattern (Pure Max Consecutive)**
   - Example: "Max 4 consecutive days, but no fixed pattern"
   - JSON: Use horizontal_sum instead with `{"operator": "<=", "value": 4}`
   - Rationale: More flexible when fixed patterns not required

**OR-Tools Implementation:**
```python
# Pseudocode for 5-on-2-off pattern
for start in range(len(time_slots) - (work_days + rest_days) + 1):
    work_window = time_slots[start:start+work_days]
    rest_window = time_slots[start+work_days:start+work_days+rest_days]
    
    # If all work_days are worked, then all rest_days must be off
    all_worked = [shifts[(resource, t)] == target_state for t in work_window]
    must_rest = [shifts[(resource, t)] == 0 for t in rest_window]
    
    # Implication: (all_worked) → (must_rest)
    model.AddImplication(
        model.all(all_worked),
        model.all(must_rest)
    )
```

**Edge Cases:**
- **Incomplete Cycles:** Roster ends mid-pattern (e.g., 5 days worked, only 1 rest day at end)
  - **Decision:** Constraint relaxed at boundaries (last cycle doesn't require full rest)
- **Overlapping Patterns:** Staff has multiple sliding window constraints
  - **Decision:** All must be satisfied (may make roster infeasible)
- **Variable States:** Does "rest" mean Off (0) or can include OnCall (2)?
  - **Decision:** Rest = Off (0) only. OnCall is not rest.

---

## 3. Constraint Interactions

### 3.1 Priority Order
When multiple constraints conflict, enforcement order:
1. **Point Constraints** (highest priority - explicit requests)
2. **Vertical Sum Constraints** (coverage requirements - business critical)
3. **Sliding Window Constraints** (pattern enforcement - important for wellness)
4. **Horizontal Sum Constraints** (soft limits - flexible if needed)

### 3.2 Conflict Resolution

**Scenario A: Point vs Sliding Window**
- Point: "Amy must work Monday"
- Sliding Window: "Amy has 2 days off after working 5 days"
- **Resolution:** Point wins. Sliding window may be violated if necessary.

**Scenario B: Vertical Sum vs Horizontal Sum**
- Vertical: "At least 3 nurses every day"
- Horizontal: "Bob max 3 consecutive days"
- **Resolution:** Vertical sum has priority. Bob may work 4+ days if coverage critical.

**Scenario C: Multiple Sliding Windows**
- Window 1: "Alice 5-on-2-off"
- Window 2: "Alice 3-on-4-off"
- **Resolution:** Solver attempts to satisfy both. If impossible, roster marked INFEASIBLE.

### 3.3 Soft Constraints (Future)
Currently all constraints are **hard** (must be satisfied). Future enhancement:
- Add `priority` field (1-10, higher = more important)
- Solver attempts to maximize satisfied constraints weighted by priority
- Allows "preferences" vs "requirements"

---

## 4. Healthcare Industry Standards

### 4.1 Regulatory Requirements

**European Working Time Directive (EWTD):**
- Max 48 hours per week averaged over 17 weeks
- Min 11 consecutive hours rest per day
- Min 24 hours rest per week
- **Implementation:** Use horizontal_sum and sliding_window to enforce

**US Fair Labor Standards Act (FLSA):**
- No federal limit on consecutive days
- Overtime pay after 40 hours/week
- State-specific regulations vary
- **Implementation:** Track hours via shift duration metadata

**UK NHS Guidelines:**
- Max 4 consecutive night shifts
- Min 46 hours rest after night shift block
- No more than 7 consecutive days
- **Implementation:** Use horizontal_sum for nights, sliding_window for rest

### 4.2 Common Nursing Patterns

| Pattern | Work Days | Rest Days | Use Case |
|---------|-----------|-----------|----------|
| Standard | 5 | 2 | Full-time, day shift |
| 12-Hour | 3 | 4 | ICU, ER (long shifts) |
| Continental | 7 | 7 | Extended coverage, rural |
| Part-Time | 2-3 | 4-5 | Flexible staff |
| Night Rotation | 3 | 7 | Specialized night staff |

---

## 5. Pattern Detection Algorithms

### 5.1 Consecutive Counting

**Algorithm:** Sliding Window Maximum
```python
def count_consecutive(schedule: List[int], target_state: int) -> int:
    """
    Count maximum consecutive occurrences of target_state
    
    Examples:
    - [1,1,1,0,1,1] → max_consecutive(1) = 3
    - [0,1,0,1,0,1] → max_consecutive(1) = 1
    """
    max_count = 0
    current_count = 0
    
    for state in schedule:
        if state == target_state:
            current_count += 1
            max_count = max(max_count, current_count)
        else:
            current_count = 0
    
    return max_count
```

### 5.2 Pattern Matching

**Algorithm:** KMP String Matching (adapted for state sequences)
```python
def find_pattern_violations(
    schedule: List[int], 
    work_pattern: int, 
    rest_pattern: int
) -> List[int]:
    """
    Find locations where work-rest pattern is violated
    
    Examples:
    - schedule: [1,1,1,1,1,0,0,1,1,1,1,1,1,0]
    - work_pattern: 5, rest_pattern: 2
    - violations: [index 7] (6 consecutive work days)
    """
    violations = []
    
    for i in range(len(schedule) - work_pattern):
        work_window = schedule[i:i+work_pattern]
        
        # Check if all work_days are worked
        if all(s == 1 for s in work_window):
            # Check rest period
            rest_start = i + work_pattern
            rest_end = rest_start + rest_pattern
            
            if rest_end <= len(schedule):
                rest_window = schedule[rest_start:rest_end]
                if not all(s == 0 for s in rest_window):
                    violations.append(i)
    
    return violations
```

---

## 6. Validation Rules

### 6.1 Pre-Solve Validation

Before sending to OR-Tools solver:

1. **Type Validation**
   - All `type` fields must be: "point", "vertical_sum", "horizontal_sum", "sliding_window"
   - Unknown types rejected

2. **Parameter Validation**
   - `time_slots` must be within roster range [0, timeSlots-1]
   - `resource` must exist in active staff list
   - `state` must be 0, 1, or 2
   - `operator` must be "<=", ">=", or "=="
   - `value` must be positive integer

3. **Logical Validation**
   - `work_days + rest_days` ≤ total roster time slots
   - Horizontal sum `value` ≤ length of `time_slots`
   - No contradictory point constraints (same resource/time, different states)

4. **Feasibility Check (Optional)**
   - Warn if constraints likely make roster infeasible
   - Example: "All staff have max 2 consecutive days, but 10 days to cover"
   - Allow user to proceed or revise

### 6.2 Post-Solve Validation

After OR-Tools generates roster:

1. **Constraint Verification**
   - Iterate through all constraints
   - Verify each is satisfied in solution
   - Log any violations (should be 0 for OPTIMAL solution)

2. **Quality Metrics**
   - Calculate fairness score (variance in total work days per staff)
   - Count constraint violations (if using soft constraints)
   - Generate report for review

---

## 7. Error Messages

### 7.1 User-Facing Messages

**Infeasible Roster:**
```
Unable to generate roster. The following constraints conflict:
- "Amy must work Monday" (Point Constraint)
- "Amy has Monday off requested" (Point Constraint)

Suggestion: Remove one of the conflicting constraints.
```

**Invalid Constraint:**
```
Invalid constraint: "Bob max 3 consecutive days"
Error: time_slots [0-10] exceeds roster length (7 days)

Suggestion: Adjust time_slots to [0-6]
```

**Resource Not Found:**
```
Invalid constraint: "Charlie needs 2 days off after 5 work days"
Error: Staff member "Charlie" not found

Suggestion: Check staff list. Available staff: Amy, Bob, Alice
```

### 7.2 Developer Messages

**OR-Tools Solver Error:**
```
Solver Status: INFEASIBLE
Constraints Applied: 15
- Point: 3
- Vertical Sum: 4
- Horizontal Sum: 5
- Sliding Window: 3

Debug: Enable solver logging with model.EnableOutput() to see conflict.
```

---

## 8. Implementation Checklist

### Backend (Python Engine)

- [ ] Add `HorizontalSumConstraint` Pydantic model to `schemas.py`
- [ ] Add `SlidingWindowConstraint` Pydantic model to `schemas.py`
- [ ] Implement `apply_horizontal_sum_constraint()` in `solver.py`
- [ ] Implement `apply_sliding_window_constraint()` in `solver.py`
- [ ] Update `solve()` method to handle new constraint types
- [ ] Add unit tests for consecutive counting logic
- [ ] Add integration tests for pattern enforcement

### Backend (Next.js)

- [ ] Update Prisma schema with new constraint types
- [ ] Extend Zod schemas in `solver.ts` validation
- [ ] Update `solver-integration.service.ts` to pass new types
- [ ] Add constraint preview in UI before saving

### Frontend

- [ ] Create UI for horizontal_sum constraints (manual form)
- [ ] Create UI for sliding_window constraints (manual form)
- [ ] Add AI translation for pattern descriptions
- [ ] Display constraint violations in roster view

### Testing

- [ ] Test Case 1: "Max 3 consecutive work days" across 7-day roster
- [ ] Test Case 2: "5-on-2-off pattern" for full-time staff
- [ ] Test Case 3: Multiple overlapping patterns (infeasible)
- [ ] Test Case 4: Mixed constraints (point + horizontal + sliding window)

---

## 9. Performance Considerations

### 9.1 Solver Complexity

**Big O Analysis:**
- Point Constraint: O(1) - direct variable assignment
- Vertical Sum: O(n) where n = number of resources
- Horizontal Sum: O(m) where m = number of time slots
- Sliding Window: O(m * w) where w = window size

**Scalability:**
- 10 staff × 30 days × 4 constraint types = ~1200 constraint applications
- OR-Tools can handle this efficiently (< 5 seconds solve time)
- Performance degrades with >50 staff or >60 days

### 9.2 Optimization Strategies

1. **Constraint Pruning**
   - Remove redundant constraints before solving
   - Example: If horizontal_sum already enforces max 3 consecutive, don't add overlapping sliding_window

2. **Incremental Solving**
   - For large rosters (>30 days), solve in chunks
   - Week 1 solution becomes constraints for Week 2

3. **Caching**
   - Cache constraint parsing results
   - Reuse solver models for similar requests

---

## 10. Future Enhancements

### 10.1 Advanced Patterns

**Weekend Rotation:**
- "Each staff member works at most 2 weekends per month"
- Implementation: Custom constraint counting weekend slots

**Night-to-Day Transition Prevention:**
- "No day shift immediately after night shift"
- Implementation: Point constraint for t+1 if t is night shift

**Fairness Constraints:**
- "Distribute shifts evenly among staff"
- Implementation: Minimize variance in total work days

**Preference Constraints (Soft):**
- "Alice prefers morning shifts (priority 7)"
- Implementation: Maximize weighted preference satisfaction

### 10.2 AI-Enhanced Pattern Detection

**Historical Analysis:**
- Analyze past rosters to identify common patterns
- Suggest constraints based on observed schedules

**Anomaly Detection:**
- Flag unusual scheduling (e.g., "Bob worked 10 consecutive days last month")
- Recommend corrective constraints

**Predictive Scheduling:**
- Learn staff preferences from accepted/rejected rosters
- Auto-generate constraints for next period

---

## 11. Appendix

### A. Constraint Type Decision Tree

```
Is the constraint about a single cell?
├─ YES → Use Point Constraint
└─ NO ↓

Is the constraint about coverage (multiple staff at same time)?
├─ YES → Use Vertical Sum Constraint
└─ NO ↓

Is the constraint about consecutive occurrences?
├─ YES ↓
│   Is it a fixed work-rest pattern?
│   ├─ YES → Use Sliding Window Constraint
│   └─ NO → Use Horizontal Sum Constraint
└─ NO → Unsupported (request new constraint type)
```

### B. Example Constraint Combinations

**Scenario: Full-Time Nurse with Balanced Schedule**
```json
[
  {
    "type": "horizontal_sum",
    "resource": "Amy",
    "time_slots": [0,1,2,3,4,5,6],
    "target_state": 1,
    "operator": "<=",
    "value": 5
  },
  {
    "type": "horizontal_sum",
    "resource": "Amy",
    "time_slots": [0,1,2,3,4,5,6],
    "target_state": 0,
    "operator": ">=",
    "value": 2
  },
  {
    "type": "point",
    "resource": "Amy",
    "time_slot": 6,
    "state": 0
  }
]
```
**Result:** Amy works max 5 days, has at least 2 consecutive days off, and Sunday off guaranteed.

---

**Document Version:** 1.0  
**Last Updated:** November 21, 2025  
**Author:** Business Analyst  
**Status:** Implementation Ready
