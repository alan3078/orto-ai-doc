# FN/ADM/STF/004 - Tasks

## Subtasks

### FN/ADM/STF/004-01: Add toggle button to staff card
- [x] Add Switch or Toggle component to StaffCard (staff-list.tsx)
- [x] Position in new "Active" column
- [x] Wire up onClick handler
- [x] Add confirmation dialog (window.confirm) before toggle

### FN/ADM/STF/004-02: Create toggleStaffActive server action
- [x] Create toggleStaffActiveAction in staff.actions.ts
- [x] Accept staffId parameter
- [x] Toggle isActive field
- [x] Revalidate staff list path
- [x] Return success/error response with message

### FN/ADM/STF/004-03: Add visual indicator for inactive staff
- [x] Add Badge "Inactive" to name cell
- [x] Reduce opacity (opacity-50) for inactive rows
- [x] Add strikethrough to name
- [x] Different background color (bg-muted/30)

### FN/ADM/STF/004-04: Add filter to show/hide inactive staff
- [x] Add Select dropdown above staff grid
- [x] Options: "Active Only", "Inactive Only", "All Staff"
- [x] Create getStaffWithFilterAction to accept filter parameter
- [x] Create useStaffWithFilter hook
- [x] Default to "Active Only"

## Implementation Summary

**Files Modified:**
- `src/app/actions/staff.actions.ts` - Added `toggleStaffActiveAction` and `getStaffWithFilterAction`
- `src/features/dashboard/hooks/use-staff.ts` - Added `useToggleStaffActive` and `useStaffWithFilter` hooks
- `src/features/dashboard/components/staff-list.tsx` - Added toggle switch, visual indicators, and filter dropdown

**Status:** ✅ Complete (29 Nov 2025)
