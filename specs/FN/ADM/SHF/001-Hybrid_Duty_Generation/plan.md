# FN/ADM/SHF/001 - Hybrid Duty Generation

> **V1.0 Requirement #6**: Able to generate hybrid duty, i.e. A/7/P/E/N/customized shifts together

## Overview

Enable roster generation with mixed shift types from both 7E (7, E) and APN (A, P, N) patterns, plus custom shifts, in a single roster.

## Goals

1. Add HYBRID shift type to enum
2. Dynamic state expansion based on active shifts
3. Update solver for variable state count
4. UI for hybrid mode selection

## Scope

### In Scope
- HYBRID shift type that includes all active ShiftDefinitions
- Dynamic states array based on enabled shifts
- Updated constraints UI for hybrid mode
- Validation of hybrid constraints

### Out of Scope
- Per-staff shift type restrictions
- Automatic shift type suggestions
- Shift type rotation rules

## Success Criteria

- [ ] Can select HYBRID when creating roster
- [ ] Solver includes all active shifts as valid states
- [ ] Constraints work with any active shift
- [ ] Roster grid shows all shift types correctly
- [ ] Validation handles hybrid rosters

## Current Implementation

- ShiftType enum: SEVEN_E, APN
- Each roster tied to one ShiftType
- ShiftDefinitions have shiftType field
- States hardcoded per shift type

## Technical Approach

1. **Add HYBRID enum value**: New ShiftType.HYBRID
2. **Query active shifts**: For HYBRID, query all active ShiftDefinitions
3. **Build dynamic states**: ["OFF", "7", "E", "A", "P", "N", ...custom]
4. **Update solver payload**: Include extended states array
5. **Update constraints**: Support any shift code in constraints
