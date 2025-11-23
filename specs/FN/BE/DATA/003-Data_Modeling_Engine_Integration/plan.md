# Plan - FN/BE/DATA/003 - Data Modeling & Engine Integration

## 1. Overview

This feature bridges the Next.js Web Application and the Python Solver Engine by designing a persistent PostgreSQL schema and implementing server-side orchestration logic. It transforms database records into the JSON format required by the Python API and stores the resulting schedules back into the database.

This is the **critical integration layer** that connects all pieces of the system together.

## 2. Goals

- **Persistence**: Design a normalized relational schema to store Staff, Constraints, Rosters, and Shifts
- **Translation**: Build a "Mapper Service" that converts Prisma models to the Python Engine's JSON contract
- **Orchestration**: Create a Server Action that triggers the solver and handles the complete workflow transactionally
- **Testability**: Ensure the integration can be tested independently of the UI

## 3. Scope (MVP)

### Database Schema
- Define Prisma models for:
  - `Staff` (resource entities)
  - `Constraint` (scheduling rules)
  - `Roster` (schedule metadata)
  - `Shift` (individual schedule assignments)

### Seed Data
- Create a database seed script with sample data:
  - 5 sample nurses/staff members
  - 2-3 basic constraints (point and vertical_sum)
  - Configurable time slots (7 days default)

### Integration Service
- TypeScript service: `solveRoster(config, constraints)`
- HTTP client to communicate with Python API at `http://localhost:8000`
- Request/response transformation logic

### Result Handling
- Parse solver response
- Bulk insert shifts into database
- Handle infeasible scenarios gracefully
- Transaction management for data integrity

## 4. Out of Scope

- **Frontend UI**: No React components or pages (comes in Phase 3)
- **Authentication**: No user/org separation yet
- **Advanced Constraints**: Only supporting point and vertical_sum from FN/BE/ENG/001
- **Real-time Updates**: No WebSocket or polling
- **Schedule Editing**: Read-only schedule results

## 5. Success Criteria

✅ **Schema Validation**: `npx prisma db push` creates all tables without errors

✅ **Seed Execution**: `npx prisma db seed` populates database with test data

✅ **API Communication**: Server Action successfully calls Python engine and receives response

✅ **Data Persistence**: Generated schedule appears in `Shift` table (viewable in Prisma Studio)

✅ **Error Handling**: Infeasible scenarios are logged properly without crashing

✅ **Type Safety**: All TypeScript types are properly inferred from Prisma schema

## 6. Dependencies

### Required (Blocking)
- **FN/BE/ENG/001**: Python Solver Engine must be running and accessible
- **FN/FE/WEB/001**: Prisma setup must be complete with working database connection

### Future Dependents (Blocked By)
- **FN/FE/SCD/020**: Frontend schedule view needs this data layer
- **FN/ADM/RUL/001**: Constraint management UI requires this schema

## 7. Estimated Effort

- **Schema Design**: 4 hours
- **Seed Script**: 2 hours
- **Mapper Service**: 4 hours
- **Server Action**: 4 hours
- **Testing & Debugging**: 4 hours
- **Total**: ~2 days (18 hours)

## 8. Priority

**P0** - Critical Path

This is a foundational feature that unblocks both frontend and admin features. Cannot proceed with UI development without this data layer.

## 9. Risks & Mitigation

| Risk | Impact | Mitigation |
|------|--------|------------|
| Python API not available | High | Add retry logic and clear error messages |
| Type mismatch between schemas | Medium | Use Zod validation at integration boundary |
| Large roster performance | Medium | Start with small test data, optimize later |
| Transaction rollback complexity | Medium | Use Prisma transactions with proper error handling |

## 10. Testing Strategy

### Unit Tests
- Mapper service transformation logic
- Constraint serialization

### Integration Tests
- Full workflow: DB → Python API → DB
- Error scenarios (infeasible, API down, invalid data)

### Manual Testing
- Prisma Studio inspection after generation
- Multiple roster scenarios

---

**Next Steps**: See `implementation.md` for technical details and `task.md` for actionable subtasks.
