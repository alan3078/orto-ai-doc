# Implementation - FN/BE/DATA/003 - Data Modeling & Engine Integration

## 1. Tech Stack

- **Language**: TypeScript
- **ORM**: Prisma 7.x with `@prisma/adapter-pg`
- **Database**: PostgreSQL 16.x
- **HTTP Client**: Native `fetch` API
- **Validation**: Zod (for API boundary validation)
- **Environment**: Next.js 15 Server Actions

## 2. Database Schema

### 2.1 Prisma Schema Definition

Location: `/web/prisma/schema.prisma`

```prisma
// Core User model (existing)
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

// Staff (Resources)
model Staff {
  id          String   @id @default(cuid())
  employeeId  String   @unique  // External employee ID
  name        String
  email       String?
  isActive    Boolean  @default(true)
  
  shifts      Shift[]
  
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  @@index([employeeId])
  @@index([isActive])
}

// Constraint (Scheduling Rules)
model Constraint {
  id          String   @id @default(cuid())
  name        String   // Human-readable name
  description String?
  
  // JSON representation matching Python API schema
  type        String   // "point" | "vertical_sum"
  config      Json     // Flexible JSON for rule parameters
  
  isActive    Boolean  @default(true)
  priority    Int      @default(0)  // For future ordering
  
  rosters     Roster[]
  
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  @@index([type])
  @@index([isActive])
}

// Roster (Schedule Generation Metadata)
model Roster {
  id          String   @id @default(cuid())
  name        String   // e.g., "Week of 2025-01-20"
  startDate   DateTime
  endDate     DateTime
  
  status      RosterStatus  @default(PENDING)
  
  // Solver metadata
  solveTimeMs Float?
  solverStatus String?  // "OPTIMAL" | "FEASIBLE" | "INFEASIBLE"
  errorMessage String?  @db.Text
  
  // Configuration snapshot
  timeSlots   Int      // Number of time slots (e.g., 7 days)
  states      Json     // [0, 1] or [0, 1, 2] for Off/Work/OnCall
  
  // Relations
  shifts      Shift[]
  constraints Constraint[]
  
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  @@index([status])
  @@index([startDate, endDate])
}

enum RosterStatus {
  PENDING      // Waiting to be processed
  SOLVING      // Currently being solved
  COMPLETED    // Successfully generated
  FAILED       // Solver failed
  INFEASIBLE   // No solution exists
}

// Shift (Individual Schedule Assignment)
model Shift {
  id         String   @id @default(cuid())
  
  rosterId   String
  roster     Roster   @relation(fields: [rosterId], references: [id], onDelete: Cascade)
  
  staffId    String
  staff      Staff    @relation(fields: [staffId], references: [id], onDelete: Cascade)
  
  timeSlot   Int      // 0-indexed position (e.g., day 0, day 1...)
  state      Int      // 0=Off, 1=Work, 2=OnCall, etc.
  date       DateTime // Actual date for this time slot
  
  // Optional metadata
  notes      String?
  
  createdAt  DateTime @default(now())
  
  @@unique([rosterId, staffId, timeSlot])  // Prevent duplicates
  @@index([rosterId, timeSlot])
  @@index([staffId, date])
}
```

### 2.2 Schema Rationale

**Staff**: 
- Uses `cuid()` instead of auto-increment for better distributed ID generation
- `employeeId` for external system integration
- `isActive` for soft deletion

**Constraint**:
- Stores rules in generic JSON format matching Python API
- `type` field for quick filtering
- `isActive` allows disabling without deletion

**Roster**:
- Tracks generation metadata (solve time, status)
- Stores config snapshot for reproducibility
- Many-to-many with Constraints to track which rules were applied

**Shift**:
- Denormalized `date` for easier querying
- Unique constraint prevents duplicate assignments
- Cascade delete ensures orphaned shifts are removed

## 3. Integration Service Architecture

