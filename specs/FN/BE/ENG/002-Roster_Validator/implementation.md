# Implementation - FN/BE/ENG/002 - Roster Validator (Audit Mode)

## 1. Architecture Overview

```
┌─────────────────┐         ┌──────────────────┐         ┌─────────────────┐
│   Web Frontend  │         │  Next.js Backend │         │  Python Engine  │
│                 │         │                  │         │                 │
│  Check Rules    │────────▶│  validateRoster  │────────▶│  POST /validate │
│  Button         │         │  Action          │         │                 │
│                 │         │                  │         │  Validator      │
│  Validation     │◀────────│  Transform       │◀────────│  Service        │
│  Drawer         │         │  Results         │         │                 │
└─────────────────┘         └──────────────────┘         └─────────────────┘
```

## 2. Backend Implementation (Python)

### 2.1 Validation Schemas (`engine/src/core/schemas.py`)

Add the following to existing schemas:

```python
class ValidateRequest(BaseModel):
    """Request model for the validate endpoint."""
    
    config: ConfigModel  # Same as SolveRequest
    schedule: Dict[str, List[int]] = Field(
        ...,
        description="Existing schedule to validate: resource_id -> [state_per_timeslot]"
    )
    constraints: List[ConstraintModel] = Field(
        default_factory=list,
        description="List of constraints to validate against"
    )
    
    @field_validator('schedule')
    @classmethod
    def validate_schedule(cls, v, info):
        """Validate schedule matrix matches config."""
        if 'config' not in info.data:
            return v
            
        config = info.data['config']
        
        # Check all resources are present
        if set(v.keys()) != set(config.resources):
            raise ValueError(
                f"Schedule resources {set(v.keys())} don't match config resources {set(config.resources)}"
            )
        
        # Check all state arrays have correct length and valid states
        for resource_id, states in v.items():
            if len(states) != config.time_slots:
                raise ValueError(
                    f"Resource {resource_id} has {len(states)} time slots, expected {config.time_slots}"
                )
            for state in states:
                if state not in config.states:
                    raise ValueError(
                        f"Resource {resource_id} has invalid state {state}, allowed states: {config.states}"
                    )
        
        return v


class ConstraintValidationResult(BaseModel):
    """Result for a single constraint validation."""
    
    constraint_index: int = Field(..., description="Index of constraint in input array")
    constraint_type: str = Field(..., description="Type of constraint (point, vertical_sum, etc.)")
    constraint_name: Optional[str] = Field(None, description="Human-readable name if provided")
    status: Literal["PASS", "FAIL"] = Field(..., description="Validation status")
    details: Optional[str] = Field(None, description="Human-readable explanation of result")
    violations: Optional[List[Dict[str, Any]]] = Field(
        default_factory=list,
        description="List of specific violations (resource, time_slot, expected, actual)"
    )


class ValidateResponse(BaseModel):
    """Response model for the validate endpoint."""
    
    overall_status: Literal["PASS", "FAIL"] = Field(
        ...,
        description="PASS if all constraints pass, FAIL if any fail"
    )
    total_constraints: int = Field(..., description="Total number of constraints checked")
    passed_constraints: int = Field(..., description="Number of constraints that passed")
    failed_constraints: int = Field(..., description="Number of constraints that failed")
    results: List[ConstraintValidationResult] = Field(
        ...,
        description="Detailed results for each constraint"
    )
    validation_time_ms: Optional[float] = Field(
        None,
        description="Time taken to validate in milliseconds"
    )
```

### 2.2 Validator Service (`engine/src/services/validator.py`)

Create new file:

