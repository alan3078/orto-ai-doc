# 003 – Engine & UI Integration

## 1. Mapping Pipeline
Location (planned): `web/src/features/engine/mapper.ts`
Responsibilities:
1. Fetch active SystemConfigItems (global + roster)
2. Produce two artifacts:
   - `systemPolicies`: raw normalized list for UI (id, groupCode, key, label, value, locked)
   - `systemConstraints`: array of constraint objects matching existing solver schema
3. Merge `systemConstraints` with user `Constraint` records prior to solver call

### 1.1 Fetch Logic (Pseudo)
```ts
const groups = await prisma.systemConfigGroup.findMany({
  where: { isActive: true },
  include: { items: { where: { isActive: true } } }
})
```
Filter items by `scope`:
- Always include GLOBAL
- Include ROSTER when generating roster (context flag)

### 1.2 Transformation Rules
- `max_consecutive_nights` → For each staff resource create `horizontal_sum` constraint limiting consecutive night state occurrences.
  - Assume night state numeric code (confirm: using existing states; placeholder: state=2 for Night if current mapping; adjust after validation)
  - Config shape:
    ```json
    {
      "type": "horizontal_sum",
      "resource": STAFF_ID,
      "time_slots": [0,1,2,3,4,5,6],
      "target_state": NIGHT_STATE,
      "operator": "<=",
      "value": limit
    }
    ```
- `night_to_day_block` → Derive pattern block: prevent Night → Day immediate transition.
  - Strategy: generate invalid transition constraints using sliding window or synthetic point forbids; simplest: create a set of constraints marking slot N+1 Off if slot N Night (post-processing in solver) OR add solver logic directly (preferred: engine side check, produce metadata constraint with custom type `pattern_block` not enforced by Prisma). For MVP: create constraints using a custom `pattern_block` type consumed only by solver.
  - Example item config:
    ```json
    { "enabled": true }
    ```
    Mapped constraint object:
    ```json
    { "name": "Block Night→Day Consecutive", "type": "pattern_block", "config": { "pattern": ["NIGHT","DAY"], "resources": "ALL" } }
    ```
- `min_daily_coverage` → vertical_sum constraint (existing type)
  ```json
  {
    "type": "vertical_sum",
    "time_slot": "ALL",
    "target_state": target_state,
    "operator": ">=",
    "value": min
  }
  ```

### 1.3 Unknown Types
Items with unrecognized `type` logged and skipped.

## 2. Solver Integration
File impacted: `web/src/app/actions/generate-roster.action.ts`
Flow modification:
1. Fetch system constraints via mapper
2. Read user constraints via existing query
3. Merge: `const allConstraints = [...systemConstraints, ...userConstraints]`
4. Pass to solver integration service
5. Include `systemPolicies` for front-end display if needed

## 3. Python Engine Changes
Files: `engine/src/services/solver.py`, tests in `engine/src/tests/test_solver.py`

### 3.1 Pattern Block Enforcement
Add support for custom `pattern_block` constraints (not persisted in Prisma, only runtime). Handling:
- During assignment generation or validation phase, scan each staff sequence for forbidden two-length transitions.
- If detected, mark solution infeasible or adjust search to avoid transition.

Pseudo:
```python
for c in constraints:
    if c.type == 'pattern_block':
        apply_pattern_block(c.config)
```
`apply_pattern_block` builds adjacency forbidden pairs used by solver.

### 3.2 Night→Day Transition Test
Test case ensures when `night_to_day_block` active, any candidate schedule with Night followed by Day fails / not produced.

Test Steps:
1. Inject system constraint `pattern_block`.
2. Provide roster scenario with minimal days forcing potential Night→Day.
3. Assert solver output lacks that transition OR solver returns infeasible if unavoidable.

### 3.3 Horizontal Sum Nights
Reuse existing `horizontal_sum` logic; ensure night state ID consistent (update if actual state mapping differs from assumption).

## 4. UI Components

### 4.1 Admin Config Page (`/app/admin/config/page.tsx`)
Sections:
- Global Policies (read-only table)
- Roster Policies (editable JSON value via modal or inline textarea)
Columns: Label, Key, Type, Value (stringified excerpt), Active toggle (if not locked), Locked indicator.
Server Actions:
- `updateSystemConfigItemValue(id: number, value: Json)` – validation & persistence
- Guards: reject if `locked === true`

### 4.2 Sidebar Active Policies List
Add collapsible "Active Policies" beneath existing navigation, rendering table (narrow):
Columns: Policy, Scope (G/R), Value summary.

### 4.3 Value Editing UX
- Click row → opens drawer/dialog with JSON editor (basic `<textarea>` initially)
- Validate JSON parse client-side before server action call

## 5. Data Fetching
Shared hook or server-side function:
`getActiveSystemPolicies(scopeFilter?: 'ALL'|'GLOBAL'|'ROSTER')`
Returns:
```ts
{
  policies: Array<{ id: number; scope: 'GLOBAL'|'ROSTER'; key: string; label: string; type: string; value: unknown; locked: boolean }>
}
```

## 6. Error Handling
- Missing tables (pre-migration) → mapper returns empty arrays
- Invalid JSON update → action returns structured error
- Infeasible due to system constraints → roster status INFEASIBLE with aggregated system constraint names for debugging

## 7. Performance Considerations
- Single query with nested items (groups+items) is small
- Mapping cost linear in staff count for `max_consecutive_nights`; optimize by group-level constraint later

## 8. Security
- Ensure admin page route restricted (future auth middleware) – current assumption: only admins can access `/admin/*`
- Prevent editing locked items server-side regardless of client state

## 9. Logging & Observability
- Mapper logs count of system constraints generated
- Solver logs forbidden transition application summary

## 10. Future Enhancements
- Schema-based validation per item type
- Real-time preview of derived constraints before save
- Differential application per staff group (limit nights only for specific groups)

