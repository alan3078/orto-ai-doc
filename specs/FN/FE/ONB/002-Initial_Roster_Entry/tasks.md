# FN/FE/ONB/002 - Tasks

## Subtasks

### FN/FE/ONB/002-01: Design manual roster entry form
- [ ] Create entry grid component
- [ ] Display staff names in rows
- [ ] Display last 3 days as columns
- [ ] Add shift selector per cell (dropdown)
- [ ] Show available shifts from ShiftDefinitions
- [ ] Allow empty (skip) for unknown

### FN/FE/ONB/002-02: Implement roster data import API
- [ ] Create InitialRoster model or use existing
- [ ] Define shape: { staffId, date, shiftCode }[]
- [ ] Create saveInitialRoster server action
- [ ] Validate staff IDs exist
- [ ] Validate shift codes exist
- [ ] Store in database

### FN/FE/ONB/002-03: Build onboarding wizard step
- [ ] Create /onboarding/initial-roster page
- [ ] Show instructions and context
- [ ] Embed entry form component
- [ ] Add Save and Skip buttons
- [ ] Track completion status
- [ ] Redirect to dashboard on complete

### FN/FE/ONB/002-04: Validate and persist initial data
- [ ] Validate all entries before save
- [ ] Check for duplicate entries
- [ ] Handle partial entries gracefully
- [ ] Show validation errors inline
- [ ] Confirm save with summary
- [ ] Mark onboarding step complete