```python
"""
Validator service for checking constraints against existing schedules.
Uses imperative validation (no OR-Tools) for performance.
"""

from typing import List, Dict, Any, Optional
from src.core.schemas import (
    ConfigModel,
    ConstraintModel,
    PointConstraint,
    VerticalSumConstraint,
    HorizontalSumConstraint,
    SlidingWindowConstraint,
    PatternBlockConstraint,
    AttributeVerticalSumConstraint,
    ResourceStateCountConstraint,
    ConstraintValidationResult,
)


class ConstraintValidator:
    """Validates constraints against an existing schedule matrix."""
    
    def __init__(self, config: ConfigModel, schedule: Dict[str, List[int]]):
        self.config = config
        self.schedule = schedule
        self.resource_attrs = config.resource_attributes or {}
    
    def validate_constraint(
        self, 
        constraint: ConstraintModel, 
        index: int,
        name: Optional[str] = None
    ) -> ConstraintValidationResult:
        """Validate a single constraint against the schedule."""
        
        if isinstance(constraint, PointConstraint):
            return self._validate_point(constraint, index, name)
        elif isinstance(constraint, VerticalSumConstraint):
            return self._validate_vertical_sum(constraint, index, name)
        elif isinstance(constraint, HorizontalSumConstraint):
            return self._validate_horizontal_sum(constraint, index, name)
        elif isinstance(constraint, SlidingWindowConstraint):
            return self._validate_sliding_window(constraint, index, name)
        elif isinstance(constraint, PatternBlockConstraint):
            return self._validate_pattern_block(constraint, index, name)
        elif isinstance(constraint, AttributeVerticalSumConstraint):
            return self._validate_attribute_vertical_sum(constraint, index, name)
        elif isinstance(constraint, ResourceStateCountConstraint):
            return self._validate_resource_state_count(constraint, index, name)
        else:
            return ConstraintValidationResult(
                constraint_index=index,
                constraint_type="unknown",
                constraint_name=name,
                status="FAIL",
                details=f"Unsupported constraint type: {type(constraint).__name__}",
                violations=[]
            )
    
    def _validate_point(
        self, 
        constraint: PointConstraint, 
        index: int,
        name: Optional[str]
    ) -> ConstraintValidationResult:
        """Validate point constraint: Resource X at Time Y must be State Z."""
        actual_state = self.schedule[constraint.resource][constraint.time_slot]
        
        if actual_state == constraint.state:
            return ConstraintValidationResult(
                constraint_index=index,
                constraint_type="point",
                constraint_name=name,
                status="PASS",
                details=f"{constraint.resource} at time {constraint.time_slot} is state {constraint.state} ✓",
                violations=[]
            )
        else:
            return ConstraintValidationResult(
                constraint_index=index,
                constraint_type="point",
                constraint_name=name,
                status="FAIL",
                details=f"{constraint.resource} at time {constraint.time_slot} is state {actual_state}, expected {constraint.state}",
                violations=[{
                    "resource": constraint.resource,
                    "time_slot": constraint.time_slot,
                    "expected": constraint.state,
                    "actual": actual_state
                }]
            )
    
    def _validate_vertical_sum(
        self, 
        constraint: VerticalSumConstraint, 
        index: int,
        name: Optional[str]
    ) -> ConstraintValidationResult:
        """Validate vertical sum: Count of target_state across resources at time slot(s)."""
        violations = []
        
        time_slots = (
            range(self.config.time_slots) 
            if constraint.time_slot == "ALL" 
            else [constraint.time_slot]
        )
        
        for ts in time_slots:
            count = sum(
                1 for resource_states in self.schedule.values() 
                if resource_states[ts] == constraint.target_state
            )
            
            # Check operator
            passes = self._check_operator(count, constraint.operator, constraint.value)
            
            if not passes:
                violations.append({
                    "time_slot": ts,
                    "expected": f"{constraint.operator} {constraint.value}",
                    "actual": count,
                    "target_state": constraint.target_state
                })
        
        if violations:
            return ConstraintValidationResult(
                constraint_index=index,
                constraint_type="vertical_sum",
                constraint_name=name,
                status="FAIL",
                details=f"Failed at {len(violations)} time slot(s): {', '.join(str(v['time_slot']) for v in violations)}",
                violations=violations
            )
        else:
            return ConstraintValidationResult(
                constraint_index=index,
                constraint_type="vertical_sum",
                constraint_name=name,
                status="PASS",
                details=f"All time slots meet {constraint.operator} {constraint.value} workers in state {constraint.target_state} ✓",
                violations=[]
            )
    
    def _validate_horizontal_sum(
        self, 
        constraint: HorizontalSumConstraint, 
        index: int,
        name: Optional[str]
    ) -> ConstraintValidationResult:
        """Validate horizontal sum: Count of target_state for resource across time slots."""
        resource_states = self.schedule[constraint.resource]
        count = sum(
            1 for ts in constraint.time_slots 
            if resource_states[ts] == constraint.target_state
        )
        
        passes = self._check_operator(count, constraint.operator, constraint.value)
        
        if passes:
            return ConstraintValidationResult(
                constraint_index=index,
                constraint_type="horizontal_sum",
                constraint_name=name,
                status="PASS",
                details=f"{constraint.resource} has {count} occurrences of state {constraint.target_state} (meets {constraint.operator} {constraint.value}) ✓",
                violations=[]
            )
        else:
            return ConstraintValidationResult(
                constraint_index=index,
                constraint_type="horizontal_sum",
                constraint_name=name,
                status="FAIL",
                details=f"{constraint.resource} has {count} occurrences of state {constraint.target_state}, expected {constraint.operator} {constraint.value}",
                violations=[{
                    "resource": constraint.resource,
                    "time_slots": constraint.time_slots,
                    "expected": f"{constraint.operator} {constraint.value}",
                    "actual": count,
                    "target_state": constraint.target_state
                }]
            )
    
    def _validate_sliding_window(
        self, 
        constraint: SlidingWindowConstraint, 
        index: int,
        name: Optional[str]
    ) -> ConstraintValidationResult:
        """Validate sliding window: work-rest pattern enforcement."""
        resource_states = self.schedule[constraint.resource]
        violations = []
        
        window_size = constraint.work_days + constraint.rest_days
        
        for start in range(len(resource_states) - window_size + 1):
            window = resource_states[start:start + window_size]
            work_days_count = sum(1 for s in window[:constraint.work_days] if s == constraint.target_state)
            
            # If work_days period is fully worked, check rest_days
            if work_days_count == constraint.work_days:
                rest_days_count = sum(1 for s in window[constraint.work_days:] if s != constraint.target_state)
                if rest_days_count < constraint.rest_days:
                    violations.append({
                        "resource": constraint.resource,
                        "start_time_slot": start,
                        "end_time_slot": start + window_size - 1,
                        "issue": f"After {constraint.work_days} work days, only {rest_days_count}/{constraint.rest_days} rest days"
                    })
        
        if violations:
            return ConstraintValidationResult(
                constraint_index=index,
                constraint_type="sliding_window",
                constraint_name=name,
                status="FAIL",
                details=f"{constraint.resource} violates {constraint.work_days} work / {constraint.rest_days} rest pattern at {len(violations)} position(s)",
                violations=violations
            )
        else:
            return ConstraintValidationResult(
                constraint_index=index,
                constraint_type="sliding_window",
                constraint_name=name,
                status="PASS",
                details=f"{constraint.resource} meets {constraint.work_days} work / {constraint.rest_days} rest pattern ✓",
                violations=[]
            )
    
    def _validate_pattern_block(
        self, 
        constraint: PatternBlockConstraint, 
        index: int,
        name: Optional[str]
    ) -> ConstraintValidationResult:
        """Validate pattern block: Ensure forbidden transitions don't occur."""
        violations = []
        
        # Get state mapping
        state_map = constraint.state_mapping or {}
        pattern_states = [state_map.get(p, None) for p in constraint.pattern]
        
        if None in pattern_states:
            return ConstraintValidationResult(
                constraint_index=index,
                constraint_type="pattern_block",
                constraint_name=name,
                status="FAIL",
                details=f"Invalid state_mapping for pattern {constraint.pattern}",
                violations=[]
            )
        
        # Check all resources
        for resource_id, resource_states in self.schedule.items():
            for t in range(len(resource_states) - 1):
                if (resource_states[t] == pattern_states[0] and 
                    resource_states[t + 1] == pattern_states[1]):
                    violations.append({
                        "resource": resource_id,
                        "time_slot": t,
                        "pattern": constraint.pattern,
                        "issue": f"{constraint.pattern[0]} → {constraint.pattern[1]} transition forbidden"
                    })
        
        if violations:
            return ConstraintValidationResult(
                constraint_index=index,
                constraint_type="pattern_block",
                constraint_name=name,
                status="FAIL",
                details=f"Found {len(violations)} forbidden {constraint.pattern[0]}→{constraint.pattern[1]} transitions",
                violations=violations
            )
        else:
            return ConstraintValidationResult(
                constraint_index=index,
                constraint_type="pattern_block",
                constraint_name=name,
                status="PASS",
                details=f"No forbidden {constraint.pattern[0]}→{constraint.pattern[1]} transitions ✓",
                violations=[]
            )
    
    def _validate_attribute_vertical_sum(
        self, 
        constraint: AttributeVerticalSumConstraint, 
        index: int,
        name: Optional[str]
    ) -> ConstraintValidationResult:
        """Validate attribute vertical sum: Filtered resource count by attribute."""
        # Filter resources by attribute
        filtered_resources = [
            res_id for res_id in self.config.resources
            if res_id in self.resource_attrs
            and constraint.attribute in self.resource_attrs[res_id]
            and self.resource_attrs[res_id][constraint.attribute] in constraint.attribute_values
        ]
        
        violations = []
        time_slots = (
            range(self.config.time_slots) 
            if constraint.time_slot == "ALL" 
            else [constraint.time_slot]
        )
        
        for ts in time_slots:
            count = sum(
                1 for res_id in filtered_resources
                if self.schedule[res_id][ts] == constraint.target_state
            )
            
            passes = self._check_operator(count, constraint.operator, constraint.value)
            
            if not passes:
                violations.append({
                    "time_slot": ts,
                    "expected": f"{constraint.operator} {constraint.value}",
                    "actual": count,
                    "attribute_filter": f"{constraint.attribute} in {constraint.attribute_values}"
                })
        
        if violations:
            return ConstraintValidationResult(
                constraint_index=index,
                constraint_type="attribute_vertical_sum",
                constraint_name=name,
                status="FAIL",
                details=f"Failed at {len(violations)} time slot(s) for {constraint.attribute} filter",
                violations=violations
            )
        else:
            return ConstraintValidationResult(
                constraint_index=index,
                constraint_type="attribute_vertical_sum",
                constraint_name=name,
                status="PASS",
                details=f"All time slots meet {constraint.operator} {constraint.value} filtered workers ✓",
                violations=[]
            )
    
    def _validate_resource_state_count(
        self, 
        constraint: ResourceStateCountConstraint, 
        index: int,
        name: Optional[str]
    ) -> ConstraintValidationResult:
        """Validate resource state count: Total occurrences of state for resource."""
        resource_states = self.schedule[constraint.resource]
        count = sum(
            1 for ts in constraint.time_slots
            if resource_states[ts] == constraint.target_state
        )
        
        passes = self._check_operator(count, constraint.operator, constraint.value)
        
        if passes:
            return ConstraintValidationResult(
                constraint_index=index,
                constraint_type="resource_state_count",
                constraint_name=name,
                status="PASS",
                details=f"{constraint.resource} has {count} occurrences of state {constraint.target_state} (meets {constraint.operator} {constraint.value}) ✓",
                violations=[]
            )
        else:
            return ConstraintValidationResult(
                constraint_index=index,
                constraint_type="resource_state_count",
                constraint_name=name,
                status="FAIL",
                details=f"{constraint.resource} has {count} occurrences of state {constraint.target_state}, expected {constraint.operator} {constraint.value}",
                violations=[{
                    "resource": constraint.resource,
                    "expected": f"{constraint.operator} {constraint.value}",
                    "actual": count,
                    "target_state": constraint.target_state
                }]
            )
    
    @staticmethod
    def _check_operator(actual: int, operator: str, expected: int) -> bool:
        """Check if actual value meets operator comparison with expected."""
        if operator == ">=":
            return actual >= expected
        elif operator == "<=":
            return actual <= expected
        elif operator == "==":
            return actual == expected
        else:
            return False
```

