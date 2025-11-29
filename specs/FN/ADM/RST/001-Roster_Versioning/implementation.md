# Implementation - FN/ADM/RST/001 - Roster Versioning

---

## 1. Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        orto-ai-app                              │
├─────────────────────────────────────────────────────────────────┤
│  Prisma Schema          │  Server Actions       │  UI Layer    │
│  - Roster.version       │  - getRosterVersions  │  - Confirm   │
│  - Roster.deletedAt     │  - deleteRosterVersion│    Modal     │
│  - Shift.deletedAt      │  - generateRoster*    │  - Version   │
│                         │  - getRoster*         │    Selector  │
└────────────┬────────────┴──────────┬────────────┴──────┬───────┘
             │                       │                   │
             ▼                       ▼                   ▼
┌─────────────────────────────────────────────────────────────────┐
│                         Database                                │
├─────────────────────────────────────────────────────────────────┤
│  roster table                      │  shift table               │
│  - version INT DEFAULT 1           │  - deleted_at TIMESTAMP    │
│  - deleted_at TIMESTAMP            │                            │
│  - UNIQUE(start_date, end_date,    │                            │
│           shift_type, version)     │                            │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Database Schema Changes

### 2.1 Roster Model Extension

**File:** `orto-ai-app/prisma/schema.prisma`

```prisma
model Roster {
  id            String       @id @default(cuid())
  name          String
  startDate     DateTime     @map("start_date")
  endDate       DateTime     @map("end_date")
  status        RosterStatus @default(PENDING)
  shiftType     ShiftType    @default(APN) @map("shift_type")
  version       Int          @default(1)
  deletedAt     DateTime?    @map("deleted_at")
  solveTimeMs   Float?       @map("solve_time_ms")
  solverStatus  String?      @map("solver_status")
  errorMessage  String?      @db.Text @map("error_message")
  timeSlots     Int          @map("time_slots")
  states        Json
  shifts        Shift[]
  constraints   Constraint[]
  createdAt     DateTime     @default(now()) @map("created_at")
  updatedAt     DateTime     @updatedAt @map("updated_at")

  @@unique([startDate, endDate, shiftType, version])
  @@map("roster")
}
```

### 2.2 Shift Model Extension

```prisma
model Shift {
  id         String    @id @default(cuid())
  rosterId   String    @map("roster_id")
  roster     Roster    @relation(fields: [rosterId], references: [id], onDelete: Cascade)
  staffId    String    @map("staff_id")
  staff      Staff     @relation(fields: [staffId], references: [id], onDelete: Cascade)
  timeSlot   Int       @map("time_slot")
  state      Int
  date       DateTime
  notes      String?
  isIC       Boolean   @default(false) @map("is_ic")
  deletedAt  DateTime? @map("deleted_at")
  createdAt  DateTime  @default(now()) @map("created_at")

  @@unique([rosterId, staffId, timeSlot])
  @@index([rosterId, timeSlot])
  @@index([staffId, date])
  @@map("shift")
}
```

---

## 3. Server Actions Implementation

### 3.1 Get Roster Versions Action

**File:** `orto-ai-app/src/app/actions/roster.actions.ts`

```typescript
export async function getRosterVersionsAction(params: { month: string }) {
  const { month } = params
  const { monthStart, monthEnd } = getMonthBounds(month)

  const versions = await prisma.roster.findMany({
    where: {
      startDate: { gte: monthStart },
      endDate: { lte: monthEnd },
      deletedAt: null,
    },
    select: {
      id: true,
      version: true,
      status: true,
      shiftType: true,
      createdAt: true,
      solverStatus: true,
    },
    orderBy: { version: 'asc' },
  })

  return versions
}
```

### 3.2 Delete Roster Version Action

```typescript
export async function deleteRosterVersionAction(rosterId: string) {
  const now = new Date()

  await prisma.$transaction([
    prisma.shift.updateMany({
      where: { rosterId },
      data: { deletedAt: now },
    }),
    prisma.roster.update({
      where: { id: rosterId },
      data: { deletedAt: now },
    }),
  ])

  revalidatePath('/admin/roster')
  return { success: true }
}
```

### 3.3 Modified Generate Roster Action

Key changes:
1. Remove hard-delete of existing rosters
2. Calculate next available version (1-3)
3. Throw error if 3 versions exist
4. Add version field when creating roster

```typescript
export async function generateRosterAction(params: GenerateRosterInput) {
  const { month, shiftType } = params
  const { monthStart, monthEnd, timeSlots, dates } = getMonthBounds(month)

  // Find existing active versions
  const existingVersions = await prisma.roster.findMany({
    where: {
      startDate: { gte: monthStart },
      endDate: { lte: monthEnd },
      shiftType,
      deletedAt: null,
    },
    select: { version: true },
  })

  // Calculate next version
  const usedVersions = new Set(existingVersions.map(r => r.version))
  let nextVersion: number | null = null
  for (let v = 1; v <= 3; v++) {
    if (!usedVersions.has(v)) {
      nextVersion = v
      break
    }
  }

  if (nextVersion === null) {
    throw new Error('Maximum 3 versions reached. Please delete a version first.')
  }

  // Create roster with version
  const roster = await prisma.roster.create({
    data: {
      name: `${month} Roster v${nextVersion}`,
      startDate: monthStart,
      endDate: monthEnd,
      shiftType,
      version: nextVersion,
      timeSlots,
      states: statesConfig,
      status: 'PENDING',
    },
  })

  // ... rest of generation logic
}
```

### 3.4 Modified Get Roster Action

