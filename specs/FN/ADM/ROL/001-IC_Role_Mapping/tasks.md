# FN/ADM/ROL/001 - Tasks

## Subtasks

### FN/ADM/ROL/001-01: Define rank-to-IC mapping rules
- [ ] Create RankRoleMapping model in schema
- [ ] Add fields: rank, roleId, isAutomatic
- [ ] Seed default mappings (SNO→IC, SRN→IC, RN→Non-IC)
- [ ] Add utility to check IC eligibility by rank
- [ ] Document mapping rules

### FN/ADM/ROL/001-02: Create SRN/RN group balance constraints
- [ ] Define new constraint type: group_balance
- [ ] Add schema fields: groupField, minPerShift, maxPerShift
- [ ] Create constraint builder UI for group balance
- [ ] Add validation for group constraint parameters

### FN/ADM/ROL/001-03: Update solver for group-based distribution
- [ ] Parse group_balance constraint in Python solver
- [ ] Group staff by specified field (rank)
- [ ] Add vertical sum constraint per group
- [ ] Test with SRN min=1, RN min=2 scenarios

### FN/ADM/ROL/001-04: Admin UI for mapping configuration
- [ ] Create /admin/config/role-mappings page
- [ ] Display rank-to-role mapping table
- [ ] Allow toggle IC capability per rank
- [ ] Add manual override for individual staff
- [ ] Show affected staff count per rule