### 2.3 API Endpoint (`engine/src/api/v1/solve.py`)

Add to existing router:

```python
from src.core.schemas import ValidateRequest, ValidateResponse, ConstraintValidationResult
from src.services.validator import ConstraintValidator
import time

@router.post("/validate", response_model=ValidateResponse)
async def validate(request: ValidateRequest) -> ValidateResponse:
    """
    Validate an existing schedule against a set of constraints.
    
    Args:
        request: ValidateRequest containing config, schedule, and constraints
        
    Returns:
        ValidateResponse with per-constraint validation results
        
    Example:
        ```json
        {
          "config": {
            "resources": ["A", "B", "C"],
            "time_slots": 7,
            "states": [0, 1]
          },
          "schedule": {
            "A": [0, 1, 1, 0, 0, 1, 0],
            "B": [1, 1, 0, 0, 1, 1, 0],
            "C": [0, 0, 1, 1, 1, 0, 0]
          },
          "constraints": [
            {
              "type": "point",
              "resource": "A",
              "time_slot": 0,
              "state": 0
            },
            {
              "type": "vertical_sum",
              "time_slot": "ALL",
              "target_state": 1,
              "operator": ">=",
              "value": 2
            }
          ]
        }
        ```
    """
    try:
        start_time = time.time()
        
        validator = ConstraintValidator(request.config, request.schedule)
        
        results = []
        for idx, constraint in enumerate(request.constraints):
            result = validator.validate_constraint(constraint, idx)
            results.append(result)
        
        passed = sum(1 for r in results if r.status == "PASS")
        failed = sum(1 for r in results if r.status == "FAIL")
        
        validation_time_ms = (time.time() - start_time) * 1000
        
        return ValidateResponse(
            overall_status="PASS" if failed == 0 else "FAIL",
            total_constraints=len(results),
            passed_constraints=passed,
            failed_constraints=failed,
            results=results,
            validation_time_ms=validation_time_ms
        )
    except Exception as e:
        raise HTTPException(
            status_code=500,
            detail=f"Validation error: {str(e)}"
        )
```

