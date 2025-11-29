# FN/FE/ONB/002 - Initial Roster Entry

> **V1.0 Requirement #13**: 1st time user experience - Manager: enter last 3 days of last month roster

## Overview

Enable managers to enter the final days of the previous month's roster during initial setup, ensuring continuity constraints are respected from the first generated roster.

## Goals

1. Design manual roster entry form
2. Implement roster data import API
3. Build onboarding wizard step
4. Validate and persist initial data

## Scope

### In Scope
- Manual entry grid for last 3 days
- Staff list with shift selection
- Validation against shift definitions
- Save as "previous month context"
- One-time setup per organization

### Out of Scope
- Full roster import from Excel
- Bulk paste from spreadsheet
- Image/PDF roster parsing

## Success Criteria

- [ ] Manager can enter shifts for last 3 days
- [ ] All staff shown in entry grid
- [ ] Shifts validated against active definitions
- [ ] Data persisted for first roster generation
- [ ] Step skippable if no previous roster

## Dependencies

- FN/ADM/RST/002 (Previous Month Integration) - for data consumption

## User Flow

```
Onboarding (Manager) → Initial Roster Entry → Enter shifts → Save → Complete
                                ↓
                        Skip (first time setup)
```
