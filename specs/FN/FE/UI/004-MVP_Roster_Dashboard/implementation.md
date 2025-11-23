````markdown
# Implementation - FN/FE/UI/004 - MVP Roster Dashboard

## 1. Tech Stack

- **Framework**: Next.js 15 (App Router)
- **Language**: TypeScript 5.x
- **State Management**: TanStack Query (React Query) v5
- **Forms**: react-hook-form + Zod validation
- **UI Components**: Shadcn UI (Radix primitives)
- **Styling**: Tailwind CSS 3.x
- **Icons**: Lucide React
- **Date Handling**: date-fns v3

## 2. Page Architecture

### Route Structure

```
app/
└── dashboard/
    └── page.tsx          // Main dashboard page (Server Component)
```

### Layout Composition

```tsx
<div className="container mx-auto p-6">
  {/* Header */}
  <div className="flex justify-between items-center mb-6">
    <h1>Roster Dashboard</h1>
    <Button variant="outline">
      <Download className="mr-2 h-4 w-4" />
      Export CSV
    </Button>
  </div>

  {/* Main Grid Layout */}
  <div className="grid grid-cols-12 gap-6">
    {/* Sidebar - 3 columns */}
    <aside className="col-span-12 lg:col-span-3 space-y-6">
      <StaffList />
      <ConstraintBuilder />
    </aside>

    {/* Main Content - 9 columns */}
    <main className="col-span-12 lg:col-span-9">
      <RosterControls />
      <RosterGrid />
    </main>
  </div>
</div>
```

**Responsive Breakpoints**:
- Mobile (<768px): Single column, stack vertically
- Tablet (768-1023px): Single column, collapsible sections
- Desktop (≥1024px): 3-9 split as shown above

## 3. Feature Folder Structure

```
src/
└── features/
    └── dashboard/
        ├── components/
        │   ├── staff-list.tsx           // Client component
        │   ├── add-staff-dialog.tsx     // Client component
        │   ├── constraint-builder.tsx   // Client component
        │   ├── add-constraint-form.tsx  // Client component
        │   ├── roster-controls.tsx      // Client component
        │   ├── roster-grid.tsx          // Client component
        │   └── state-badge.tsx          // Reusable badge
        ├── hooks/
        │   ├── use-staff.ts             // useStaff, useAddStaff, useDeleteStaff
        │   ├── use-constraints.ts       // useConstraints, useAddConstraint, useDeleteConstraint
        │   └── use-roster.ts            // useRoster, useGenerateRoster
        └── services/
            └── dashboard.service.ts     // DTOs, types, schemas
```

## 4. Data Architecture

### React Query Configuration

Query keys using factory pattern:

```typescript
// src/features/dashboard/services/dashboard.service.ts

export const staffKeys = {
  all: ['staff'] as const,
  lists: () => [...staffKeys.all, 'list'] as const,
  list: (filters: string) => [...staffKeys.lists(), { filters }] as const,
}

export const constraintKeys = {
  all: ['constraints'] as const,
  lists: () => [...constraintKeys.all, 'list'] as const,
}

export const rosterKeys = {
  all: ['roster'] as const,
  month: (month: string) => [...rosterKeys.all, { month }] as const,
  detail: (id: string) => [...rosterKeys.all, 'detail', id] as const,
}
```

### Mutation Flows

#### Add Staff Flow
```
User clicks "Add Staff" 
  → Dialog opens
  → User fills form (name, employeeId, email)
  → Form validates with Zod
  → onSubmit calls addStaff.mutate()
  → Optimistic update: Add to cache immediately
  → API call: POST /api/staff
  → On success: Invalidate ['staff']
  → On error: Rollback optimistic update
  → Toast notification
  → Dialog closes
```

#### Generate Roster Flow
```
User clicks "Generate Roster"
  → Validate: staffIds.length > 0
  → Set local loading state
  → Call generateRoster.mutate()
  → Server Action creates Roster (status: SOLVING)
  → Immediately: Invalidate ['roster', { month }]
  → Start polling (every 2s)
  → When status !== 'SOLVING': Stop polling
  → Display result in RosterGrid
  → Toast: "Generated!" or "Failed"
```

## 5. Component Specifications

### 5.1 StaffList Component

**File**: `src/features/dashboard/components/staff-list.tsx`