```typescript
export async function getRosterAction(params: { month: string; version?: number }) {
  const { month, version } = params
  const { monthStart, monthEnd } = getMonthBounds(month)

  const roster = await prisma.roster.findFirst({
    where: {
      startDate: { gte: monthStart },
      endDate: { lte: monthEnd },
      deletedAt: null,
      ...(version ? { version } : {}),
    },
    orderBy: { version: 'desc' }, // Latest version if not specified
    include: {
      shifts: {
        where: { deletedAt: null },
        include: { staff: true },
      },
    },
  })

  return roster
}
```

---

## 4. Frontend Components

### 4.1 RosterVersionConfirmModal

**File:** `orto-ai-app/src/features/dashboard/components/roster-version-confirm-modal.tsx`

```tsx
interface RosterVersionConfirmModalProps {
  open: boolean
  onOpenChange: (open: boolean) => void
  existingVersions: number[]
  nextVersion: number
  onConfirm: () => void
  isLoading?: boolean
}

export function RosterVersionConfirmModal({
  open,
  onOpenChange,
  existingVersions,
  nextVersion,
  onConfirm,
  isLoading,
}: RosterVersionConfirmModalProps) {
  return (
    <Dialog open={open} onOpenChange={onOpenChange}>
      <DialogContent>
        <DialogHeader>
          <DialogTitle>Generate New Version</DialogTitle>
          <DialogDescription>
            You already have {existingVersions.length} version(s) for this month.
          </DialogDescription>
        </DialogHeader>
        <div className="py-4">
          <p>This will create <strong>Version {nextVersion}</strong>.</p>
          <p className="text-sm text-muted-foreground mt-2">
            Maximum 3 versions allowed per month.
          </p>
        </div>
        <DialogFooter>
          <Button variant="outline" onClick={() => onOpenChange(false)}>
            Cancel
          </Button>
          <Button onClick={onConfirm} disabled={isLoading}>
            {isLoading ? 'Generating...' : 'Generate'}
          </Button>
        </DialogFooter>
      </DialogContent>
    </Dialog>
  )
}
```

### 4.2 RosterVersionSelector

**File:** `orto-ai-app/src/features/dashboard/components/roster-version-selector.tsx`

```tsx
interface RosterVersion {
  id: string
  version: number
  status: RosterStatus
  createdAt: Date
}

interface RosterVersionSelectorProps {
  versions: RosterVersion[]
  selectedVersion: number
  onSelectVersion: (version: number) => void
  onDeleteVersion: (rosterId: string) => void
  isDeleting?: boolean
}

export function RosterVersionSelector({
  versions,
  selectedVersion,
  onSelectVersion,
  onDeleteVersion,
  isDeleting,
}: RosterVersionSelectorProps) {
  return (
    <div className="flex items-center gap-2">
      <Select
        value={String(selectedVersion)}
        onValueChange={(v) => onSelectVersion(Number(v))}
      >
        <SelectTrigger className="w-[140px]">
          <SelectValue placeholder="Version" />
        </SelectTrigger>
        <SelectContent>
          {versions.map((v) => (
            <SelectItem key={v.id} value={String(v.version)}>
              <div className="flex items-center gap-2">
                <span>Version {v.version}</span>
                <Badge variant={v.status === 'COMPLETED' ? 'default' : 'secondary'}>
                  {v.status}
                </Badge>
              </div>
            </SelectItem>
          ))}
        </SelectContent>
      </Select>
      
      {versions.length > 0 && (
        <Button
          variant="ghost"
          size="icon"
          onClick={() => {
            const current = versions.find(v => v.version === selectedVersion)
            if (current) onDeleteVersion(current.id)
          }}
          disabled={isDeleting}
        >
          <Trash2 className="h-4 w-4 text-destructive" />
        </Button>
      )}
    </div>
  )
}
```

---

## 5. Hooks Implementation

### 5.1 useRosterVersions Hook

**File:** `orto-ai-app/src/features/dashboard/hooks/use-roster.ts`

```typescript
export const rosterKeys = {
  all: ['roster'] as const,
  month: (month: string) => [...rosterKeys.all, { month }] as const,
  versions: (month: string) => [...rosterKeys.all, 'versions', { month }] as const,
  detail: (month: string, version?: number) => 
    [...rosterKeys.all, 'detail', { month, version }] as const,
}

export function useRosterVersions(month: string) {
  return useQuery({
    queryKey: rosterKeys.versions(month),
    queryFn: () => getRosterVersionsAction({ month }),
    enabled: !!month,
  })
}

export function useDeleteRosterVersion() {
  const queryClient = useQueryClient()
  
  return useMutation({
    mutationFn: deleteRosterVersionAction,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: rosterKeys.all })
    },
  })
}
```

---

## 6. Migration Script

```sql
-- Add version column to roster table
ALTER TABLE roster ADD COLUMN version INT NOT NULL DEFAULT 1;

-- Add deleted_at column to roster table
ALTER TABLE roster ADD COLUMN deleted_at TIMESTAMP;

-- Add deleted_at column to shift table
ALTER TABLE shift ADD COLUMN deleted_at TIMESTAMP;

-- Add unique constraint
ALTER TABLE roster ADD CONSTRAINT roster_start_date_end_date_shift_type_version_key 
  UNIQUE (start_date, end_date, shift_type, version);

-- Create index for soft-delete queries
CREATE INDEX roster_deleted_at_idx ON roster (deleted_at);
CREATE INDEX shift_deleted_at_idx ON shift (deleted_at);
```
