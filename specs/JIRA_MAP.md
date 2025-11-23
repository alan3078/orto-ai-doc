# JIRA Ticket Map - Universal Scheduler (Nurse Edition)

> **Last Updated**: 23 November 2025

This document provides a hierarchical index of all Jira tickets mapped to their specification folders.

---

## Ticket ID Convention

```
{Type}/{Context}/{Module}/{ID}[-{Task}]
```

- **Type**: `FN` (Feature) | `BG` (Bug) | `CH` (Chore)
- **Context**: `ADM` (Admin CMS) | `FE` (Frontend) | `BE` (Backend/Engine)
- **Module**: `RUL` (Rules) | `STF` (Staff) | `ENG` (Engine) | `SCD` (Schedule) | `RPT` (Report)
- **ID**: 3-digit feature number (e.g., `001`, `002`)
- **Task**: 2-digit subtask number (e.g., `-01`, `-02`)

---

## 📋 Feature Index

### FN - Features

#### BE - Backend/Engine

##### ENG - Engine Module

| Jira ID | Status | Path | Description |
|---------|--------|------|-------------|
| **FN/BE/ENG/001** | ✅ Completed | `/specs/FN/BE/ENG/001-Core_Universal_Solver/` | Core Universal Solver (MVP) - Generic OR-Tools engine with JSON API |
| └─ FN/BE/ENG/001-01 | ✅ Completed | | Project Init & Scaffold |
| └─ FN/BE/ENG/001-02 | ✅ Completed | | Define Pydantic Models (7 constraint types) |
| └─ FN/BE/ENG/001-03 | ✅ Completed | | Implement Variable Initialization Logic |
| └─ FN/BE/ENG/001-04 | ✅ Completed | | Implement Constraint Parsers (7 types) |
| └─ FN/BE/ENG/001-05 | ✅ Completed | | API Endpoint & Integration (/api/v1/solve, /health) |
| └─ FN/BE/ENG/001-06 | ✅ Completed | | Unit Tests (15 comprehensive tests) |
| **FN/BE/ENG/002** | ✅ Completed | `/specs/FN/BE/ENG/002-Roster_Validator/` | Roster Validator (Audit Mode) - Validate existing rosters against constraints with detailed reporting |
| └─ FN/BE/ENG/002-01 | ✅ Completed | | Define Validation Schemas |
| └─ FN/BE/ENG/002-02 | ✅ Completed | | Implement Validator Service - Core Logic |
| └─ FN/BE/ENG/002-03 | ✅ Completed | | Implement Validator - Point & Sum Constraints |
| └─ FN/BE/ENG/002-04 | ✅ Completed | | Implement Validator - Advanced Constraints |
| └─ FN/BE/ENG/002-05 | ✅ Completed | | Create Validation API Endpoint |
| └─ FN/BE/ENG/002-06 | ✅ Completed | | Write Backend Unit Tests |
| └─ FN/BE/ENG/002-07 | ✅ Completed | | Create TypeScript Validation Schemas |
| └─ FN/BE/ENG/002-08 | ✅ Completed | | Extend Solver Integration Service |
| └─ FN/BE/ENG/002-09 | ✅ Completed | | Create Validation Server Action |
| └─ FN/BE/ENG/002-10 | ✅ Completed | | Build ValidationDrawer Component |
| └─ FN/BE/ENG/002-11 | ✅ Completed | | Build ConstraintReportCard Component |
| └─ FN/BE/ENG/002-12 | ✅ Completed | | Integrate Validation Button - Roster Management |
| └─ FN/BE/ENG/002-13 | ✅ Completed | | Integrate Validation Button - Test Roster Detail |
| └─ FN/BE/ENG/002-14 | ✅ Completed | | End-to-End Testing |
| └─ FN/BE/ENG/002-15 | ✅ Completed | | Documentation and Cleanup |

##### DATA - Data Integration Module