```tsx
'use client'

import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'
import { Button } from '@/components/ui/button'
import { Plus, Trash2 } from 'lucide-react'
import { useStaff, useDeleteStaff } from '../hooks/use-staff'
import { AddStaffDialog } from './add-staff-dialog'
import { useState } from 'react'

export function StaffList() {
  const { data: staff, isLoading } = useStaff()
  const deleteStaff = useDeleteStaff()
  const [dialogOpen, setDialogOpen] = useState(false)

  const handleDelete = (id: string, name: string) => {
    if (confirm(`Delete ${name}?`)) {
      deleteStaff.mutate(id)
    }
  }

  return (
    <Card>
      <CardHeader>
        <CardTitle className="flex justify-between items-center">
          <span>Staff Members</span>
          <Button 
            size="sm" 
            variant="outline"
            onClick={() => setDialogOpen(true)}
          >
            <Plus className="h-4 w-4 mr-1" />
            Add
          </Button>
        </CardTitle>
      </CardHeader>
      <CardContent>
        {isLoading ? (
          <div>Loading...</div>
        ) : staff?.length === 0 ? (
          <p className="text-sm text-muted-foreground">
            No staff added yet.
          </p>
        ) : (
          <div className="space-y-2 max-h-[400px] overflow-y-auto">
            {staff?.map((s) => (
              <div 
                key={s.id}
                className="flex justify-between items-center p-2 rounded border"
              >
                <div>
                  <p className="font-medium">{s.name}</p>
                  <p className="text-xs text-muted-foreground">{s.employeeId}</p>
                </div>
                <Button
                  size="icon"
                  variant="ghost"
                  onClick={() => handleDelete(s.id, s.name)}
                >
                  <Trash2 className="h-4 w-4 text-destructive" />
                </Button>
              </div>
            ))}
          </div>
        )}
      </CardContent>
      
      <AddStaffDialog open={dialogOpen} onOpenChange={setDialogOpen} />
    </Card>
  )
}
```

### 5.2 ConstraintBuilder Component

**File**: `src/features/dashboard/components/constraint-builder.tsx`

```tsx
'use client'

import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'
import { Button } from '@/components/ui/button'
import { Badge } from '@/components/ui/badge'
import { DropdownMenu, DropdownMenuContent, DropdownMenuItem, DropdownMenuTrigger } from '@/components/ui/dropdown-menu'
import { Plus, Trash2 } from 'lucide-react'
import { useConstraints, useDeleteConstraint } from '../hooks/use-constraints'
import { AddConstraintForm } from './add-constraint-form'
import { useState } from 'react'

type ConstraintType = 'point' | 'vertical_sum' | null

export function ConstraintBuilder() {
  const { data: constraints, isLoading } = useConstraints()
  const deleteConstraint = useDeleteConstraint()
  const [formType, setFormType] = useState<ConstraintType>(null)

  return (
    <Card>
      <CardHeader>
        <CardTitle className="flex justify-between items-center">
          <span>Active Constraints</span>
          <DropdownMenu>
            <DropdownMenuTrigger asChild>
              <Button size="sm" variant="outline">
                <Plus className="h-4 w-4 mr-1" />
                Add Rule
              </Button>
            </DropdownMenuTrigger>
            <DropdownMenuContent>
              <DropdownMenuItem onClick={() => setFormType('point')}>
                Day Off (Point)
              </DropdownMenuItem>
              <DropdownMenuItem onClick={() => setFormType('vertical_sum')}>
                Min/Max Workers (Sum)
              </DropdownMenuItem>
            </DropdownMenuContent>
          </DropdownMenu>
        </CardTitle>
      </CardHeader>
      <CardContent className="space-y-4">
        {/* Add Constraint Form */}
        {formType && (
          <AddConstraintForm 
            type={formType} 
            onCancel={() => setFormType(null)}
            onSuccess={() => setFormType(null)}
          />
        )}

        {/* Constraints List */}
        {isLoading ? (
          <div>Loading...</div>
        ) : constraints?.length === 0 ? (
          <p className="text-sm text-muted-foreground">
            No constraints added yet.
          </p>
        ) : (
          <div className="space-y-2 max-h-[300px] overflow-y-auto">
            {constraints?.map((c) => (
              <div 
                key={c.id}
                className="flex justify-between items-center p-2 rounded border"
              >
                <div className="flex-1">
                  <Badge variant={c.type === 'point' ? 'secondary' : 'default'}>
                    {c.type}
                  </Badge>
                  <p className="text-sm mt-1">{c.name}</p>
                </div>
                <Button
                  size="icon"
                  variant="ghost"
                  onClick={() => deleteConstraint.mutate(c.id)}
                >
                  <Trash2 className="h-4 w-4 text-destructive" />
                </Button>
              </div>
            ))}
          </div>
        )}
      </CardContent>
    </Card>
  )
}
```

