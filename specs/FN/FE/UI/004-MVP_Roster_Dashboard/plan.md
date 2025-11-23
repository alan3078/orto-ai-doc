# Plan - FN/FE/UI/004 - MVP Roster Dashboard

## 1. Overview

Build a unified "Mission Control" dashboard at `/dashboard` that consolidates the complete roster workflow into a single page. Users can manage staff, configure constraints, generate rosters, and visualize schedules in a reactive grid interface. This eliminates navigation between multiple pages and provides real-time feedback during roster generation.

This is the **primary user interface** that ties together all backend services created in previous phases.

## 2. Goals

- **Single-Page Workflow**: Manage staff, rules, and roster generation without page navigation
- **Real-Time Updates**: Use React Query for optimistic updates and automatic cache invalidation
- **Visual Grid Display**: Clear matrix view of staff × time slots with color-coded state indicators
- **Progressive Enhancement**: Start with basic constraint forms, extensible to NLP interface later
- **Mobile Responsive**: Functional on tablet and desktop devices

## 3. Scope (MVP)

### Dashboard Layout
- `/dashboard` route with three-column grid layout
- Left sidebar (3 cols): Staff list + Constraint builder
- Main content (9 cols): Roster controls + Grid visualization
- Header with title, date range, and export button

### Staff Management
- List all active staff members
- Add new staff via dialog form
- Delete staff with confirmation
- Real-time updates using React Query

### Constraint Configuration
- Visual forms for two constraint types:
  - **Point Constraint**: Assign specific state to staff on specific day
  - **Vertical Sum**: Min/max workers per day
- Add/delete constraints
- Display active constraints with type badges

### Roster Generation
- "Generate Roster" button with loading state
- Month selector to view past rosters
- Poll roster status during SOLVING phase
- Display success/error/infeasible states

### Schedule Visualization
- Staff (rows) × Days (columns) grid
- Color-coded state badges (Off=grey, Work=blue, OnCall=purple)
- Skeleton loaders during generation
- Empty state when no roster exists
- Responsive horizontal scrolling

## 4. Out of Scope

- **NLP Constraint Input**: Manual forms only (text-to-constraint comes later)
- **Drag-and-Drop Editing**: Read-only grid (manual overrides post-MVP)
- **Multi-Month View**: Single month at a time
- **Advanced Filtering**: No filtering by staff/constraint
- **Schedule Templates**: No save/load constraint sets
- **Export Formats**: CSV only (no PDF/iCal)
- **Authentication**: Single-user mode (multi-tenancy later)

## 5. Success Criteria

✅ **Staff Management**: Add, view, delete staff without page refresh

✅ **Constraint Builder**: Create point and sum constraints using form inputs

✅ **Roster Generation**: Click "Generate" → Status updates to SOLVING → Grid appears with results

✅ **Visual Grid**: Clear display of 5 staff × 7 days with color-coded states

✅ **Error Handling**: INFEASIBLE and FAILED states display helpful messages

✅ **Responsive Design**: Works on 1024px+ screens without horizontal scroll (except grid)

## 6. Dependencies

### Required (Blocking)
- **FN/BE/DATA/003**: Database schema and integration service must be complete
- **FN/BE/ENG/001**: Python solver engine must be running
- **Shadcn UI Components**: table, dialog, badge, skeleton, select, dropdown-menu, sonner

### Future Dependents (Blocked By)
- **NLP Constraint Parser**: Will replace manual forms
- **Schedule Editor**: Requires this dashboard as foundation
- **Staff Preferences**: Needs staff management UI

## 7. Estimated Effort

- **Component Installation**: 1 hour (Shadcn components)
- **Feature Structure**: 1 hour (folders, types, schemas)
- **Server Actions**: 3 hours (Staff + Constraint + Roster CRUD)
- **React Query Hooks**: 3 hours (Queries + mutations + invalidation)
- **StaffList Component**: 2 hours (List + Add dialog + Delete)
- **ConstraintBuilder**: 4 hours (Form switching + Validation)
- **RosterGrid**: 4 hours (Table layout + State badges + Loading)
- **Dashboard Page**: 3 hours (Layout + Controls + Integration)
- **Testing & Polish**: 3 hours (Manual testing + Accessibility)
- **Total**: ~3 days (24 hours)

## 8. Priority

**P0** - Critical Path

This is the main user interface for the MVP. Without this dashboard, users cannot interact with the scheduling system. All previous backend work culminates in this UI.

## 9. Risks & Mitigation

| Risk | Impact | Mitigation |
|------|--------|------------|
| React Query learning curve | Medium | Follow existing `features/users` pattern closely |
| Complex form state management | Medium | Use react-hook-form + Zod like existing forms |
| Grid performance with many staff | Medium | Start with small datasets, add virtualization if needed |
| Polling during SOLVING degrades UX | Low | Use 2s intervals with exponential backoff |
| Mobile responsiveness | Medium | Focus on tablet+ (768px), mobile is stretch goal |

## 10. Testing Strategy

### Unit Tests
- Staff list rendering and interactions
- Constraint form validation
- Mutation hooks invalidation logic

### Integration Tests
- Full workflow: Add staff → Add constraint → Generate roster → View grid
- Error scenarios (no staff, API down, infeasible)

### Manual Testing
- Generate roster with various constraint combinations
- Verify state colors in grid
- Test responsive breakpoints
- Keyboard navigation and accessibility

### Acceptance Testing
- Product owner can generate a 7-day roster for 5 nurses
- Grid clearly shows who works which days
- Error messages are understandable

---

**Next Steps**: See `implementation.md` for technical architecture and `task.md` for actionable checklist.
