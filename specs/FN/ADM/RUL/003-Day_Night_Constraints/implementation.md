# Implementation - FN/ADM/RUL/003 - Day/Night Staffing Constraints

---

## 1. Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        orto-ai-app                              │
├─────────────────────────────────────────────────────────────────┤
│  Prisma Schema          │  Zod Validation       │  UI Layer    │
│  - Shift.isIC field     │  - CompoundAttr...    │  - RosterGrid│
│                         │    Schema             │    IC color  │
└────────────┬────────────┴──────────┬────────────┴──────┬───────┘
             │                       │                   │
             ▼                       ▼                   ▼
┌─────────────────────────────────────────────────────────────────┐
│                       orto-ai-engine                            │
├─────────────────────────────────────────────────────────────────┤
│  schemas.py                    │  solver.py                     │
│  - CompoundAttributeVertical   │  - _apply_compound_attribute_  │
│    SumConstraint               │    vertical_sum_constraint()   │
├────────────────────────────────┼────────────────────────────────┤
│  validator.py                  │                                │
│  - _validate_compound_         │                                │
│    attribute_vertical_sum()    │                                │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Backend Implementation (Python)

### 2.1 Schema Definition

**File:** `orto-ai-engine/src/core/schemas.py`

```python
class CompoundAttributeVerticalSumConstraint(BaseModel):
    """Constraint: Sum of specific state across resources filtered by MULTIPLE attributes (AND logic).
    
    Example: At least 1 female IC (gender=F AND role=IC) working (state=1) each time slot.
    """

    type: Literal["compound_attribute_vertical_sum"] = "compound_attribute_vertical_sum"
    time_slot: Union[int, Literal["ALL"]] = Field(
        ...,
        description="Time slot index or 'ALL' for all time slots"
    )
    target_state: int = Field(..., ge=0, description="State to count")
    operator: Literal[">=", "<=", "=="] = Field(..., description="Comparison operator")
    value: int = Field(..., ge=0, description="Target value for comparison")
    attribute_filters: Dict[str, List[str]] = Field(
        ...,
        description="Attribute filters combined with AND logic. E.g., {'gender': ['F'], 'roles': ['IC']}"
    )
```

### 2.2 Solver Implementation

**File:** `orto-ai-engine/src/services/solver.py`

```python
def _apply_compound_attribute_vertical_sum_constraint(
    self, 
    constraint: CompoundAttributeVerticalSumConstraint
):
    """Apply compound attribute filtered vertical sum constraint.
    
    Filters resources where ALL attribute conditions match (AND logic),
    then counts resources in target_state.
    """
    target_state = constraint.target_state
    operator = constraint.operator
    value = constraint.value
    attribute_filters = constraint.attribute_filters

    if constraint.time_slot == "ALL":
        slots = range(self.time_slots)
    else:
        slots = [constraint.time_slot]

    # Pre-filter resources matching ALL attribute conditions (AND logic)
    filtered_resources = []
    for r in self.resources:
        if r not in self.resource_attributes:
            continue
        
        # Check all attribute filters (AND logic)
        all_match = True
        for attr_key, attr_values in attribute_filters.items():
            if attr_key not in self.resource_attributes[r]:
                all_match = False
                break
            
            raw_val = self.resource_attributes[r][attr_key]
            attr_values_set = set(attr_values)
            
            # Handle list attributes (e.g., roles: ['IC', 'RN'])
            if isinstance(raw_val, list):
                match = any(str(v) in attr_values_set for v in raw_val)
            else:
                match = str(raw_val) in attr_values_set
            
            if not match:
                all_match = False
                break
        
        if all_match:
            filtered_resources.append(r)

    # Apply vertical sum constraint to filtered resources
    for t in slots:
        bool_vars = []
        for resource in filtered_resources:
            b = self.model.NewBoolVar(
                f"compound_{resource}_{t}_{target_state}"
            )
            self.model.Add(
                self.shifts[(resource, t)] == target_state
            ).OnlyEnforceIf(b)
            self.model.Add(
                self.shifts[(resource, t)] != target_state
            ).OnlyEnforceIf(b.Not())
            bool_vars.append(b)
        
        sum_expr = sum(bool_vars)
        if operator == ">=":
            self.model.Add(sum_expr >= value)
        elif operator == "<=":
            self.model.Add(sum_expr <= value)
        elif operator == "==":
            self.model.Add(sum_expr == value)
```