### 5.3 RosterGrid Component

**File**: `src/features/dashboard/components/roster-grid.tsx`

```tsx
'use client'

import { Card, CardContent } from '@/components/ui/card'
import { Skeleton } from '@/components/ui/skeleton'
import { Alert, AlertDescription, AlertTitle } from '@/components/ui/alert'
import { AlertCircle, AlertTriangle, Calendar } from 'lucide-react'
import { useRoster } from '../hooks/use-roster'
import { StateBadge } from './state-badge'
import { format, addDays } from 'date-fns'

interface RosterGridProps {
  month: string // "2025-11"
}

export function RosterGrid({ month }: RosterGridProps) {
  const { data: roster, isLoading } = useRoster({ month })

  if (isLoading || roster?.status === 'SOLVING') {
    return <Skeleton className="h-[400px] w-full" />
  }

  if (!roster) {
    return (
      <Card className="flex flex-col items-center justify-center h-[400px]">
        <Calendar className="h-16 w-16 text-muted-foreground mb-4" />
        <p className="text-lg font-medium">No roster generated</p>
        <p className="text-sm text-muted-foreground">
          Add staff and constraints, then click "Generate Roster"
        </p>
      </Card>
    )
  }

  if (roster.status === 'FAILED') {
    return (
      <Alert variant="destructive">
        <AlertCircle className="h-4 w-4" />
        <AlertTitle>Generation Failed</AlertTitle>
        <AlertDescription>{roster.errorMessage}</AlertDescription>
      </Alert>
    )
  }

  if (roster.status === 'INFEASIBLE') {
    return (
      <Alert>
        <AlertTriangle className="h-4 w-4" />
        <AlertTitle>No Feasible Solution</AlertTitle>
        <AlertDescription>
          The constraints cannot be satisfied. Try adjusting rules.
        </AlertDescription>
      </Alert>
    )
  }

  // Group shifts by staff
  const shiftsByStaff = roster.shifts.reduce((acc, shift) => {
    if (!acc[shift.staffId]) {
      acc[shift.staffId] = {
        staff: shift.staff,
        shifts: Array(roster.timeSlots).fill(null),
      }
    }
    acc[shift.staffId].shifts[shift.timeSlot] = shift
    return acc
  }, {} as Record<string, { staff: any; shifts: any[] }>)

  return (
    <Card>
      <CardContent className="p-6">
        <div className="overflow-x-auto">
          <table className="w-full border-collapse">
            <thead>
              <tr>
                <th className="sticky left-0 bg-white p-2 text-left border">
                  Staff
                </th>
                {Array.from({ length: roster.timeSlots }).map((_, i) => (
                  <th key={i} className="p-2 text-center border min-w-[80px]">
                    <div>Day {i}</div>
                    <div className="text-xs text-muted-foreground">
                      {format(addDays(new Date(roster.startDate), i), 'MMM d')}
                    </div>
                  </th>
                ))}
              </tr>
            </thead>
            <tbody>
              {Object.values(shiftsByStaff).map(({ staff, shifts }) => (
                <tr key={staff.id}>
                  <td className="sticky left-0 bg-white p-2 font-medium border">
                    {staff.name}
                  </td>
                  {shifts.map((shift, i) => (
                    <td key={i} className="p-2 text-center border">
                      {shift ? <StateBadge state={shift.state} /> : '-'}
                    </td>
                  ))}
                </tr>
              ))}
            </tbody>
          </table>
        </div>
        
        {/* Metadata */}
        <div className="mt-4 text-sm text-muted-foreground">
          Status: {roster.solverStatus} • 
          Generated in {roster.solveTimeMs}ms
        </div>
      </CardContent>
    </Card>
  )
}
```

### 5.4 StateBadge Component

**File**: `src/features/dashboard/components/state-badge.tsx`

