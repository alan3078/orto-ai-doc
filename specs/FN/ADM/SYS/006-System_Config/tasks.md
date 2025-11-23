# Tasks – FN/ADM/SYS/006 System Config Layer
Status Legend: ✅ Done | 🚧 In Progress | ⏳ Pending

## High-Level Ticket Breakdown
- SYS/006-01: Schema & Seed
- SYS/006-02: Python Engine Upgrade (Extended Types / Pattern Block)
- SYS/006-03: Mapper Logic
- SYS/006-04: Admin Config UI
- SYS/006-05: Roster Visibility

## Detailed Task Checklist
| ID | Task | Status | Notes |
|----|------|--------|-------|
| 1 | Add Prisma system config models | ✅ | `SystemConfigGroup`, `SystemConfigItem`, enum created in `schema.prisma`. |
| 2 | Extend seed with config groups/items | 🚧 | Upsert logic added; awaiting migration run & seed execution. |
| 3 | Migration generation/reset | 🚧 | Schema ready; awaiting fresh baseline migration. |
| 4 | Create mapper module | ✅ | `mapper.ts` implemented with fetch & transform logic. |
| 5 | Merge system constraints in roster action | ✅ | Updated `generate-roster.action.ts` & solver integration service. |
| 6 | Admin config UI page | ✅ | Created `/app/admin/config/` with table, edit dialog, server actions. |
| 7 | Sidebar Active Policies list | ✅ | Added collapsible section in `sidebar.tsx` + API route. |
| 8 | Solver pattern handling (Night→Day block) | ✅ | Added `PatternBlockConstraint` schema + handler in solver. |
| 9 | Python Night-Day pattern test | ✅ | Added 3 tests in `test_solver.py`; all passing. |
| 10 | Split specs into plan/implementation/tasks | ✅ | Files: `plan.md`, `implementation.md`, `tasks.md`. |

## Next Actions
1. Rebuild migrations from clean slate (dev reset) and apply.
2. Run seed to materialize groups/items.
3. Implement mapper & roster action merge.
4. Extend engine + tests.
5. Deliver admin UI & sidebar visibility.

## Blockers & Risks
- Migration shadow DB error (P3006) requires reset to avoid ordering conflict.
- State codes for Night/Day need confirmation before horizontal/pattern logic.

## Completion Criteria
- All tasks marked ✅.
- Roster generation includes system-derived constraints & passes Night→Day test.
- Admin & sidebar views reflect live system policies.