### 2.3 Validator Implementation

**File:** `orto-ai-engine/src/services/validator.py`

```python
def _validate_compound_attribute_vertical_sum(
    self,
    constraint: CompoundAttributeVerticalSumConstraint,
    index: int,
    name: Optional[str]
) -> ConstraintValidationResult:
    """Validate compound attribute vertical sum constraint against schedule."""
    target_state = constraint.target_state
    operator = constraint.operator
    value = constraint.value
    attribute_filters = constraint.attribute_filters
    
    # Pre-filter resources matching ALL attributes
    filtered_resources = []
    for r in self.resources:
        if r not in self.resource_attributes:
            continue
        
        all_match = True
        for attr_key, attr_values in attribute_filters.items():
            if attr_key not in self.resource_attributes[r]:
                all_match = False
                break
            
            raw_val = self.resource_attributes[r][attr_key]
            attr_values_set = set(attr_values)
            
            if isinstance(raw_val, list):
                match = any(str(v) in attr_values_set for v in raw_val)
            else:
                match = str(raw_val) in attr_values_set
            
            if not match:
                all_match = False
                break
        
        if all_match:
            filtered_resources.append(r)
    
    # Check constraint at each time slot
    if constraint.time_slot == "ALL":
        slots = range(self.time_slots)
    else:
        slots = [constraint.time_slot]
    
    violations = []
    for t in slots:
        count = sum(
            1 for r in filtered_resources 
            if self.schedule[r][t] == target_state
        )
        
        passes = self._compare(count, operator, value)
        if not passes:
            violations.append({
                "time_slot": t,
                "expected": f"{operator} {value}",
                "actual": count,
                "filtered_resources": filtered_resources
            })
    
    status = "PASS" if len(violations) == 0 else "FAIL"
    return ConstraintValidationResult(
        constraint_index=index,
        constraint_type="compound_attribute_vertical_sum",
        constraint_name=name,
        status=status,
        details=f"Compound filter {attribute_filters}: {len(violations)} violations" if violations else "All time slots pass",
        violations=violations
    )
```

---

## 3. TypeScript Implementation

### 3.1 Zod Schema

**File:** `orto-ai-app/src/lib/validations/solver.ts`

```typescript
export const CompoundAttributeVerticalSumConstraintSchema = z.object({
  type: z.literal('compound_attribute_vertical_sum'),
  time_slot: z.union([z.number().nonnegative(), z.literal('ALL')]),
  target_state: z.number().nonnegative(),
  operator: z.enum(['>=', '<=', '==']),
  value: z.number().nonnegative(),
  attribute_filters: z.record(z.string(), z.array(z.string())),
})

export type CompoundAttributeVerticalSumConstraint = z.infer<
  typeof CompoundAttributeVerticalSumConstraintSchema
>
```

---

## 4. Database Schema Extension

### 4.1 Prisma Migration

**File:** `orto-ai-app/prisma/schema.prisma`

```prisma
model Shift {
  id         String   @id @default(cuid())
  rosterId   String   @map("roster_id")
  roster     Roster   @relation(fields: [rosterId], references: [id], onDelete: Cascade)
  staffId    String   @map("staff_id")
  staff      Staff    @relation(fields: [staffId], references: [id], onDelete: Cascade)
  timeSlot   Int      @map("time_slot")
  state      Int
  date       DateTime
  notes      String?
  isIC       Boolean  @default(false) @map("is_ic")  // NEW FIELD
  createdAt  DateTime @default(now()) @map("created_at")

  @@unique([rosterId, staffId, timeSlot])
  @@index([rosterId, timeSlot])
  @@index([staffId, date])
  @@map("shift")
}
```

---

## 5. Frontend Implementation

### 5.1 Roster Grid IC Highlighting

**File:** `orto-ai-app/src/features/dashboard/components/roster-grid.tsx`

```tsx
import { ROSTER_GRID_COLORS } from '../constants/roster-colors';

// In the table cell rendering:
<td
  key={i}
  className='p-3 text-center border'
  style={{
    backgroundColor: shift?.isIC ? ROSTER_GRID_COLORS.LIGHT_GREEN : undefined
  }}
>
  {shift ? (
    <div className='flex justify-center'>
      <StateBadge state={shift.state} shiftType={shiftType} />
    </div>
  ) : (
    <span className='text-muted-foreground'>-</span>
  )}
</td>
```