```
┌─────────────────────────────────────────────────┐
│         Server Action (Next.js)                 │
│         generateRosterAction()                  │
└────────────────┬────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────┐
│     SolverIntegrationService                    │
│     generateRoster()                            │
├─────────────────────────────────────────────────┤
│  1. Fetch Staff & Constraints from DB           │
│  2. Create Roster (status: SOLVING)             │
│  3. buildSolverRequest() - Transform data       │
│  4. callSolverApi() - HTTP POST                 │
│  5. saveSolverResults() - Transaction           │
└────────────────┬────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────┐
│      Python Solver Engine                       │
│      POST /api/v1/solve                         │
│      Returns: { status, schedule }              │
└─────────────────────────────────────────────────┘
```

## 4. Zod Validation Schemas

Location: `/web/src/lib/validations/solver.ts`

```typescript
import { z } from 'zod'

export const SolverConfigSchema = z.object({
  resources: z.array(z.string()).min(1),
  time_slots: z.number().positive(),
  states: z.array(z.number()),
})

export const PointConstraintSchema = z.object({
  type: z.literal('point'),
  resource: z.string(),
  time_slot: z.number().nonnegative(),
  state: z.number().nonnegative(),
})

export const VerticalSumConstraintSchema = z.object({
  type: z.literal('vertical_sum'),
  time_slot: z.union([z.number().nonnegative(), z.literal('ALL')]),
  target_state: z.number().nonnegative(),
  operator: z.enum(['>=', '<=', '==']),
  value: z.number().nonnegative(),
})

export const SolverConstraintSchema = z.union([
  PointConstraintSchema,
  VerticalSumConstraintSchema,
])

export const SolverResponseSchema = z.object({
  status: z.enum(['OPTIMAL', 'FEASIBLE', 'INFEASIBLE', 'ERROR']),
  schedule: z.record(z.string(), z.array(z.number())).optional(),
  message: z.string().optional(),
  solve_time_ms: z.number().optional(),
})
```

## 5. Integration Service Implementation

Location: `/web/src/services/solver-integration.service.ts`

### Key Methods:

**`generateRoster()`**
- Main orchestration method
- Handles complete workflow: DB → API → DB
- Wraps in try/catch for error handling

**`buildSolverRequest()`**
- Transforms Prisma models to Python API JSON format
- Maps database IDs to employeeIds
- Validates constraints before sending

**`callSolverApi()`**
- Makes HTTP POST to Python engine
- Uses native `fetch` API
- Validates response with Zod

**`saveSolverResults()`**
- Parses solver response
- Creates Shift records
- Updates Roster status
- Uses Prisma transaction for atomicity

## 6. Data Transformation Logic

### Database Constraint → Python API Format

```typescript
// Database: Constraint model
{
  type: 'point',
  config: {
    resource: 'EMP001',
    time_slot: 0,
    state: 0
  }
}

// Transformed to Python API:
{
  type: 'point',
  resource: 'EMP001',
  time_slot: 0,
  state: 0
}
```

### Python Response → Database Shifts

```typescript
// Python returns:
{
  schedule: {
    'EMP001': [0, 1, 1, 1, 0, 1, 1],
    'EMP002': [1, 1, 0, 1, 1, 0, 0]
  }
}

// Transform to Shift records:
[
  { rosterId, staffId: 'staff-1', timeSlot: 0, state: 0, date: '2025-01-20' },
  { rosterId, staffId: 'staff-1', timeSlot: 1, state: 1, date: '2025-01-21' },
  // ... etc
]
```

## 7. Error Handling Strategy

### API Errors
```typescript
try {
  const response = await fetch(engineUrl, {...})
  if (!response.ok) {
    throw new Error(`Solver API error: ${response.statusText}`)
  }
} catch (error) {
  await prisma.roster.update({
    where: { id: rosterId },
    data: { status: 'FAILED', errorMessage: error.message }
  })
}
```