| Jira ID | Status | Path | Description |
|---------|--------|------|-------------|
| **FN/BE/DATA/003** | 🚧 In Progress | `/specs/FN/BE/DATA/003-Data_Modeling_Engine_Integration/` | Data Modeling & Engine Integration - Prisma schema, solver service, and orchestration |
| └─ FN/BE/DATA/003-01 | ✅ Completed | | Define Prisma Schema Models |
| └─ FN/BE/DATA/003-02 | ⬜ Not Started | | Create Migration and Apply Schema |
| └─ FN/BE/DATA/003-03 | ✅ Completed | | Create Database Seed Script |
| └─ FN/BE/DATA/003-04 | ⬜ Not Started | | Build Solver Integration Service |
| └─ FN/BE/DATA/003-05 | ⬜ Not Started | | Implement generateRoster() Orchestration |
| └─ FN/BE/DATA/003-06 | ⬜ Not Started | | Create Server Action |
| └─ FN/BE/DATA/003-07 | ⬜ Not Started | | Create Test Script |
| └─ FN/BE/DATA/003-08 | ⬜ Not Started | | End-to-End Testing |
| └─ FN/BE/DATA/003-09 | ⬜ Not Started | | Documentation and Cleanup |

#### ADM - Admin CMS

##### RUL - Rules Module

| Jira ID | Status | Path | Description |
|---------|--------|------|-------------|
| **FN/ADM/RUL/001** | 📝 Planned | `/specs/FN/ADM/RUL/001-Basic_Constraints/` | Basic Constraints - Define and manage foundational scheduling rules |
| **FN/ADM/RUL/002** | 📝 Planned | `/specs/FN/ADM/RUL/002-Nurse_Patterns/` | Nurse Patterns - Sliding window and sequence pattern constraints |

##### STF - Staff Module

| Jira ID | Status | Path | Description |
|---------|--------|------|-------------|
| **FN/ADM/STF/003** | 📝 Planned | `/specs/FN/ADM/STF/003-Staff_Import/` | Staff Import - Bulk upload and management of staff data |

#### FE - Frontend

##### WEB - Web Foundation Module

| Jira ID | Status | Path | Description |
|---------|--------|------|-------------|
| **FN/FE/WEB/001** | 🚧 In Progress | `/specs/FN/FE/WEB/001-Web_Foundation_Setup/` | Web Foundation Setup - Next.js 15 + TypeScript + PostgreSQL + Prisma + shadcn/ui (Node 22, no auth) |
| └─ FN/FE/WEB/001-01 | ⬜ Not Started | | Initialize Next.js Project |
| └─ FN/FE/WEB/001-02 | ⬜ Not Started | | Setup PostgreSQL Database |
| └─ FN/FE/WEB/001-03 | ⬜ Not Started | | Configure Prisma ORM |
| └─ FN/FE/WEB/001-04 | ⬜ Not Started | | Initialize shadcn/ui |
| └─ FN/FE/WEB/001-05 | ⬜ Not Started | | Create Database Test Pages & API |
| └─ FN/FE/WEB/001-06 | ⬜ Not Started | | Create Homepage |
| └─ FN/FE/WEB/001-07 | ⬜ Not Started | | Final Integration Test |

##### SCD - Schedule Module

| Jira ID | Status | Path | Description |
|---------|--------|------|-------------|
| **FN/FE/SCD/020** | 📝 Planned | `/specs/FN/FE/SCD/020-My_Roster_View/` | My Roster View - Individual nurse schedule viewing interface |

---

## 🐛 Bug Index

### BG - Bugs

*No bugs documented yet.*

---

## 🔧 Chore Index

### CH - Chores

*No chores documented yet.*

---

## Status Legend

- 🚧 **In Progress** - Actively being worked on
- ✅ **Completed** - Feature fully implemented and tested
- 📝 **Planned** - Spec folder created, awaiting implementation
- ⬜ **Not Started** - Subtask not yet begun
- ⏸️ **Blocked** - Waiting on dependencies

---

## Navigation

To view full specifications for any feature:

```bash
cd specs/{Type}/{Context}/{Module}/{ID}-{Feature_Name}/
```

Each feature folder contains:
- `plan.md` - Overview, goals, scope, success criteria
- `implementation.md` - Technical details, schemas, logic
- `task.md` - Jira subtask breakdown with checkboxes