## 3. Frontend Implementation (Next.js)

### 3.1 Validation Schemas (`web/src/lib/validations/validator.ts`)

Create new file:

```typescript
import { z } from 'zod'

export const ConstraintValidationResultSchema = z.object({
  constraint_index: z.number(),
  constraint_type: z.string(),
  constraint_name: z.string().nullable().optional(),
  status: z.enum(['PASS', 'FAIL']),
  details: z.string().nullable().optional(),
  violations: z.array(z.record(z.any())).optional(),
})

export const ValidateResponseSchema = z.object({
  overall_status: z.enum(['PASS', 'FAIL']),
  total_constraints: z.number(),
  passed_constraints: z.number(),
  failed_constraints: z.number(),
  results: z.array(ConstraintValidationResultSchema),
  validation_time_ms: z.number().nullable().optional(),
})

export type ConstraintValidationResult = z.infer<typeof ConstraintValidationResultSchema>
export type ValidateResponse = z.infer<typeof ValidateResponseSchema>
```

### 3.2 Extend Solver Integration Service

Add to `web/src/services/solver-integration.service.ts`:

```typescript
import { ValidateResponseSchema, type ValidateResponse } from '@/lib/validations/validator'

/**
 * Validate an existing roster against constraints
 * @param rosterId Database ID of the roster to validate
 */
async validateRoster(rosterId: string): Promise<ValidateResponse> {
  // Step 1: Fetch roster with shifts and constraints from database
  const roster = await prisma.roster.findUnique({
    where: { id: rosterId },
    include: {
      shifts: {
        include: { staff: true },
        orderBy: [{ staffId: 'asc' }, { timeSlot: 'asc' }],
      },
      constraints: { where: { isActive: true } },
    },
  })

  if (!roster) {
    throw new Error(`Roster ${rosterId} not found`)
  }

  if (!roster.shifts || roster.shifts.length === 0) {
    throw new Error(`Roster ${rosterId} has no shifts to validate`)
  }

  // Step 2: Rebuild schedule matrix from shifts
  const schedule: Record<string, number[]> = {}
  const staffMap = new Map<string, string>() // staffId -> employeeId

  for (const shift of roster.shifts) {
    const employeeId = shift.staff.employeeId
    staffMap.set(shift.staffId, employeeId)
    
    if (!schedule[employeeId]) {
      schedule[employeeId] = []
    }
    schedule[employeeId][shift.timeSlot] = shift.state
  }

  // Step 3: Get staff details for resource_attributes
  const staffIds = Array.from(staffMap.keys())
  const staff = await prisma.staff.findMany({
    where: { id: { in: staffIds } },
    include: {
      staffRoles: { include: { role: true } },
    },
  })

  const resourceAttributes = Object.fromEntries(
    staff.map((s) => [
      s.employeeId,
      {
        gender: s.gender,
        roles: s.staffRoles.map((sr) => sr.role.name),
      },
    ])
  )

  // Step 4: Build validation request
  const validateRequest = {
    config: {
      resources: Object.keys(schedule),
      time_slots: roster.timeSlots,
      states: roster.states,
      resource_attributes: resourceAttributes,
    },
    schedule,
    constraints: roster.constraints.map((c) => {
      const config = c.config as Record<string, unknown>
      // Transform DB constraint to solver format (same logic as generateRoster)
      switch (c.type) {
        case 'point':
          return {
            type: 'point',
            resource: config.resource as string,
            time_slot: config.time_slot as number,
            state: config.state as number,
          }
        case 'vertical_sum':
          return {
            type: 'vertical_sum',
            time_slot: config.time_slot as number | 'ALL',
            target_state: config.target_state as number,
            operator: config.operator as '>=' | '<=' | '==',
            value: config.value as number,
          }
        case 'horizontal_sum':
          return {
            type: 'horizontal_sum',
            resource: config.resource as string,
            time_slots: config.time_slots as number[],
            target_state: config.target_state as number,
            operator: config.operator as '>=' | '<=' | '==',
            value: config.value as number,
          }
        case 'sliding_window':
          return {
            type: 'sliding_window',
            resource: config.resource as string,
            work_days: config.work_days as number,
            rest_days: config.rest_days as number,
            target_state: config.target_state as number,
          }
        case 'pattern_block':
          return {
            type: 'pattern_block',
            pattern: config.pattern as string[],
            resources: 'ALL',
            state_mapping: config.state_mapping as Record<string, number>,
          }
        case 'attribute_vertical_sum':
          return {
            type: 'attribute_vertical_sum',
            time_slot: config.time_slot as number | 'ALL',
            target_state: config.target_state as number,
            operator: config.operator as '>=' | '<=' | '==',
            value: config.value as number,
            attribute: config.attribute as string,
            attribute_values: config.attribute_values as string[],
          }
        case 'resource_state_count':
          return {
            type: 'resource_state_count',
            resource: config.resource as string,
            time_slots: config.time_slots as number[],
            target_state: config.target_state as number,
            operator: config.operator as '>=' | '<=' | '==',
            value: config.value as number,
          }
        default:
          throw new Error(`Unsupported constraint type: ${c.type}`)
      }
    }),
  }

  // Step 5: Call Python validation API
  return this.callValidationApi(validateRequest)
}

/**
 * Call Python validation API
 */
private async callValidationApi(request: unknown): Promise<ValidateResponse> {
  const response = await fetch(`${this.engineUrl}/api/v1/validate`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(request),
  })

  if (!response.ok) {
    const errorText = await response.text()
    throw new Error(`Validation API error: ${response.status} ${errorText}`)
  }

  const data = await response.json()

  // Validate response
  return ValidateResponseSchema.parse(data)
}
```

