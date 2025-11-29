# FN/ADM/RUL/004 - Fairness & Soft Constraints Implementation

**Feature Type:** Business Rules / Constraint Logic  
**Status:** Planning  
**Priority:** High  
**Dependencies:** FN/BE/ENG/001-Core_Universal_Solver, FN/ADM/RUL/003-Day_Night_Constraints  

---

## 1. Overview

This document defines the implementation for **soft constraints** and **fairness optimization** in the roster scheduling system. The goal is to distinguish between:

- **Hard constraints** (`isRequired: true`) - Must be satisfied; roster is INFEASIBLE if violated
- **Soft constraints** (`isRequired: false`) - Should be satisfied when possible; violations incur penalties

The solver will minimize total penalty to achieve fairer shift distributions across staff.

---

## 2. Database Schema Changes

### 2.1 Constraint Model Update

Add `isRequired` field to the `Constraint` model in `prisma/schema.prisma`:

```prisma
model Constraint {
  id          String     @id @default(cuid())
  name        String
  description String?
  type        String     // point | vertical_sum | horizontal_sum | sliding_window | etc.
  config      Json
  isActive    Boolean    @default(true) @map("is_active")
  isRequired  Boolean    @default(true) @map("is_required")  // NEW: Hard vs Soft
  priority    Int        @default(0)
  shiftType   ShiftType? @map("shift_type")
  rosters     Roster[]
  createdAt   DateTime   @default(now()) @map("created_at")
  updatedAt   DateTime   @updatedAt @map("updated_at")

  @@index([type])
  @@index([isActive])
  @@index([shiftType])
  @@map("constraint")
}
```

### 2.2 Migration

```bash
npx prisma migrate dev --name add_is_required_to_constraint
```

---

## 3. Constraint Classification

### 3.1 Hard Constraints (isRequired: true)

These constraints are non-negotiable for operational safety:

| Constraint Type | Example | Rationale |
|----------------|---------|-----------|
| Coverage | Day Coverage == 4 | Patient safety requires minimum staffing |
| IC Coverage | IC >= 1 per shift | Clinical leadership required |
| Female Coverage | Female >= 1 per shift | Patient care requirements |
| Pattern Blocks | No Night→Day transition | Fatigue prevention (safety) |
| Post-Night Rest | 1 day off after night | Recovery requirement |

### 3.2 Soft Constraints (isRequired: false)

These constraints optimize fairness but can be relaxed:

| Constraint Type | Example | Rationale |
|----------------|---------|-----------|
| Night Distribution | 2-8 nights per person | Fair workload distribution |
| Weekend Distribution | Balance weekend shifts | Quality of life |
| Consecutive Days Off | Prefer 2+ days off together | Rest quality |

---

## 4. Solver Integration

### 4.1 Python Schema Updates

Add `is_required` field to all constraint Pydantic models in `schemas.py`:

```python
class VerticalSumConstraint(BaseModel):
    type: Literal["vertical_sum"]
    time_slot: Union[int, Literal["ALL"]]
    target_state: int
    operator: Literal[">=", "<=", "=="]
    value: int
    is_required: bool = True  # NEW

class ResourceStateCountConstraint(BaseModel):
    type: Literal["resource_state_count"]
    resource: str
    time_slots: List[int]
    target_state: int
    operator: Literal[">=", "<=", "=="]
    value: int
    is_required: bool = True  # NEW

# ... same for all constraint types
```

### 4.2 Solver Logic Updates

In `solver.py`, implement soft constraint handling:

```python
class SolverService:
    def __init__(self):
        self.model = None
        self.shifts = {}
        self.resources = []
        self.time_slots = 0
        self.states = []
        self.resource_attributes = {}
        self.penalties = []  # NEW: Track penalty terms

    def solve(self, request: SolveRequest) -> SolveResponse:
        # Initialize model and variables
        self._initialize_model(request.config)
        
        # Apply all constraints
        for constraint in request.constraints:
            self._apply_constraint(constraint)
        
        # NEW: Add optimization objective if soft constraints exist
        if self.penalties:
            self.model.Minimize(sum(self.penalties))
        
        # Solve the model
        solver = cp_model.CpSolver()
        status = solver.Solve(self.model)
        # ... rest of solve logic

    def _apply_resource_state_count_soft(self, constraint):
        """Apply soft constraint with slack variables for violations."""
        resource = constraint.resource
        target_state = constraint.target_state
        time_slots = constraint.time_slots
        operator = constraint.operator
        value = constraint.value
        
        # Count occurrences of target_state
        bool_vars = []
        for t in time_slots:
            bv = self.model.NewBoolVar(f"soft_{resource}_{t}_{target_state}")
            self.model.Add(self.shifts[(resource, t)] == target_state).OnlyEnforceIf(bv)
            self.model.Add(self.shifts[(resource, t)] != target_state).OnlyEnforceIf(bv.Not())
            bool_vars.append(bv)
        
        count = sum(bool_vars)
        
        # Create slack variable for violation
        slack = self.model.NewIntVar(0, len(time_slots), f"slack_{resource}_{target_state}")
        
        if operator == ">=":
            # count + slack >= value (slack measures shortfall)
            self.model.Add(count + slack >= value)
        elif operator == "<=":
            # count - slack <= value (slack measures excess)
            self.model.Add(count <= value + slack)
        elif operator == "==":
            # |count - value| = slack
            diff = self.model.NewIntVar(-len(time_slots), len(time_slots), f"diff_{resource}")
            self.model.Add(diff == count - value)
            self.model.AddAbsEquality(slack, diff)
        
        # Add penalty (weight = 100 per unit of violation)
        PENALTY_WEIGHT = 100
        self.penalties.append(slack * PENALTY_WEIGHT)
```

