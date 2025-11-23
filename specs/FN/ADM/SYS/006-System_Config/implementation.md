# System Config Layer – Implementation Details

## 1. Prisma Schema
Models added:
- `SystemConfigGroup(id, code, name, scope, isActive, locked, createdAt, updatedAt)`
- `SystemConfigItem(id, groupId, key, label, type, value Json, priority, isActive, locked, createdAt, updatedAt)`
- Enum `SystemConfigScope { GLOBAL, ROSTER }`
Indices: scope, isActive; unique composite (groupId,key).

## 2. Migration Strategy
Development reset approach:
1. Remove existing migrations (`rm -rf prisma/migrations`).
2. `npx prisma migrate reset --force` (drops & recreates schema from scratch using current schema file).
3. `npx prisma migrate dev --name init` (creates fresh baseline migration including new models).
4. `npx prisma generate`.

Alternative (fast sync, no history): `npx prisma db push --force-reset`.

## 3. Seed Logic (`prisma/seed.ts`)
- Avoid truncating `system_config_group` & `system_config_item`.
- Upsert groups by `code`.
- Upsert items by composite unique `(groupId, key)`; skip overwrite unless `RESET_SYSTEM_CONFIG=true`.
- Items: 
  - Global: `max_consecutive_nights { limit: 3 }`, `night_to_day_block { enabled: true }` (locked).
  - Roster: `min_daily_coverage { min: 3, target_state: 1 }` (editable).

## 4. Mapper (`web/src/features/engine/mapper.ts`)
Exports:
- `fetchSystemPolicies({ includeRoster: boolean })` → raw policies list.
- `buildSystemConstraints(policies, staffList)` → derived constraints array.

Validation:
- Ensure each policy has `value` object.
- Skip unknown `type` with warning.

Generated Constraint Types:
- Horizontal Sum: For each staff when `max_consecutive_nights` defined.
- Vertical Sum: Single constraint for `min_daily_coverage`.
- Pattern Block: Custom runtime object for `night_to_day_block` (type: `pattern_block`).

## 5. Roster Action Integration
Patch `generate-roster.action.ts`:
1. Load staff & user constraints (existing).
2. Fetch system policies (global + roster).
3. Derive system constraints.
4. Merge: `const constraintsForSolver = [...systemConstraints, ...userConstraints]`.
5. Pass to solver integration service.
6. Optionally record applied system policy IDs with roster metadata (future).

## 6. Engine Changes (`engine/src/services/solver.py`)
Add handling:
- Accept additional `pattern_block` constraints.
- Precompute forbidden transitions set (Night→Day) from policy; verify each staff sequence avoids it.
- Integrate with existing feasibility checks (horizontal_sum, sliding_window, etc.).

Pseudo addition:
```python
for c in constraints:
    if c['type'] == 'pattern_block':
        forbidden_pairs.add(('NIGHT','DAY'))
# During sequence evaluation:
if (prev_state, curr_state) in forbidden_pairs: infeasible = True
```
(State mapping confirmed separately; adapt string to ints.)

## 7. Python Unit Test (`engine/src/tests/test_solver.py`)
Test: `test_night_to_day_block()`
Steps:
1. Provide minimal roster context with potential Night→Day.
2. Inject `pattern_block` via test fixture.
3. Run solver; assert result either adjusts schedule or returns infeasible, depending on design.

## 8. Admin UI (`/app/admin/config/page.tsx`)
Components:
- Fetch groups + items server-side.
- Render two tables: Global (read-only), Roster (editable JSON).
- Edit flow: Click row → dialog with `<textarea>` for JSON; parse & validate; call server action.
- Server action `updateSystemConfigItemValue(itemId, jsonString)` guards locked items.

## 9. Sidebar Active Policies
Update `sidebar.tsx`:
- Add collapsible section: "Active Policies".
- Render concise table: Policy | Scope | Value Summary.
- Use badge for scope (GLOBAL/ROSTER). Limit value preview (truncate JSON to ~40 chars).

## 10. Error Handling
- Mapper: log unknown types, skip.
- Server action: reject invalid JSON parse or locked item edits.
- Solver: If system policy leads to infeasibility, tag response with `systemPolicyNames` for UI feedback (future addition).

## 11. Logging
- Seed script: prints creation/upsert status.
- Mapper: counts generated constraints & skipped policies.
- Solver: logs applied pattern blocks and horizontal limits.

## 12. Performance Considerations
- Deriving horizontal_sum for every staff may scale linearly; optimize later by grouping staff categories.
- Pattern block check is O(n) per staff sequence.

## 13. Future Enhancements
- JSON schema validation per `type`.
- Policy versioning/audit trail.
- Role-based access for editing roster policies.
- Pre-solve feasibility simulation (show potential conflicts).

## 14. Deployment Notes
- After merging schema: run migration commands under Node 22.x.
- Ensure environment variable `RESET_SYSTEM_CONFIG` only set intentionally.

