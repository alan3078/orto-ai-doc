# System Config Layer – Plan

## Vision
Introduce a System Configuration layer providing centrally managed scheduling policies (Global + Roster-scoped) that automatically merge with user constraints for the Python solver.

## Scope (Phase 1)
- Schema & Migration: Add `SystemConfigGroup`, `SystemConfigItem`, `SystemConfigScope`.
- Seed: Establish baseline nurse safety policies.
- Mapping: Convert active config items → solver constraint objects.
- Integration: Merge system + user constraints in roster generation.
- Engine: Enforce Night→Day transition block and max consecutive nights.
- UI: Admin page to view/edit (unlocked) roster policies; sidebar table for Active Policies.

## Groups
- Global (`core_global`): Always applied; locked items.
- Roster Management (`roster_management`): Applied during roster generation; editable JSON where not locked.

## Phased Plan
1. Schema + Migration + Seed
   - Add models & enum.
   - Rebuild migrations if necessary (dev reset) and seed upsert groups/items.
2. Mapping Layer
   - Implement `mapper.ts` to fetch groups/items and build `systemConstraints`.
3. Roster Action Integration
   - Pull system constraints; merge with user constraints before solver call.
4. Engine Enhancements
   - Add support for runtime `pattern_block` or direct Night→Day check.
   - Add unit test for Night→Day block.
5. UI Delivery
   - Admin config page with table + JSON edit.
   - Sidebar Active Policies table (read-only).
6. Stabilization
   - Logging, guard against invalid JSON values, performance review.

## Constraint Derivation Rules
- `max_consecutive_nights` → horizontal_sum per staff (limit from JSON).
- `night_to_day_block` → pattern_block or solver adjacency prohibition.
- `min_daily_coverage` → vertical_sum (ALL time slots).

## Success Metrics
- Policies seeded & visible.
- Roster generation includes derived system constraints.
- Test verifying Night→Day transition blocked passes.
- Admin edits to roster-scoped JSON reflected in subsequent solves.

## Out of Scope (Future)
- Auditing/version history
- Per-tenant segmentation
- Advanced JSON schema validation
- UI diff previews of derived constraints

## Risks & Mitigations
- Migration conflicts → Use full dev reset & fresh init migration.
- Invalid JSON → Light shape validation in mapper.
- Infeasibility from overlapping rules → Tag origin (system/user) for debugging and surface in roster status messages.