### 3.3 Server Action (`web/src/app/actions/validate-roster.action.ts`)

Create new file:

```typescript
'use server'

import { SolverIntegrationService } from '@/services/solver-integration.service'

/**
 * Server Action to validate an existing roster against its constraints
 */
export async function validateRosterAction(rosterId: string) {
  try {
    const service = new SolverIntegrationService()
    const validationResult = await service.validateRoster(rosterId)

    return {
      success: true as const,
      data: validationResult,
    }
  } catch (error) {
    console.error('Validate roster error:', error)
    return {
      success: false as const,
      error: error instanceof Error ? error.message : 'Unknown error',
      data: null,
    }
  }
}
```

### 3.4 UI Components

#### ValidationDrawer (`web/src/features/dashboard/components/validation-drawer.tsx`)

```typescript
'use client'

import { useState } from 'react'
import { Button } from '@/components/ui/button'
import {
  Sheet,
  SheetContent,
  SheetDescription,
  SheetHeader,
  SheetTitle,
  SheetTrigger,
} from '@/components/ui/sheet'
import { Badge } from '@/components/ui/badge'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'
import { CheckCircle2, XCircle, AlertCircle, Loader2 } from 'lucide-react'
import { validateRosterAction } from '@/app/actions/validate-roster.action'
import type { ValidateResponse, ConstraintValidationResult } from '@/lib/validations/validator'

interface ValidationDrawerProps {
  rosterId: string
  children?: React.ReactNode
}

export function ValidationDrawer({ rosterId, children }: ValidationDrawerProps) {
  const [isOpen, setIsOpen] = useState(false)
  const [isValidating, setIsValidating] = useState(false)
  const [result, setResult] = useState<ValidateResponse | null>(null)
  const [error, setError] = useState<string | null>(null)

  const handleValidate = async () => {
    setIsValidating(true)
    setError(null)

    const response = await validateRosterAction(rosterId)

    setIsValidating(false)

    if (response.success && response.data) {
      setResult(response.data)
    } else {
      setError(response.error || 'Validation failed')
    }
  }

  return (
    <Sheet open={isOpen} onOpenChange={setIsOpen}>
      <SheetTrigger asChild>
        {children || (
          <Button variant="outline" size="sm">
            <AlertCircle className="h-4 w-4 mr-2" />
            Check Rules
          </Button>
        )}
      </SheetTrigger>
      <SheetContent side="right" className="w-full sm:max-w-2xl overflow-y-auto">
        <SheetHeader>
          <SheetTitle>Roster Validation Report</SheetTitle>
          <SheetDescription>
            Review constraint compliance for this roster
          </SheetDescription>
        </SheetHeader>

        <div className="mt-6 space-y-4">
          {/* Validate Button */}
          <Button 
            onClick={handleValidate} 
            disabled={isValidating}
            className="w-full"
          >
            {isValidating ? (
              <>
                <Loader2 className="h-4 w-4 mr-2 animate-spin" />
                Validating...
              </>
            ) : (
              'Run Validation'
            )}
          </Button>

          {/* Error State */}
          {error && (
            <Card className="border-destructive">
              <CardContent className="pt-6">
                <div className="flex items-center gap-2 text-destructive">
                  <XCircle className="h-5 w-5" />
                  <span className="font-medium">{error}</span>
                </div>
              </CardContent>
            </Card>
          )}

          {/* Results Summary */}
          {result && (
            <>
              <Card>
                <CardHeader>
                  <CardTitle className="flex items-center gap-2">
                    {result.overall_status === 'PASS' ? (
                      <CheckCircle2 className="h-5 w-5 text-green-600" />
                    ) : (
                      <XCircle className="h-5 w-5 text-destructive" />
                    )}
                    Overall Status: {result.overall_status}
                  </CardTitle>
                </CardHeader>
                <CardContent className="space-y-2">
                  <div className="flex justify-between text-sm">
                    <span className="text-muted-foreground">Total Constraints:</span>
                    <span className="font-medium">{result.total_constraints}</span>
                  </div>
                  <div className="flex justify-between text-sm">
                    <span className="text-muted-foreground">Passed:</span>
                    <span className="font-medium text-green-600">{result.passed_constraints}</span>
                  </div>
                  <div className="flex justify-between text-sm">
                    <span className="text-muted-foreground">Failed:</span>
                    <span className="font-medium text-destructive">{result.failed_constraints}</span>
                  </div>
                  {result.validation_time_ms && (
                    <div className="flex justify-between text-sm">
                      <span className="text-muted-foreground">Validation Time:</span>
                      <span className="font-medium">{result.validation_time_ms.toFixed(2)}ms</span>
                    </div>
                  )}
                </CardContent>
              </Card>

              {/* Individual Constraint Results */}
              <div className="space-y-3">
                <h3 className="font-semibold text-sm">Constraint Details</h3>
                {result.results.map((constraintResult, idx) => (
                  <ConstraintReportCard key={idx} result={constraintResult} />
                ))}
              </div>
            </>
          )}
        </div>
      </SheetContent>
    </Sheet>
  )
}

function ConstraintReportCard({ result }: { result: ConstraintValidationResult }) {
  return (
    <Card className={result.status === 'FAIL' ? 'border-destructive/50' : 'border-green-600/50'}>
      <CardHeader className="pb-3">
        <div className="flex items-start justify-between">
          <div className="flex-1">
            <div className="flex items-center gap-2 mb-1">
              {result.status === 'PASS' ? (
                <CheckCircle2 className="h-4 w-4 text-green-600 shrink-0" />
              ) : (
                <XCircle className="h-4 w-4 text-destructive shrink-0" />
              )}
              <Badge variant={result.status === 'PASS' ? 'outline' : 'destructive'} className="text-xs">
                {result.constraint_type}
              </Badge>
            </div>
            {result.constraint_name && (
              <p className="text-sm font-medium">{result.constraint_name}</p>
            )}
          </div>
          <Badge variant={result.status === 'PASS' ? 'default' : 'destructive'}>
            {result.status}
          </Badge>
        </div>
      </CardHeader>
      <CardContent>
        <p className="text-sm text-muted-foreground">{result.details}</p>
        
        {result.violations && result.violations.length > 0 && (
          <div className="mt-3 space-y-2">
            <p className="text-xs font-semibold text-destructive">Violations:</p>
            <div className="space-y-1">
              {result.violations.slice(0, 5).map((violation, idx) => (
                <div key={idx} className="text-xs bg-destructive/10 p-2 rounded">
                  <pre className="whitespace-pre-wrap font-mono">
                    {JSON.stringify(violation, null, 2)}
                  </pre>
                </div>
              ))}
              {result.violations.length > 5 && (
                <p className="text-xs text-muted-foreground italic">
                  ... and {result.violations.length - 5} more violations
                </p>
              )}
            </div>
          </div>
        )}
      </CardContent>
    </Card>
  )
}
```

