# 002 – Data Model & Seed Plan

## 1. Prisma Models
```prisma
enum SystemConfigScope {
  GLOBAL
  ROSTER
}

model SystemConfigGroup {
  id        Int               @id @default(autoincrement())
  code      String            @unique
  name      String
  scope     SystemConfigScope
  isActive  Boolean           @default(true) @map("is_active")
  locked    Boolean           @default(false)
  items     SystemConfigItem[]
  createdAt DateTime          @default(now()) @map("created_at")
  updatedAt DateTime          @updatedAt @map("updated_at")

  @@index([scope])
  @@index([isActive])
  @@map("system_config_group")
}

model SystemConfigItem {
  id         Int               @id @default(autoincrement())
  groupId    Int               @map("group_id")
  group      SystemConfigGroup @relation(fields: [groupId], references: [id], onDelete: Cascade)
  key        String
  label      String
  type       String
  value      Json
  priority   Int               @default(0)
  isActive   Boolean           @default(true) @map("is_active")
  locked     Boolean           @default(false)
  createdAt  DateTime          @default(now()) @map("created_at")
  updatedAt  DateTime          @updatedAt @map("updated_at")

  @@unique([groupId, key])
  @@index([isActive])
  @@index([type])
  @@map("system_config_item")
}
```

## 2. JSON Value Conventions
Each `value` field is JSON; expected top-level keys vary by `type`:
- `nurse_safety.max_consecutive_nights`: `{ "limit": number }`
- `nurse_safety.night_to_day_block`: `{ "enabled": boolean }`
- `coverage.min_daily_coverage`: `{ "min": number, "target_state": number }`

Mapper must treat unknown `type` as passthrough (ignored until supported).

## 3. Seed Strategy
Adjust seed script to:
1. Remove full-table truncation for `system_config_group` & `system_config_item` (avoid losing edits).
2. Upsert groups by `code`.
3. Upsert items by composite `[groupId, key]` after fetching group IDs.
4. Only create items if missing; do not overwrite existing `value` unless a hard reset env var (e.g. `RESET_SYSTEM_CONFIG=true`).

## 4. Seed Data (Initial)
Global group (`code: core_global`, locked=true):
| key | label | type | value | locked |
|-----|-------|------|-------|--------|
| max_consecutive_nights | Max Consecutive Night Shifts | nurse_safety | `{ "limit": 3 }` | true |
| night_to_day_block | Block Night→Day Immediate Transition | nurse_safety | `{ "enabled": true }` | true |

Roster group (`code: roster_management`, locked=false):
| key | label | type | value | locked |
|-----|-------|------|-------|--------|
| min_daily_coverage | Minimum Daily Coverage | coverage | `{ "min": 3, "target_state": 1 }` | false |

## 5. Migration Naming
Command:
```
npx prisma migrate dev --name add_system_config
```
Resulting SQL should create two tables with snake_case columns + indices.

## 6. Data Integrity Rules
- A group must have unique `code` (stable reference in code).
- Item uniqueness enforced by `[groupId, key]`.
- Locked items: UI must prevent modification (server-side guard).
- Deactivating a group deactivates downstream items logically (mapper filters by both flags).

## 7. Soft vs Hard Deletion
Currently no deletion planned—use `isActive=false` to disable. Future: add archival table if audit needed.

## 8. Potential Extensions
- Add `category` enum to `SystemConfigItem` for grouping in UI
- Add `jsonSchema` validation column at group level for item shapes

## 9. Open Questions
- Should coverage constraints be generated per time slot or global? (Initial: global ALL).
- Should max nights apply uniformly to each staff ID or only those working Night state? (Initial: apply when state encountered.)

## 10. Rollback Considerations
Migration reversible: dropping tables affects mapper—ensure try/catch in mapper gracefully handles absence.