### Infeasible Solutions
```typescript
if (solverResponse.status === 'INFEASIBLE') {
  await prisma.roster.update({
    where: { id: rosterId },
    data: {
      status: 'INFEASIBLE',
      errorMessage: 'No feasible solution found'
    }
  })
  return // Don't create shifts
}
```

### Transaction Failures
```typescript
await prisma.$transaction([
  prisma.roster.update({...}),
  prisma.shift.createMany({...})
])
// If either fails, both are rolled back
```

## 8. Environment Configuration

```env
# Database
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/orto_dev?schema=public"

# Solver Engine
SOLVER_ENGINE_URL="http://localhost:8000"
```

## 9. Server Action Layer

Location: `/web/src/app/actions/generate-roster.action.ts`

### Key Functions:

**`generateRosterAction(formData: FormData)`**
- Next.js Server Action with `'use server'` directive
- Validates form inputs (name, startDate, timeSlots)
- Fetches active staff and constraints
- Calls `SolverIntegrationService.generateRoster()`
- Uses `revalidatePath()` for cache invalidation
- Returns success/error response

**`getRostersAction()`**
- Fetches recent rosters with shift count
- Returns list ordered by creation date

**`getRosterByIdAction(rosterId: string)`**
- Fetches specific roster with all shifts and staff details
- Returns detailed roster object or error

## 10. Testing Approach

### Manual Testing Script

Location: `/web/scripts/test-solver-integration.ts`

```bash
# Prerequisites:
# 1. Start Python engine: cd engine && uv run uvicorn src.main:app --reload
# 2. Seed database: cd web && npx prisma db seed

# Run test:
npx tsx scripts/test-solver-integration.ts
```

**Test Output**:
- ✅ Staff and constraints count
- ✅ Roster generation with timing
- ✅ Schedule matrix visualization
- ✅ Constraint verification (Alice off Monday, min coverage)
- ✅ Success/failure status

**What it tests**:
- Database queries (fetch staff/constraints)
- Solver integration (API call)
- Data transformation (DB → API → DB)
- Transaction integrity
- Constraint satisfaction

### Verification Steps

1. **Check Roster status**: 
   ```bash
   npx prisma studio
   # Navigate to: Roster table → status should be "COMPLETED"
   ```

2. **Check Shift creation**:
   ```bash
   # Should have: num_staff × time_slots records
   # Example: 5 staff × 7 days = 35 shifts
   ```

3. **Verify constraints**:
   - Alice (EMP001) should be off on Monday (timeSlot=0, state=0)
   - Each day should have ≥3 staff working (state=1)

4. **Test INFEASIBLE scenario**:
   ```typescript
   // Add conflicting constraints:
   // - Alice must work Monday (state=1)
   // - Alice must be off Monday (state=0)
   // Expected: Roster status = "INFEASIBLE", no shifts created
   ```

5. **Test error handling**:
   ```bash
   # Stop Python engine
   # Run test → should fail gracefully with FAILED status
   ```

## 11. Migration Commands

```bash
# Generate migration
npx prisma migrate dev --name add_scheduling_models

# Apply migration (production)
npx prisma migrate deploy

# Regenerate Prisma Client
npx prisma generate

# Seed database
npx prisma db seed

# Inspect data
npx prisma studio
```

## 12. Implementation Status

✅ **Completed Tasks**:
1. Prisma schema defined (Staff, Constraint, Roster, Shift models)
2. Migration created and applied: `20251120154729_add_scheduling_models`
3. Seed script created with sample data (5 staff, 2 constraints)
4. Zod validation schemas (`/web/src/lib/validations/solver.ts`)
5. Integration service (`/web/src/services/solver-integration.service.ts`)
6. Server Actions (`/web/src/app/actions/generate-roster.action.ts`)
7. Test script (`/web/scripts/test-solver-integration.ts`)

🔄 **Pending Tasks**:
- End-to-end testing (requires Python engine running)
- INFEASIBLE scenario testing
- Error handling validation
- Documentation updates (.env.example, README)

---

**Next**: See `task.md` for step-by-step implementation checklist.
