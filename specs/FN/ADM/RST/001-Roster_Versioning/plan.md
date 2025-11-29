# FN/ADM/RST/001 - Roster Versioning

**Feature Type:** Roster Management  
**Status:** In Progress  
**Priority:** High  
**Dependencies:** FN/BE/DATA/003-Data_Modeling_Engine_Integration  

---

## 1. Overview

This feature implements roster versioning, allowing up to 3 versions of a roster per month. Each time a roster is generated, if existing versions exist, a confirmation modal is shown. Users can view, switch between, and soft-delete versions from the UI.

---

## 2. Problem Statement

Currently, generating a new roster for a month **hard-deletes** any existing roster for that period. This leads to:
- Loss of previously generated schedules
- No ability to compare different scheduling options
- No history of past roster attempts

Hospital administrators need:
- Multiple roster versions to compare scheduling options
- Ability to keep alternative schedules before finalizing
- Safe deletion with soft-delete for audit/recovery purposes

---

## 3. Scope

### In Scope
1. **Version Field** - Add `version` column (1-3) to Roster model
2. **Soft Delete** - Add `deletedAt` to both Roster and Shift models
3. **Max 3 Versions** - Enforce maximum of 3 active versions per month
4. **Confirmation Modal** - Show dialog when generating with existing versions
5. **Version Selector** - UI component to list and switch between versions
6. **Delete Version** - Soft-delete a version (sets deletedAt on roster + shifts)
7. **Filter Deleted** - All queries exclude soft-deleted records

### Out of Scope (Future)
- Version comparison side-by-side view
- Version naming/labeling
- Hard-delete cleanup job for old soft-deleted records
- Version restore functionality

---

## 4. Success Criteria

1. ✅ Prisma schema extended with `version` and `deletedAt` on Roster and Shift
2. ✅ Unique constraint prevents duplicate versions for same month/shiftType
3. ✅ `generateRosterAction` creates new version (1-3) without deleting existing
4. ✅ Error thrown if user tries to generate 4th version
5. ✅ Confirmation modal appears when generating with existing versions
6. ✅ Version selector displays all active versions with status badges
7. ✅ Delete button soft-deletes roster and its shifts
8. ✅ All roster queries filter out soft-deleted records

---

## 5. Technical Approach

### 5.1 Database Changes

**Roster Model:**
```prisma
model Roster {
  // ... existing fields
  version     Int       @default(1)
  deletedAt   DateTime? @map("deleted_at")
  
  @@unique([startDate, endDate, shiftType, version])
}
```

**Shift Model:**
```prisma
model Shift {
  // ... existing fields
  deletedAt   DateTime? @map("deleted_at")
}
```

### 5.2 Version Calculation Logic

```typescript
// Find existing active versions for the month
const existingVersions = await prisma.roster.findMany({
  where: {
    startDate: { gte: monthStart },
    endDate: { lte: monthEnd },
    shiftType,
    deletedAt: null,
  },
  select: { version: true },
  orderBy: { version: 'asc' },
})

// Find next available version (1-3)
const usedVersions = new Set(existingVersions.map(r => r.version))
let nextVersion = null
for (let v = 1; v <= 3; v++) {
  if (!usedVersions.has(v)) {
    nextVersion = v
    break
  }
}

if (nextVersion === null) {
  throw new Error('Maximum 3 versions reached. Delete a version first.')
}
```

### 5.3 Soft Delete Implementation

```typescript
async function deleteRosterVersion(rosterId: string) {
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
}
```

---

## 6. Dependencies

| Component | Dependency | Status |
|-----------|------------|--------|
| Prisma | Database ORM | ✅ Installed |
| shadcn/ui Dialog | Confirmation modal | ✅ Available |
| TanStack Query | Data fetching hooks | ✅ Installed |
