# FN/ADM/STF/007 – Extended Staff & Shift Definitions (Plan)

## Objective
Introduce richer staff profiles (gender, ordered roles, monthly contract hour bounds) and codified shift definitions (code, start time, duration) with activation and audited hard deletion. Provide groundwork for future coverage and contract compliance constraints in the solver.

## Scope (MVP)
- Staff: gender (F/M), monthlyMinHours, monthlyMaxHours, normalized roles via join table.
- Roles: ordered priority (e.g., IC first), activation control.
- Shifts: ShiftDefinition table with code ("N","E","A","P","7"), startMinutes (0–1439), durationMinutes, isActive. Hard delete produces audit record; soft delete toggles isActive.
- Audit: ShiftDefinitionAudit captures deleted shift data + metadata.
- Engine: Expose resource_attributes { employeeId: { gender, rolesOrdered, monthlyMaxHours } } placeholder.
- Backward compatibility: Existing roster generation untouched; new features optional.

## Constraints & Decisions
- Gender limited to enum { F, M } per requirement; extensibility possible later.
- Role ordering: enforce unique Int order; lowest numbers = higher priority.
- Shift times stored as minutes from midnight (Int) for efficient arithmetic.
- Monthly hours: stored as optional Int bounds; enforcement deferred to later constraint implementation.
- Pivot table chosen over string array for roles to allow ordering, metadata & future constraints.

## Data Model Changes (Summary)
- enum Gender { F M }
- model Role (id, name, order, isActive, timestamps)
- model StaffRole (composite PK staffId+roleId, assignedAt)
- Staff additions: gender, monthlyMinHours, monthlyMaxHours
- model ShiftDefinition (activation + deletedAt)
- model ShiftDefinitionAudit (snapshot of deleted definitions)

## API/UI Additions (Later Phases)
- Staff form: gender select, roles multi-select (ordered display), monthly hours inputs.
- Role management admin page (CRUD + ordering).
- Shift settings admin page (CRUD, activate/deactivate, hard delete).

## Non-Goals
- Weekly hour enforcement, payroll, skill mix logic, solver role coverage constraints (future).

## Risks & Mitigations
- Added joins may increase query complexity: use selective includes & indexing.
- Hard delete race conditions: wrap delete + audit creation in a transaction.
- Order conflicts: validate uniqueness before update; may auto-resequence later.

## Milestones
1. Schema + migrations
2. Seed defaults (roles & shifts)
3. Staff UI enhancements
4. Role management UI
5. Shift settings UI
6. Solver integration attributes
7. Documentation & task completion tracking

## Success Criteria
- Can create staff Amy (F) with role IC stored in Role + StaffRole.
- Can define shift N (start 1380, duration 480) active.
- Hard deleting shift N writes audit entry.
- Resource attributes map contains gender & ordered roles for each roster staff.
