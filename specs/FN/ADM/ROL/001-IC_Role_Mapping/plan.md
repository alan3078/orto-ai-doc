# FN/ADM/ROL/001 - IC Role Mapping

> **V1.0 Requirement #4**: Able to assign IC role according to staff title (IC role mapping), plus SRN and RN group balance

## Overview

Automatically determine IC capability based on staff rank/title, and ensure balanced distribution of shifts between Senior Registered Nurses (SRN) and Registered Nurses (RN).

## Goals

1. Define rank-to-IC mapping rules
2. Create SRN/RN group balance constraints
3. Update solver for group-based distribution
4. Admin UI for mapping configuration

## Scope

### In Scope
- Rank-based IC eligibility (SNO, SRN auto-IC, RN configurable)
- Automatic IC role assignment based on rank
- Group balance constraints (SRN:RN ratio per shift)
- Admin configuration for mapping rules
- Override capability for individual staff

### Out of Scope
- Complex qualification tracking
- Training/certification requirements
- Seniority-based shift preferences

## Success Criteria

- [ ] SNO and SRN staff automatically marked as IC-capable
- [ ] IC role derived from rank (not manually assigned)
- [ ] Each shift has target SRN:RN ratio
- [ ] Solver balances groups across shifts
- [ ] Admin can configure rank-to-IC mappings

## Current Implementation

- Staff model has `rank` field (SNO, SRN, RN)
- Role model exists for IC/Non-IC
- StaffRole junction links staff to roles
- IC assigned manually via seed data

## Technical Approach

1. **Mapping rules**: Create RankRoleMapping table
2. **Auto-assign**: On staff create/update, derive roles from rank
3. **Group constraint**: New constraint type `group_balance`
4. **Solver update**: Handle group-based vertical distribution
