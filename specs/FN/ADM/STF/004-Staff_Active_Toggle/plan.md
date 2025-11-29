# FN/ADM/STF/004 - Staff Active/Inactive Toggle

> **V1.0 Requirement #10**: Add active/inactive function on staff card

## Overview

Add UI functionality to toggle staff active status. The `isActive` field already exists in the Prisma schema - this feature adds the UI controls.

## Goals

1. Add toggle button to staff card
2. Create server action to update staff status
3. Add visual indicator for inactive staff
4. Add filter to show/hide inactive staff

## Scope

### In Scope
- Toggle button on staff card
- Confirmation dialog before deactivation
- Visual indicator (badge, opacity, strikethrough)
- Filter dropdown in staff list
- Prevent inactive staff from roster generation

### Out of Scope
- Bulk activate/deactivate
- Scheduled activation/deactivation
- Reason tracking for status changes

## Success Criteria

- [ ] Staff card shows toggle button
- [ ] Clicking toggle prompts confirmation
- [ ] Inactive staff visually distinguished
- [ ] Filter allows show all/active only/inactive only
- [ ] Inactive staff excluded from roster generation

## Existing Implementation

- `isActive` field exists in Staff model (schema.prisma line 26)
- `fetchStaff` already filters by isActive by default
- Engine mapper filters active staff

## Dependencies

- None (quick win feature)