---

## 5. TypeScript Integration

### 5.1 Solver Integration Service

Update `solver-integration.service.ts` to pass `is_required`:

```typescript
// In buildSolverRequest method
const solverConstraints: SolverConstraint[] = constraints.map((c) => {
  const config = c.config as Record<string, unknown>
  const baseConstraint = {
    is_required: c.isRequired ?? true,  // Include isRequired
  }
  
  switch (c.type) {
    case ConstraintType.VERTICAL_SUM:
      return {
        ...baseConstraint,
        type: ConstraintType.VERTICAL_SUM,
        time_slot: config.time_slot as number | 'ALL',
        target_state: config.target_state as number,
        operator: config.operator as '>=' | '<=' | '==',
        value: config.value as number,
      }
    // ... other cases
  }
})
```

### 5.2 Constraint Builder UI

Update `constraint-builder.tsx` to show Hard/Soft badge:

```tsx
// Table header
<TableHead>Required</TableHead>

// Table cell
<TableCell>
  <Badge variant={c.isRequired ? "default" : "secondary"}>
    {c.isRequired ? "Hard" : "Soft"}
  </Badge>
</TableCell>

// Edit dialog - add toggle
<div className="flex items-center space-x-2">
  <Switch
    id="edit-required"
    checked={editFormData.isRequired}
    onCheckedChange={(checked) => 
      setEditFormData({ ...editFormData, isRequired: checked })
    }
  />
  <Label htmlFor="edit-required">
    {editFormData.isRequired ? "Hard Constraint" : "Soft Constraint"}
  </Label>
</div>
```

---

## 6. Fairness Weights Configuration

### 6.1 System Config: fairness_weights

Add to seed.ts for weekday-only fairness calculation:

```typescript
await upsertItem(coreGlobal.id, 'fairness_weights', {
  label: 'Fairness Calculation Weights',
  type: 'fairness',
  value: {
    enabled: true,
    // Weights for June 2025 (Sun=0, Mon=1, ..., Sat=6)
    // Exclude weekends (weight = 0), weekdays = 1
    day_weights: {
      0: 0,  // Sunday - exclude
      1: 1,  // Monday
      2: 1,  // Tuesday
      3: 1,  // Wednesday
      4: 1,  // Thursday
      5: 1,  // Friday
      6: 0,  // Saturday - exclude
    },
    penalty_weight: 100,  // Per unit of soft constraint violation
  },
  locked: false,
})
```

### 6.2 Usage in Solver

When calculating night shift distribution fairness, only count weekday nights:

```python
def _calculate_fairness_penalty(self, fairness_config):
    """Calculate penalty for uneven night distribution (weekdays only)."""
    day_weights = fairness_config.get('day_weights', {})
    
    # Count weighted night shifts per resource
    for resource in self.resources:
        weighted_nights = []
        for t in range(self.time_slots):
            day_of_week = t % 7  # Assumes time_slot 0 = first day of month
            weight = day_weights.get(str(day_of_week), 1)
            
            if weight > 0:
                bv = self.model.NewBoolVar(f"wn_{resource}_{t}")
                self.model.Add(self.shifts[(resource, t)] == NIGHT_STATE).OnlyEnforceIf(bv)
                self.model.Add(self.shifts[(resource, t)] != NIGHT_STATE).OnlyEnforceIf(bv.Not())
                weighted_nights.append(bv)
        
        # Add to fairness tracking
        # ... minimize variance across all resources
```

---

## 7. Testing Strategy

### 7.1 Unit Tests

1. **Hard constraint enforcement**: Verify INFEASIBLE when hard constraint cannot be satisfied
2. **Soft constraint relaxation**: Verify FEASIBLE with penalties when soft constraint violated
3. **Penalty calculation**: Verify correct penalty values for violations
4. **Optimization**: Verify solver minimizes total penalty

### 7.2 Integration Tests

1. **End-to-end roster generation**: Generate roster with mix of hard/soft constraints
2. **Fairness validation**: Verify night shift distribution is more balanced
3. **UI toggle**: Verify isRequired toggle updates database correctly

---

## 8. Files to Modify

| File | Changes |
|------|---------|
| `prisma/schema.prisma` | Add `isRequired` field to Constraint model |
| `prisma/seed.ts` | Add `isRequired` to constraint creates, add `fairness_weights` config |
| `src/features/dashboard/components/constraint-builder.tsx` | Add Required column, toggle in edit dialog |
| `src/services/solver-integration.service.ts` | Pass `is_required` in constraint payload |
| `orto-ai-engine/src/core/schemas.py` | Add `is_required` to all constraint models |
| `orto-ai-engine/src/services/solver.py` | Implement soft constraint handling with penalties |

---

## 9. Success Criteria

1. ✅ `isRequired` field added to Constraint model and migrated
2. ✅ All coverage/safety constraints marked as `isRequired: true`
3. ✅ Night distribution constraint marked as `isRequired: false`
4. ✅ UI shows Hard/Soft badge for each constraint
5. ✅ Solver applies soft constraints with penalty minimization
6. ✅ Night shift distribution is more balanced (variance reduced)
