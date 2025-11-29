# FN/ADM/RST/002 - Previous Month Integration

> **V1.0 Requirement #1**: Able to interact with last month duty to avoid mistakes

## Overview

Enable roster generation to consider the last days of the previous month to ensure continuity constraints (like post-night rest, consecutive shift limits) are respected across month boundaries.

## Goals

1. Schema extension to link rosters across months
2. API to fetch previous month's final days
3. Solver integration for prior month constraints
4. UI to display previous month context

## Scope

### In Scope
- Load last 3-7 days of previous month roster
- Pass previous assignments to solver as fixed constraints
- Display previous month context in roster grid (greyed out)
- Validate continuity constraints across boundary
- Handle case when no previous roster exists

### Out of Scope
- Full roster history view
- Edit previous month from current view
- Automatic roster linking

## Success Criteria

- [ ] Roster generation considers previous month's final days
- [ ] No post-night-day violations at month boundary
- [ ] No excessive consecutive shifts at month boundary
- [ ] UI shows previous month context (read-only)
- [ ] Works correctly when no previous roster exists

## Dependencies

- Existing Roster model and generation flow

## Technical Approach

1. **Pre-generation query**: Fetch published roster for previous month
2. **Extract final days**: Get last 3-7 days of assignments
3. **Generate point constraints**: Convert to fixed assignments
4. **Extend date range**: Solver runs from (month_start - overlap_days) to month_end
5. **Trim results**: Only save/display current month assignments
