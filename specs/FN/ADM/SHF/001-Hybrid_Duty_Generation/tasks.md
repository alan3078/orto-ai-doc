# FN/ADM/SHF/001 - Tasks

## Subtasks

### FN/ADM/SHF/001-01: Add HYBRID shift type to ShiftType enum
- [ ] Update schema.prisma ShiftType enum
- [ ] Create migration
- [ ] Update TypeScript types
- [ ] Update shift type display names

### FN/ADM/SHF/001-02: Dynamic state expansion based on active ShiftDefinitions
- [ ] Create getActiveStates(shiftType) function
- [ ] For HYBRID: query all active ShiftDefinitions
- [ ] Build states array: ["OFF", ...shift codes]
- [ ] Sort states consistently
- [ ] Cache result for roster session

### FN/ADM/SHF/001-03: Update solver to handle variable state count
- [ ] Modify solver payload to pass states array
- [ ] Update Python solver to use dynamic states
- [ ] Ensure constraint validation against available states
- [ ] Test with 3, 5, 7+ states

### FN/ADM/SHF/001-04: UI for hybrid mode selection
- [ ] Add HYBRID option to shift type selector
- [ ] Show preview of included shifts when selected
- [ ] Update constraint builder for hybrid shifts
- [ ] Update roster grid to handle all shift types
- [ ] Update shift color mapping for extended shifts