### 3.5 Integration Points

Add validation button to roster pages:

#### Roster Management Page (`web/src/app/(admin)/roster-management/page.tsx`)

```typescript
import { ValidationDrawer } from '@/features/dashboard/components/validation-drawer'

// Add near RosterControls or in the header
<ValidationDrawer rosterId={currentRosterId}>
  <Button variant="outline" size="sm">
    <AlertCircle className="h-4 w-4 mr-2" />
    Check Rules
  </Button>
</ValidationDrawer>
```

#### Test Roster Detail Page (`web/src/app/test-roster/[id]/page.tsx`)

```typescript
import { ValidationDrawer } from '@/features/dashboard/components/validation-drawer'

// Add in the page header actions
<ValidationDrawer rosterId={params.id} />
```

## 4. Testing Strategy

### 4.1 Backend Unit Tests (`engine/src/tests/test_validator.py`)

```python
import pytest
from src.core.schemas import (
    ConfigModel,
    PointConstraint,
    VerticalSumConstraint,
    HorizontalSumConstraint,
)
from src.services.validator import ConstraintValidator


def test_validate_point_pass():
    config = ConfigModel(resources=["A", "B"], time_slots=3, states=[0, 1])
    schedule = {"A": [0, 1, 0], "B": [1, 0, 1]}
    validator = ConstraintValidator(config, schedule)
    
    constraint = PointConstraint(resource="A", time_slot=1, state=1)
    result = validator.validate_constraint(constraint, 0)
    
    assert result.status == "PASS"
    assert len(result.violations) == 0


def test_validate_point_fail():
    config = ConfigModel(resources=["A", "B"], time_slots=3, states=[0, 1])
    schedule = {"A": [0, 1, 0], "B": [1, 0, 1]}
    validator = ConstraintValidator(config, schedule)
    
    constraint = PointConstraint(resource="A", time_slot=1, state=0)
    result = validator.validate_constraint(constraint, 0)
    
    assert result.status == "FAIL"
    assert len(result.violations) == 1
    assert result.violations[0]["actual"] == 1


def test_validate_vertical_sum_all_pass():
    config = ConfigModel(resources=["A", "B", "C"], time_slots=3, states=[0, 1])
    schedule = {"A": [1, 1, 0], "B": [1, 0, 1], "C": [0, 1, 1]}
    validator = ConstraintValidator(config, schedule)
    
    constraint = VerticalSumConstraint(
        time_slot="ALL",
        target_state=1,
        operator=">=",
        value=2
    )
    result = validator.validate_constraint(constraint, 0)
    
    assert result.status == "PASS"


def test_validate_vertical_sum_fail():
    config = ConfigModel(resources=["A", "B"], time_slots=3, states=[0, 1])
    schedule = {"A": [0, 0, 0], "B": [0, 1, 0]}
    validator = ConstraintValidator(config, schedule)
    
    constraint = VerticalSumConstraint(
        time_slot="ALL",
        target_state=1,
        operator=">=",
        value=2
    )
    result = validator.validate_constraint(constraint, 0)
    
    assert result.status == "FAIL"
    assert len(result.violations) == 2  # Fails at slots 0 and 2
```

### 4.2 Frontend Integration Tests

Test validation drawer interaction, API error handling, and UI state management.

## 5. Performance Considerations

- **Validation Speed**: Should be < 2s for typical rosters (30 days, 20 constraints, 15 staff)
- **Memory Usage**: Schedule matrix is small (15 staff × 30 days = 450 integers ≈ 2KB)
- **Caching**: No caching in MVP; compute on-demand
- **Rate Limiting**: Consider adding rate limits if validation is abused

## 6. Security Considerations

- **Input Validation**: All inputs validated via Pydantic schemas
- **Authorization**: Ensure users can only validate their own organization's rosters (add auth layer post-MVP)
- **Resource Limits**: Set max roster size (e.g., 100 staff × 31 days) to prevent DoS

## 7. Documentation Requirements

- Update API documentation (OpenAPI/Swagger) with `/validate` endpoint
- Add validation feature to user guide with screenshots
- Document validation response format for integrators
