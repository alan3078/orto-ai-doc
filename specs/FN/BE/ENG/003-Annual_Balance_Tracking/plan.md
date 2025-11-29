# FN/BE/ENG/003 - Annual Balance Tracking

> **V1.0 Requirement #14**: Number of Weekend/PH and Night balance annually

## Overview

Track and balance weekend shifts, public holiday shifts, and night shifts across the entire year to ensure fair distribution among staff.

## Goals

1. Design annual statistics tracking model
2. Implement weekend shift counter
3. Implement PH shift counter
4. Cross-month fairness optimization
5. Annual balance reporting UI

## Scope

### In Scope
- Track per-staff annual counts: weekends, PH days, night shifts
- Consider historical data when generating new rosters
- Soft constraint for annual balance optimization
- Dashboard showing annual statistics
- Year-to-date fairness report

### Out of Scope
- Multi-year tracking
- Predictive balancing
- Compensation for imbalance

## Success Criteria

- [ ] Annual counts tracked per staff
- [ ] Solver considers YTD balance when generating
- [ ] Weekend/PH variance minimized across staff
- [ ] Night shift variance minimized across staff
- [ ] Manager can view annual balance report

## Technical Approach

1. **StaffAnnualStats model**: Track counts by staff, year
2. **Post-roster aggregation**: Update stats after roster finalized
3. **Pre-generation query**: Load YTD stats as solver input
4. **Fairness objective**: Minimize variance in annual counts
5. **Reporting**: Aggregate and display stats
