# FN/ADM/STF/007 – Implementation Notes

## Prisma Schema Changes
```prisma
enum Gender { F M }

model Role {
  id        String      @id @default(cuid())
  name      String      @unique
  order     Int         @unique
  isActive  Boolean     @default(true) @map("is_active")
  staffRoles StaffRole[]
  createdAt DateTime    @default(now()) @map("created_at")
  updatedAt DateTime    @updatedAt @map("updated_at")
  @@map("role")
}

model StaffRole {
  staffId    String   @map("staff_id")
  roleId     String   @map("role_id")
  staff      Staff    @relation(fields: [staffId], references: [id], onDelete: Cascade)
  role       Role     @relation(fields: [roleId], references: [id], onDelete: Cascade)
  assignedAt DateTime @default(now()) @map("assigned_at")
  @@id([staffId, roleId])
  @@index([roleId])
  @@map("staff_role")
}

model ShiftDefinition {
  id              String                  @id @default(cuid())
  code            String                  @unique
  startMinutes    Int                     @map("start_minutes")
  durationMinutes Int                     @map("duration_minutes")
  isActive        Boolean                 @default(true) @map("is_active")
  deletedAt       DateTime?               @map("deleted_at")
  audits          ShiftDefinitionAudit[]
  createdAt       DateTime                @default(now()) @map("created_at")
  updatedAt       DateTime                @updatedAt @map("updated_at")
  @@map("shift_definition")
}

model ShiftDefinitionAudit {
  id               String   @id @default(cuid())
  shiftDefinitionId String?  @map("shift_definition_id")
  code             String
  startMinutes     Int      @map("start_minutes")
  durationMinutes  Int      @map("duration_minutes")
  deletedAt        DateTime @default(now()) @map("deleted_at")
  deletedBy        String?  @map("deleted_by")
  reason           String?  @map("reason")
  @@map("shift_definition_audit")
}
```
Staff additions:
```prisma
  gender          Gender?    @map("gender")
  monthlyMinHours Int?       @map("monthly_min_hours")
  monthlyMaxHours Int?       @map("monthly_max_hours")
  staffRoles      StaffRole[]
```

## Soft vs Hard Delete Logic
- Soft delete: toggle `isActive=false` on `ShiftDefinition` (retain for reactivation).
- Hard delete: transaction:
  1. Insert snapshot into `ShiftDefinitionAudit` with original field values.
  2. Delete `ShiftDefinition` row.

## Seeding Defaults
In `prisma/seed.ts`:
```ts
await prisma.role.createMany({ data: [
  { id: cuid(), name: 'IC', order: 1 },
  { id: cuid(), name: 'RN', order: 2 }
]});
await prisma.shiftDefinition.createMany({ data: [
  { code: '7', startMinutes: 420, durationMinutes: 480 }, // 07:00-15:00
  { code: 'E', startMinutes: 900, durationMinutes: 480 }, // 15:00-23:00
  { code: 'A', startMinutes: 300, durationMinutes: 600 }, // 05:00-15:00 (example)
  { code: 'P', startMinutes: 840, durationMinutes: 600 }, // 14:00-24:00 (example)
  { code: 'N', startMinutes: 1380, durationMinutes: 480 } // 23:00-07:00 overnight
]});
```
(Validate actual shift patterns before production.)

## Resource Attributes Mapping
Add in solver integration:
```ts
resourceAttributes: {
  [employeeId]: {
    gender: staff.gender,
    rolesOrdered: roles.sort((a,b)=>a.order-b.order).map(r=>r.name),
    monthlyMaxHours: staff.monthlyMaxHours ?? null
  }
}
```

## UI Components
- Staff Form: Adds gender select (F/M), roles multi-select (ordered tags), monthly hours (min/max Int inputs).
- Role Management: Table (name, order, active) + create/edit modal; reordering via drag or numeric input.
- Shift Settings: Table (code, startMinutes -> HH:MM, durationMinutes -> hours), activate toggle, hard delete button (confirm with reason).

## Validation Rules
- Role order unique; enforce on create/update.
- ShiftDefinition: `durationMinutes > 0`, `startMinutes in [0,1439]`, `code` uppercase alphanumeric length <= 4.
- Monthly hours: if both provided ensure `monthlyMinHours <= monthlyMaxHours`.

## Backward Compatibility
Existing roster logic unaffected; new attributes optional. Constraints referencing roles/gender deferred until engine extension.

## Future Extensions (Not Implemented Now)
- Coverage constraints (min IC per shift, min Female per Night).
- Hour tally & contract violation constraints.
- Skill taxonomy / certifications.

## Testing Strategy (Later)
Unit tests for seed & delete logic; integration tests for new server actions; mapping test ensuring resource attributes includes ordered roles.
