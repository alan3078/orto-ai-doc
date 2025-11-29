# Orto AI - Project Phases

## Beta Release

Generate a duty roster fulfilling the below requirements:

- [x] 1. Able to generate 2 types of duty: 7/E (07-19, 19-07) and A/P/N (07-15, 15-23, 23-07)
- [x] 2. Able to assign individual working hours in target range, i.e. specific monthly hour according to different company
- [x] 3. Able to assign target manpower in different shifts, i.e. 4D2E or 4A4P2N
- [x] 4. Able to balance number of E or N shift among all staff, i.e. 4-6 E/N each staff
- [x] 5. Able to assign IC role staff in every shift
- [x] 6. Able to avoid male-only shift, i.e. at least one female staff every shift
- [x] 7. Able to assign appropriate shift block, i.e. at most 3 shifts in a row, 1 week between night shift block
- [x] 8. Able to assign dayoff after E/N shifts
- [ ] 9. Able to relatively balance numbers of dayoff monthly (placeholder - determined by coverage)

---

## Version 1.0

Includes all Beta requirements plus:

- [ ] 1. Able to interact with last month duty to avoid mistakes
- [ ] 2. Able to identify SL/PH/AL or other leaves and generate appropriate duty
- [ ] 3. Able to generate duty according to staff dayoff and no-Night requests
- [ ] 4. Able to assign IC role according to staff title (IC role mapping), plus SRN and RN group balance
- [ ] 5. Able to customize special shifts
- [ ] 6. Able to generate hybrid duty, i.e. A/7/P/E/N/customized shifts together
- [ ] 7. Able to show number of different shifts and work hours for every staff
- [ ] 8. Generate 2 types of accounts: Manager / Member
- [ ] 9. Manager: all client permissions; Member: Duty Viewer, duty request, password reset
- [ ] 10. Add active/inactive function on staff card
- [ ] 11. Language Support: CN / TC / EN
- [ ] 12. 1st time user experience - user agreement + user password reset
- [ ] 13. 1st time user experience - Manager: enter last 3 days of last month roster
- [ ] 14. Number of Weekend/PH and Night balance annually

---

## Version 1.1

Furthermore:

- [ ] 1. Able to balance number of different shift types annually among all staff
- [ ] 2. Able to balance number of different shift types according to manager's requirements
- [ ] 3. Able to customize individual shifts requirements monthly
- [ ] 4. Able to show all cumulative numbers of all staff

---

## Implementation Status Legend

| Symbol | Meaning |
|--------|---------|
| [x] | Completed |
| [ ] | Not Started |
| [~] | In Progress |

---

## Technical Implementation Mapping

### Beta Features → Current Implementation

| Requirement | Implementation | Status |
|-------------|----------------|--------|
| 7/E and A/P/N shift types | `ShiftTypeConfig` + `ShiftDefinition` tables | ✅ |
| Individual working hours | `ShiftTypeConfig.minHoursPerMonth/maxHoursPerMonth` | ✅ |
| Target manpower (4D2E, 4A4P2N) | `vertical_sum` constraints in Constraint table | ✅ |
| Balance E/N shifts (4-6) | `night_distribution` system config | ✅ |
| IC role in every shift | `attribute_vertical_sum` / `compound_attribute_vertical_sum` | ✅ |
| Avoid male-only shift | `attribute_vertical_sum` with gender filter | ✅ |
| Shift blocks (max 3 consecutive) | `max_consecutive_nights` + `sliding_window` system config | ✅ |
| Dayoff after E/N | `pattern_block` constraint + `post_night_rest` | ✅ |
| Balance dayoff monthly | Placeholder - determined by coverage requirements | ⏳ |

### Key Components

- **Solver Engine**: `orto-ai-engine` (Python + OR-Tools CP-SAT)
- **Frontend**: `orto-ai-app` (Next.js 15 + Prisma 7)
- **Constraint System**: User constraints + System config → Merged in `solver-integration.service.ts`
- **Shift Settings UI**: `/admin/config/shift-settings`
- **System Config UI**: `/admin/config`
