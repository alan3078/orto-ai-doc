# FN/ADM/SYS/006 – System Configuration Layer

Status: Draft → Implementation
Priority: High
Depends On: Existing constraint types (point, vertical_sum, horizontal_sum, sliding_window)
Related Tickets: FN/ADM/RUL/002 (Nurse Patterns), FN/AI/005 (AI Constraints)

## 1. Objective
Introduce a first-class System Configuration layer that supplies non user-authored, centrally managed scheduling policies ("Active Policies") to the solver alongside user-created constraints. Two groups:
- Global Group (code: `core_global`) – always applied
- Roster Management Group (code: `roster_management`) – applied during roster generation; editable except locked items

System configurations are stored as JSON-valued items that can (a) directly translate into solver constraints, or (b) provide parameters used to generate derived constraints (e.g. blocking Night→Day transitions).

## 2. Scope
In scope:
- Prisma models: `SystemConfigGroup`, `SystemConfigItem`, `SystemConfigScope` enum
- Seed data for nurse safety baseline policies
- Mapping layer to convert active items to solver-ready constraints
- Admin UI to view and (for unlocked roster items) update JSON values
- Dashboard/sidebar read-only list of active policies
- Python engine enforcement for Night→Day transition block

Out of scope (future):
- Versioning / audit history for config changes
- Per-organization / multi-tenant separation
- Advanced validation schemas per item type
- Fine-grained role-based access control

## 3. High-Level Flow
1. Seed establishes two groups + initial items
2. Roster generation action fetches active system items (global + roster), maps them to constraints
3. User constraints + system constraints merged, sent to Python
4. Python enforces both standard constraint types and derived pattern rules
5. UI surfaces: Admin config page (editable roster items), sidebar "Active Policies" (read-only list)

## 4. Initial Policies
Global (locked):
- `max_consecutive_nights` – limit consecutive night shifts per staff
- `night_to_day_block` – prevent immediate transition from Night (e.g. state=2?) to Day (state=1) next slot

Roster (editable):
- `max_weekly_hours` (future placeholder)
- `min_daily_coverage` – can mirror vertical sum requirement

## 5. Derived Constraint Examples
- `night_to_day_block` produces multiple point/horizontal style constraints preventing pattern: Night at slot N followed by Day at slot N+1.
- `max_consecutive_nights` becomes a horizontal_sum constraint template per staff group or staff id.

## 6. Open Questions
- Should `max_consecutive_nights` apply to all staff automatically, or only those in nursing groups? (Default: all staff)
- State encoding reference: Confirm consistent mapping for Night vs Day (document assumptions in Engine spec). 
- Large-scale patterns (e.g. weekends) – implement later.

## 7. Success Criteria
- Two config groups exist and seed without wiping user data
- System config items visible and JSON editable where unlocked
- Active policies appear in sidebar table
- Solver run includes system-derived constraints; tests validate Night→Day block

## 8. Risks & Mitigations
- Risk: Unvalidated JSON leads to runtime errors → Provide lightweight shape sanity checks in mapper.
- Risk: Seed script truncation clearing system tables → Adjust seed to upsert, not truncate system tables.
- Risk: Overlapping constraints cause infeasibility → Flag system vs user origin in constraint list for debugging.

## 9. Rollout
Phase 1: Schema + seed + mapper + Python test
Phase 2: Admin UI + sidebar policies
Phase 3: Extended validation & analytics