```tsx
import { Badge } from '@/components/ui/badge'

interface StateBadgeProps {
  state: number
}

const STATE_CONFIG = {
  0: { bg: 'bg-slate-100', text: 'text-slate-700', label: 'Off', icon: '○' },
  1: { bg: 'bg-blue-100', text: 'text-blue-700', label: 'Work', icon: '●' },
  2: { bg: 'bg-purple-100', text: 'text-purple-700', label: 'OnCall', icon: '◐' },
}

export function StateBadge({ state }: StateBadgeProps) {
  const config = STATE_CONFIG[state as keyof typeof STATE_CONFIG] || STATE_CONFIG[0]
  
  return (
    <Badge 
      className={`${config.bg} ${config.text} border-0`}
      variant="secondary"
    >
      {config.icon}
    </Badge>
  )
}
```

## 6. React Query Hooks

### 6.1 useStaff Hook

**File**: `src/features/dashboard/hooks/use-staff.ts`

```typescript
'use client'

import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query'
import { toast } from 'sonner'
import { getStaffAction, createStaffAction, deleteStaffAction } from '@/app/actions/staff.actions'
import { staffKeys } from '../services/dashboard.service'

export function useStaff() {
  return useQuery({
    queryKey: staffKeys.all,
    queryFn: async () => {
      const result = await getStaffAction()
      if (!result.success) throw new Error(result.error)
      return result.staff
    },
    staleTime: 5 * 60 * 1000, // 5 minutes
  })
}

export function useAddStaff() {
  const queryClient = useQueryClient()
  
  return useMutation({
    mutationFn: async (data: { name: string; employeeId: string; email?: string }) => {
      const result = await createStaffAction(data)
      if (!result.success) throw new Error(result.error)
      return result.staff
    },
    onMutate: async (newStaff) => {
      // Optimistic update
      await queryClient.cancelQueries({ queryKey: staffKeys.all })
      const previous = queryClient.getQueryData(staffKeys.all)
      
      queryClient.setQueryData(staffKeys.all, (old: any[]) => [
        ...old,
        { ...newStaff, id: 'temp-' + Date.now(), createdAt: new Date() }
      ])
      
      return { previous }
    },
    onError: (err, newStaff, context) => {
      queryClient.setQueryData(staffKeys.all, context?.previous)
      toast.error('Failed to add staff: ' + err.message)
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: staffKeys.all })
      toast.success('Staff added successfully!')
    },
  })
}

export function useDeleteStaff() {
  const queryClient = useQueryClient()
  
  return useMutation({
    mutationFn: async (id: string) => {
      const result = await deleteStaffAction(id)
      if (!result.success) throw new Error(result.error)
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: staffKeys.all })
      toast.success('Staff deleted')
    },
    onError: (err) => {
      toast.error('Failed to delete staff: ' + err.message)
    },
  })
}
```

### 6.2 useRoster Hook with Polling

**File**: `src/features/dashboard/hooks/use-roster.ts`

```typescript
'use client'

import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query'
import { toast } from 'sonner'
import { generateRosterAction, getRosterAction } from '@/app/actions/generate-roster.action'
import { rosterKeys } from '../services/dashboard.service'

export function useRoster({ month }: { month: string }) {
  return useQuery({
    queryKey: rosterKeys.month(month),
    queryFn: async () => {
      const result = await getRosterAction({ month })
      if (!result.success) throw new Error(result.error)
      return result.roster
    },
    refetchInterval: (query) => {
      // Poll every 2s if status is SOLVING
      const roster = query.state.data
      return roster?.status === 'SOLVING' ? 2000 : false
    },
    staleTime: 0, // Always fresh
  })
}

export function useGenerateRoster() {
  const queryClient = useQueryClient()
  
  return useMutation({
    mutationFn: async (params: {
      name: string
      startDate: string
      timeSlots: number
      staffIds: string[]
      constraintIds: string[]
    }) => {
      const result = await generateRosterAction(params)
      if (!result.success) throw new Error(result.error)
      return result
    },
    onSuccess: (data, variables) => {
      // Extract month from startDate
      const month = variables.startDate.substring(0, 7) // "2025-11"
      queryClient.invalidateQueries({ queryKey: rosterKeys.month(month) })
      toast.success('Roster generation started!')
    },
    onError: (err) => {
      toast.error('Failed to generate roster: ' + err.message)
    },
  })
}
```

## 7. Server Actions

### 7.1 Staff Actions

**File**: `src/app/actions/staff.actions.ts`

