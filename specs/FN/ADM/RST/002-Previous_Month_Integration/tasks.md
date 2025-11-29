# FN/ADM/RST/002 - Tasks

## Subtasks

### FN/ADM/RST/002-01: Schema extension to link rosters across months
- [ ] Add optional previousRosterId to Roster model
- [ ] Create query to find published roster for previous month
- [ ] Add utility function getPreviousMonthRoster(year, month)
- [ ] Handle organization scoping

### FN/ADM/RST/002-02: API to fetch previous month's final days
- [ ] Create fetchPreviousMonthAssignments function
- [ ] Accept parameters: year, month, daysToFetch (default 3)
- [ ] Return array of { staffId, date, shiftCode } assignments
- [ ] Handle missing roster gracefully (return empty)

### FN/ADM/RST/002-03: Solver integration to consider prior month constraints
- [ ] Update buildSolverPayload to include previous days
- [ ] Generate point constraints for previous assignments
- [ ] Adjust solver date range to include overlap
- [ ] Mark previous day constraints as required (hard)
- [ ] Update result parser to trim previous days

### FN/ADM/RST/002-04: UI to display previous month context
- [ ] Add columns for previous month days in roster grid
- [ ] Style previous days differently (greyed, non-editable)
- [ ] Show shift assignments from previous roster
- [ ] Add visual separator between months
- [ ] Update export to exclude previous month columns