---

## 6. Seed Data

### 6.1 Day/Night Constraints

**File:** `orto-ai-app/prisma/seed.ts`

```typescript
// Day/Night Mode Constraints
const dayNightConstraints = [
  {
    name: 'Minimum Day Staff',
    description: 'At least 3 staff on Day shift',
    type: 'vertical_sum',
    config: {
      time_slot: 'ALL',
      target_state: 1,  // Day
      operator: '>=',
      value: 3
    },
    shiftType: 'DAY_NIGHT',
    priority: 100
  },
  {
    name: 'Minimum Night Staff',
    description: 'At least 2 staff on Night shift',
    type: 'vertical_sum',
    config: {
      time_slot: 'ALL',
      target_state: 2,  // Night
      operator: '>=',
      value: 2
    },
    shiftType: 'DAY_NIGHT',
    priority: 100
  },
  {
    name: 'Female IC Day',
    description: 'At least 1 female IC-eligible staff on Day shift',
    type: 'compound_attribute_vertical_sum',
    config: {
      time_slot: 'ALL',
      target_state: 1,  // Day
      operator: '>=',
      value: 1,
      attribute_filters: {
        gender: ['F'],
        roles: ['IC']
      }
    },
    shiftType: 'DAY_NIGHT',
    priority: 90
  },
  {
    name: 'Female IC Night',
    description: 'At least 1 female IC-eligible staff on Night shift',
    type: 'compound_attribute_vertical_sum',
    config: {
      time_slot: 'ALL',
      target_state: 2,  // Night
      operator: '>=',
      value: 1,
      attribute_filters: {
        gender: ['F'],
        roles: ['IC']
      }
    },
    shiftType: 'DAY_NIGHT',
    priority: 90
  },
  {
    name: 'No Night-to-Day',
    description: 'Block Night shift immediately followed by Day shift',
    type: 'pattern_block',
    config: {
      pattern: ['NIGHT', 'DAY'],
      resources: 'ALL',
      state_mapping: {
        'OFF': 0,
        'DAY': 1,
        'NIGHT': 2
      }
    },
    shiftType: 'DAY_NIGHT',
    priority: 80
  }
]
```

---

## 7. Testing Strategy

### 7.1 Unit Tests

```python
# test_solver.py
def test_compound_attribute_vertical_sum_female_ic():
    """Test compound constraint filters by gender AND role."""
    request = SolveRequest(
        config=ConfigModel(
            resources=["A", "B", "C", "D"],
            time_slots=7,
            states=[0, 1, 2],
            resource_attributes={
                "A": {"gender": "F", "roles": ["IC"]},      # Female IC
                "B": {"gender": "F", "roles": ["Non-IC"]},  # Female Non-IC
                "C": {"gender": "M", "roles": ["IC"]},      # Male IC
                "D": {"gender": "M", "roles": ["Non-IC"]},  # Male Non-IC
            }
        ),
        constraints=[
            CompoundAttributeVerticalSumConstraint(
                time_slot="ALL",
                target_state=1,
                operator=">=",
                value=1,
                attribute_filters={"gender": ["F"], "roles": ["IC"]}
            )
        ]
    )
    
    response = solver_service.solve(request)
    assert response.status in ["OPTIMAL", "FEASIBLE"]
    
    # Verify: At each time slot with state=1, at least one must be "A" (only female IC)
    for t in range(7):
        day_staff = [r for r in ["A", "B", "C", "D"] if response.schedule[r][t] == 1]
        female_ic_on_day = [r for r in day_staff if r == "A"]
        assert len(female_ic_on_day) >= 1, f"Time slot {t} missing female IC"
```

---

## 8. Implementation Status

| Step | Component | File | Status |
|------|-----------|------|--------|
| 1 | Documentation | plan.md, implementation.md, tasks.md | ⬜ |
| 2 | Python Schema | schemas.py | ⬜ |
| 3 | Python Solver | solver.py | ⬜ |
| 4 | Python Validator | validator.py | ⬜ |
| 5 | TypeScript Schema | solver.ts | ⬜ |
| 6 | Prisma Migration | schema.prisma | ⬜ |
| 7 | UI Highlighting | roster-grid.tsx | ⬜ |
| 8 | Seed Data | seed.ts | ⬜ |

---

**Document Version:** 1.0  
**Created:** November 27, 2025  