```typescript
'use server'

import { revalidatePath } from 'next/cache'
import { prisma } from '@/lib/prisma'

export async function getStaffAction() {
  try {
    const staff = await prisma.staff.findMany({
      where: { isActive: true },
      orderBy: { name: 'asc' },
    })
    return { success: true, staff }
  } catch (error) {
    return { 
      success: false, 
      error: error instanceof Error ? error.message : 'Unknown error',
      staff: []
    }
  }
}

export async function createStaffAction(data: {
  name: string
  employeeId: string
  email?: string
}) {
  try {
    // Check for duplicate employeeId
    const existing = await prisma.staff.findUnique({
      where: { employeeId: data.employeeId },
    })
    
    if (existing) {
      return { 
        success: false, 
        error: 'Employee ID already exists',
        staff: null
      }
    }
    
    const staff = await prisma.staff.create({ 
      data: {
        name: data.name,
        employeeId: data.employeeId,
        email: data.email || null,
      }
    })
    
    revalidatePath('/dashboard')
    return { success: true, staff }
  } catch (error) {
    return { 
      success: false, 
      error: error instanceof Error ? error.message : 'Unknown error',
      staff: null
    }
  }
}

export async function deleteStaffAction(id: string) {
  try {
    // Soft delete
    await prisma.staff.update({
      where: { id },
      data: { isActive: false },
    })
    
    revalidatePath('/dashboard')
    return { success: true }
  } catch (error) {
    return { 
      success: false, 
      error: error instanceof Error ? error.message : 'Unknown error'
    }
  }
}
```

### 7.2 Constraint Actions

**File**: `src/app/actions/constraints.actions.ts`

```typescript
'use server'

import { revalidatePath } from 'next/cache'
import { prisma } from '@/lib/prisma'

export async function getConstraintsAction() {
  try {
    const constraints = await prisma.constraint.findMany({
      where: { isActive: true },
      orderBy: { priority: 'desc' },
    })
    return { success: true, constraints }
  } catch (error) {
    return { 
      success: false, 
      error: error instanceof Error ? error.message : 'Unknown error',
      constraints: []
    }
  }
}

export async function createConstraintAction(data: {
  name: string
  type: 'point' | 'vertical_sum'
  config: Record<string, unknown>
  description?: string
  priority?: number
}) {
  try {
    const constraint = await prisma.constraint.create({ data })
    revalidatePath('/dashboard')
    return { success: true, constraint }
  } catch (error) {
    return { 
      success: false, 
      error: error instanceof Error ? error.message : 'Unknown error',
      constraint: null
    }
  }
}

export async function deleteConstraintAction(id: string) {
  try {
    await prisma.constraint.update({
      where: { id },
      data: { isActive: false },
    })
    revalidatePath('/dashboard')
    return { success: true }
  } catch (error) {
    return { 
      success: false, 
      error: error instanceof Error ? error.message : 'Unknown error'
    }
  }
}
```

### 7.3 Updated Roster Actions

**File**: `src/app/actions/generate-roster.action.ts`

```typescript
'use server'

import { revalidatePath } from 'next/cache'
import { prisma } from '@/lib/prisma'
import { SolverIntegrationService } from '@/services/solver-integration.service'

export async function generateRosterAction(params: {
  name: string
  startDate: string
  timeSlots: number
  staffIds: string[]
  constraintIds: string[]
}) {
  try {
    const { name, startDate, timeSlots, staffIds, constraintIds } = params
    
    const service = new SolverIntegrationService()
    
    const result = await service.generateRoster(
      name,
      new Date(startDate),
      timeSlots,
      staffIds,
      constraintIds
    )
    
    revalidatePath('/dashboard')
    
    return {
      success: true,
      rosterId: result.rosterId,
      status: result.status,
    }
  } catch (error) {
    console.error('Generate roster error:', error)
    return {
      success: false,
      error: error instanceof Error ? error.message : 'Unknown error',
      rosterId: null,
      status: 'ERROR',
    }
  }
}

export async function getRosterAction(params: { month: string }) {
  try {
    const startDate = new Date(params.month + '-01')
    const endDate = new Date(startDate)
    endDate.setMonth(endDate.getMonth() + 1)
    
    const roster = await prisma.roster.findFirst({
      where: {
        startDate: { gte: startDate },
        endDate: { lt: endDate },
      },
      include: {
        shifts: {
          include: { staff: true },
          orderBy: [{ staffId: 'asc' }, { timeSlot: 'asc' }],
        },
        constraints: true,
      },
      orderBy: { createdAt: 'desc' },
    })
    
    return { success: true, roster }
  } catch (error) {
    return { 
      success: false, 
      error: error instanceof Error ? error.message : 'Unknown error',
      roster: null
    }
  }
}
```

