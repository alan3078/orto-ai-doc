# FN/BE/ENG/003 - Tasks

## Subtasks

### FN/BE/ENG/003-01: Design annual statistics tracking model
- [ ] Create StaffAnnualStats model in Prisma
- [ ] Fields: staffId, year, weekendCount, phCount, nightCount
- [ ] Add unique constraint on (staffId, year)
- [ ] Create migration
- [ ] Add utility functions for querying stats

### FN/BE/ENG/003-02: Implement weekend shift counter
- [ ] Create isWeekend(date) utility
- [ ] Update roster finalization to count weekend shifts
- [ ] Aggregate by staff and update annual stats
- [ ] Handle roster updates (recalculate)
- [ ] Test with sample rosters

### FN/BE/ENG/003-03: Implement PH shift counter
- [ ] Integrate with PublicHoliday table
- [ ] Create isPublicHoliday(date) utility
- [ ] Count PH working days per staff
- [ ] Update annual stats on roster finalization
- [ ] Handle overlapping weekend+PH (count both)

### FN/BE/ENG/003-04: Cross-month fairness optimization
- [ ] Query YTD stats before roster generation
- [ ] Pass staff annual counts to solver
- [ ] Add soft constraint: minimize weekend variance
- [ ] Add soft constraint: minimize night variance
- [ ] Add soft constraint: minimize PH variance
- [ ] Weight constraints appropriately

### FN/BE/ENG/003-05: Annual balance reporting UI
- [ ] Create /admin/reports/annual-balance page
- [ ] Display stats table: staff × metric matrix
- [ ] Show variance and standard deviation
- [ ] Add year selector
- [ ] Highlight outliers (high/low)
- [ ] Add export to CSV option
