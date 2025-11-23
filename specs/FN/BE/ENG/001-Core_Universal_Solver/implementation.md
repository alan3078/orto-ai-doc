# Implementation - FN/BE/ENG/001

## 1. Tech Stack

- **Language**: Python 3.10+
- **Framework**: FastAPI (for the Web API layer)
- **Solver**: Google OR-Tools (`ortools.sat.python.cp_model`)
- **Validation**: Pydantic

## 2. JSON Input Schema (The Contract)

The API will accept `POST /api/v1/solve` with this body:

```json
{
  "config": {
    "resources": ["A", "B", "C"],  // Staff IDs
    "time_slots": 7,                 // e.g., 7 days
    "states": [0, 1]                 // 0=Off, 1=Work
  },
  "constraints": [
    {
      "type": "point",
      "resource": "A",
      "time_slot": 0,
      "state": 0
      // Logic: Resource A at Time 0 MUST be State 0 (Off)
    },
    {
      "type": "vertical_sum",
      "time_slot": "ALL",
      "target_state": 1,
      "operator": ">=",
      "value": 2
      // Logic: For every time slot, count of State 1 must be >= 2
    }
  ]
}
```

## 3. Logic Mapping (Python)

We will create a `SolverService` class.

### 3.1 Variable Creation

We create a generic matrix of Integer Variables.

```python
shifts = {}
for r in resources:
    for t in range(time_slots):
        shifts[(r, t)] = model.NewIntVar(0, len(states)-1, f'shift_{r}_{t}')
```

### 3.2 Constraint Parsers

**Parser A: Point Constraint**

```python
# if type == 'point'
r = rule['resource']
t = rule['time_slot']
s = rule['state']
model.Add(shifts[(r, t)] == s)
```

**Parser B: Vertical Sum (Coverage)**

```python
# if type == 'vertical_sum'
# We need to create a temporary boolean var for "Is this state?" because shifts is an Int
# is_state_var = 1 if shift == target_state else 0
target_state = rule['target_state']
limit = rule['value']

for t in range(time_slots):
    # Create booleans for summation
    bool_vars = []
    for r in resources:
        b = model.NewBoolVar(f'temp_{r}_{t}')
        model.Add(shifts[(r, t)] == target_state).OnlyEnforceIf(b)
        model.Add(shifts[(r, t)] != target_state).OnlyEnforceIf(b.Not())
        bool_vars.append(b)
    
    if operator == ">=":
        model.Add(sum(bool_vars) >= limit)
    elif operator == "<=":
        model.Add(sum(bool_vars) <= limit)
```

## 4. Output Schema

```json
{
  "status": "OPTIMAL", // or FEASIBLE, INFEASIBLE
  "schedule": {
    "A": [0, 1, 1, 1, 0, 1, 1],
    "B": [1, 1, 0, 1, 1, 0, 0],
    "C": [1, 0, 1, 1, 1, 1, 1]
  }
}
```