## 8. TypeScript Schemas

**File**: `src/features/dashboard/services/dashboard.service.ts`

```typescript
import { z } from 'zod'

// Staff Schemas
export const createStaffSchema = z.object({
  name: z.string().min(2, 'Name must be at least 2 characters').max(100),
  employeeId: z.string()
    .regex(/^[A-Z0-9]{3,20}$/, 'Employee ID must be 3-20 uppercase alphanumeric characters'),
  email: z.string().email('Invalid email').optional().or(z.literal('')),
})

export type CreateStaffDto = z.infer<typeof createStaffSchema>

// Constraint Schemas
export const pointConstraintFormSchema = z.object({
  type: z.literal('point'),
  name: z.string().min(1).max(100),
  staffId: z.string().min(1, 'Select a staff member'),
  timeSlot: z.number().int().min(0, 'Select a day'),
  state: z.number().int().min(0).max(2),
  description: z.string().optional(),
})

export const verticalSumConstraintFormSchema = z.object({
  type: z.literal('vertical_sum'),
  name: z.string().min(1).max(100),
  timeSlot: z.union([z.number().int().min(0), z.literal('ALL')]),
  targetState: z.number().int().min(0).max(2),
  operator: z.enum(['>=', '<=', '==']),
  value: z.number().int().min(0),
  description: z.string().optional(),
})

// Query Keys
export const staffKeys = {
  all: ['staff'] as const,
}

export const constraintKeys = {
  all: ['constraints'] as const,
}

export const rosterKeys = {
  all: ['roster'] as const,
  month: (month: string) => [...rosterKeys.all, { month }] as const,
}
```

## 9. Shadcn Components to Install

```bash
cd /Users/alan/development/money-maker/orto-ai/web

npx shadcn@latest add table
npx shadcn@latest add dialog
npx shadcn@latest add badge
npx shadcn@latest add skeleton
npx shadcn@latest add select
npx shadcn@latest add dropdown-menu
npx shadcn@latest add sonner
npx shadcn@latest add alert
```

## 10. Testing Approach

### Manual Testing Checklist

```bash
# Prerequisites
cd engine && uv run uvicorn src.main:app --reload  # Terminal 1
cd web && npm run dev                               # Terminal 2
```

**Test Cases**:
1. ✅ Add 3 staff members (Alice, Bob, Carol)
2. ✅ Verify staff appears in sidebar list
3. ✅ Delete one staff member
4. ✅ Add point constraint (Alice off Monday)
5. ✅ Add vertical_sum constraint (Min 2 workers daily)
6. ✅ Click "Generate Roster" → See loading state
7. ✅ Wait for COMPLETED status → Grid appears
8. ✅ Verify Alice has Off on Monday
9. ✅ Verify at least 2 workers each day
10. ✅ Test with no staff → Show error toast

### Unit Test Examples

```typescript
// __tests__/staff-list.test.tsx
describe('StaffList', () => {
  it('renders empty state when no staff', () => {
    // Mock useStaff to return []
    // Render component
    // Assert "No staff added" message
  })
  
  it('displays staff with delete buttons', () => {
    // Mock useStaff to return sample staff
    // Assert staff names render
    // Assert delete buttons present
  })
})
```

## 11. Accessibility Considerations

1. **Keyboard Navigation**:
   - All dialogs can be opened/closed with Enter/Escape
   - Tab order flows logically: Staff → Constraints → Generate → Grid
   
2. **Screen Readers**:
   - Add `aria-label` to icon buttons
   - Use `<th scope="col">` for table headers
   - Announce roster status changes with `aria-live="polite"`

3. **Focus Management**:
   - Return focus to trigger button after dialog closes
   - Focus first input when dialog opens

4. **Color Contrast**:
   - State badge colors meet WCAG AA (4.5:1 ratio)
   - Test with color blindness simulators

## 12. Implementation Status Tracking

- [ ] Install Shadcn components
- [ ] Create feature folder structure
- [ ] Implement Server Actions (staff, constraints, roster)
- [ ] Build React Query hooks
- [ ] Create StaffList component
- [ ] Create ConstraintBuilder component
- [ ] Build RosterGrid component
- [ ] Build RosterControls component
- [ ] Create dashboard page
- [ ] Add toast notifications
- [ ] Manual testing
- [ ] Accessibility audit

---

**Next**: See `task.md` for step-by-step implementation checklist.
````
