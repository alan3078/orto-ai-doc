# JIRA Ticket Map - Universal Scheduler (Nurse Edition)

> **Last Updated**: 29 November 2025

This document provides a hierarchical index of all Jira tickets mapped to their specification folders.

---

## Ticket ID Convention

```
{Type}/{Context}/{Module}/{ID}[-{Task}]
```

- **Type**: `FN` (Feature) | `BG` (Bug) | `CH` (Chore)
- **Context**: `ADM` (Admin CMS) | `FE` (Frontend) | `BE` (Backend/Engine)
- **Module**: `RUL` (Rules) | `STF` (Staff) | `ENG` (Engine) | `SCD` (Schedule) | `RPT` (Report) | `AUTH` (Auth) | `LVE` (Leave) | `REQ` (Request) | `RST` (Roster) | `SHF` (Shift) | `ROL` (Role) | `I18N` (i18n) | `ONB` (Onboarding)
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
| **FN/BE/ENG/003** | 📝 Planned | `/specs/FN/BE/ENG/003-Annual_Balance_Tracking/` | Annual Balance Tracking - Weekend/PH/Night balance across year (V1.0 #14) |
| └─ FN/BE/ENG/003-01 | ⬜ Not Started | | Design annual statistics tracking model |
| └─ FN/BE/ENG/003-02 | ⬜ Not Started | | Implement weekend shift counter |
| └─ FN/BE/ENG/003-03 | ⬜ Not Started | | Implement PH shift counter |
| └─ FN/BE/ENG/003-04 | ⬜ Not Started | | Cross-month fairness optimization |
| └─ FN/BE/ENG/003-05 | ⬜ Not Started | | Annual balance reporting UI |

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

---

#### ADM - Admin CMS

##### AUTH - Authentication Module

| Jira ID | Status | Path | Description |
|---------|--------|------|-------------|
| **FN/ADM/AUTH/001** | 📝 Planned | `/specs/FN/ADM/AUTH/001-User_Account_Types/` | User Account Types - Manager/Member accounts with NextAuth.js (V1.0 #8) |
| └─ FN/ADM/AUTH/001-01 | ⬜ Not Started | | Define User and UserRole Prisma models |
| └─ FN/ADM/AUTH/001-02 | ⬜ Not Started | | Setup NextAuth.js with credentials provider |
| └─ FN/ADM/AUTH/001-03 | ⬜ Not Started | | Create login/logout UI |
| └─ FN/ADM/AUTH/001-04 | ⬜ Not Started | | Add session management |
| └─ FN/ADM/AUTH/001-05 | ⬜ Not Started | | Create user management admin UI |
| **FN/ADM/AUTH/002** | 📝 Planned | `/specs/FN/ADM/AUTH/002-Role_Based_Permissions/` | Role-Based Permissions - Manager/Member access control (V1.0 #9, depends on AUTH/001) |
| └─ FN/ADM/AUTH/002-01 | ⬜ Not Started | | Define permission matrix |
| └─ FN/ADM/AUTH/002-02 | ⬜ Not Started | | Create auth middleware |
| └─ FN/ADM/AUTH/002-03 | ⬜ Not Started | | Protect admin routes (Manager only) |
| └─ FN/ADM/AUTH/002-04 | ⬜ Not Started | | Implement Member-only views |
| └─ FN/ADM/AUTH/002-05 | ⬜ Not Started | | Add duty request submission for Members |

##### RUL - Rules Module

| Jira ID | Status | Path | Description |
|---------|--------|------|-------------|
| **FN/ADM/RUL/001** | 📝 Planned | `/specs/FN/ADM/RUL/001-Basic_Constraints/` | Basic Constraints - Define and manage foundational scheduling rules |
| **FN/ADM/RUL/002** | 📝 Planned | `/specs/FN/ADM/RUL/002-Nurse_Patterns/` | Nurse Patterns - Sliding window and sequence pattern constraints |
| **FN/ADM/RUL/004** | ✅ Completed | `/specs/FN/ADM/RUL/004-Fairness_Soft_Constraints/` | Fairness & Soft Constraints - Hard/Soft constraint distinction with penalty-based optimization |
| └─ FN/ADM/RUL/004-01 | ✅ Completed | | Schema & Migration - Add isRequired field |
| └─ FN/ADM/RUL/004-02 | ✅ Completed | | Seed Data Update - Add isRequired to constraints |
| └─ FN/ADM/RUL/004-03 | ✅ Completed | | Python Schema Update - Add is_required to Pydantic models |
| └─ FN/ADM/RUL/004-04 | ✅ Completed | | Solver Soft Constraint Implementation |
| └─ FN/ADM/RUL/004-05 | ✅ Completed | | TypeScript Integration - Pass is_required to solver |
| └─ FN/ADM/RUL/004-06 | ✅ Completed | | UI - Required Column & Badge |
| └─ FN/ADM/RUL/004-07 | ✅ Completed | | UI - Edit Dialog Toggle |
| └─ FN/ADM/RUL/004-08 | 🔄 Ready | | Integration Testing |

##### STF - Staff Module

| Jira ID | Status | Path | Description |
|---------|--------|------|-------------|
| **FN/ADM/STF/003** | 📝 Planned | `/specs/FN/ADM/STF/003-Staff_Import/` | Staff Import - Bulk upload and management of staff data |
| **FN/ADM/STF/004** | ✅ Completed | `/specs/FN/ADM/STF/004-Staff_Active_Toggle/` | Staff Active/Inactive Toggle - UI toggle to activate/deactivate staff (V1.0 #10) |
| └─ FN/ADM/STF/004-01 | ✅ Completed | | Add toggle button to staff card |
| └─ FN/ADM/STF/004-02 | ✅ Completed | | Create toggleStaffActive server action |
| └─ FN/ADM/STF/004-03 | ✅ Completed | | Add visual indicator for inactive staff |
| └─ FN/ADM/STF/004-04 | ✅ Completed | | Add filter to show/hide inactive staff |

##### LVE - Leave Module

| Jira ID | Status | Path | Description |
|---------|--------|------|-------------|
| **FN/ADM/LVE/001** | 📝 Planned | `/specs/FN/ADM/LVE/001-Leave_Management/` | Leave Management - SL/PH/AL and other leave types with calendar integration (V1.0 #2) |
| └─ FN/ADM/LVE/001-01 | ⬜ Not Started | | Define LeaveType enum and Leave Prisma model |
| └─ FN/ADM/LVE/001-02 | ⬜ Not Started | | Create PublicHoliday table and seed data |
| └─ FN/ADM/LVE/001-03 | ⬜ Not Started | | Build leave management UI |
| └─ FN/ADM/LVE/001-04 | ⬜ Not Started | | Integrate leaves as fixed OFF in solver |
| └─ FN/ADM/LVE/001-05 | ⬜ Not Started | | Display leaves on roster grid |

##### REQ - Staff Request Module

| Jira ID | Status | Path | Description |
|---------|--------|------|-------------|
| **FN/ADM/REQ/001** | 📝 Planned | `/specs/FN/ADM/REQ/001-Staff_Requests/` | Staff Requests - Day-off and no-Night requests with approval workflow (V1.0 #3) |
| └─ FN/ADM/REQ/001-01 | ⬜ Not Started | | Define StaffRequest Prisma model |
| └─ FN/ADM/REQ/001-02 | ⬜ Not Started | | Build staff request submission UI |
| └─ FN/ADM/REQ/001-03 | ⬜ Not Started | | Add "No Night" request type |
| └─ FN/ADM/REQ/001-04 | ⬜ Not Started | | Build manager approval workflow |
| └─ FN/ADM/REQ/001-05 | ⬜ Not Started | | Integrate approved requests as solver constraints |

##### RST - Roster Module

| Jira ID | Status | Path | Description |
|---------|--------|------|-------------|
| **FN/ADM/RST/001** | 🚧 In Progress | `/specs/FN/ADM/RST/001-Roster_Versioning/` | Roster Versioning - Version field and soft delete |
| **FN/ADM/RST/002** | 📝 Planned | `/specs/FN/ADM/RST/002-Previous_Month_Integration/` | Previous Month Integration - Load last month's final days to avoid scheduling conflicts (V1.0 #1) |
| └─ FN/ADM/RST/002-01 | ⬜ Not Started | | Schema extension to link rosters across months |
| └─ FN/ADM/RST/002-02 | ⬜ Not Started | | API to fetch previous month's final days |
| └─ FN/ADM/RST/002-03 | ⬜ Not Started | | Solver integration to consider prior month constraints |
| └─ FN/ADM/RST/002-04 | ⬜ Not Started | | UI to display previous month context |

##### SHF - Shift Module

| Jira ID | Status | Path | Description |
|---------|--------|------|-------------|
| **FN/ADM/SHF/001** | 📝 Planned | `/specs/FN/ADM/SHF/001-Hybrid_Duty_Generation/` | Hybrid Duty Generation - Combine A/7/P/E/N/custom shifts in single roster (V1.0 #6) |
| └─ FN/ADM/SHF/001-01 | ⬜ Not Started | | Add HYBRID shift type to ShiftType enum |
| └─ FN/ADM/SHF/001-02 | ⬜ Not Started | | Dynamic state expansion based on active ShiftDefinitions |
| └─ FN/ADM/SHF/001-03 | ⬜ Not Started | | Update solver to handle variable state count |
| └─ FN/ADM/SHF/001-04 | ⬜ Not Started | | UI for hybrid mode selection |

##### ROL - Role Module

| Jira ID | Status | Path | Description |
|---------|--------|------|-------------|
| **FN/ADM/ROL/001** | 📝 Planned | `/specs/FN/ADM/ROL/001-IC_Role_Mapping/` | IC Role Mapping - Auto-assign IC based on staff title + SRN/RN group balance (V1.0 #4) |
| └─ FN/ADM/ROL/001-01 | ⬜ Not Started | | Define rank-to-IC mapping rules |
| └─ FN/ADM/ROL/001-02 | ⬜ Not Started | | Create SRN/RN group balance constraints |
| └─ FN/ADM/ROL/001-03 | ⬜ Not Started | | Update solver for group-based distribution |
| └─ FN/ADM/ROL/001-04 | ⬜ Not Started | | Admin UI for mapping configuration |

---

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

##### I18N - Internationalization Module

| Jira ID | Status | Path | Description |
|---------|--------|------|-------------|
| **FN/FE/I18N/001** | 📝 Planned | `/specs/FN/FE/I18N/001-Language_Support/` | Language Support - Multi-language support (CN/TC/EN) with next-intl (V1.0 #11) |
| └─ FN/FE/I18N/001-01 | ⬜ Not Started | | Setup next-intl and locale routing |
| └─ FN/FE/I18N/001-02 | ⬜ Not Started | | Create translation files (en, zh-CN, zh-TW) |
| └─ FN/FE/I18N/001-03 | ⬜ Not Started | | Build LanguageSwitcher component |
| └─ FN/FE/I18N/001-04 | ⬜ Not Started | | Migrate all UI strings to translation keys |
| └─ FN/FE/I18N/001-05 | ⬜ Not Started | | Add locale-aware date/time formatting |

##### ONB - Onboarding Module

| Jira ID | Status | Path | Description |
|---------|--------|------|-------------|
| **FN/FE/ONB/001** | 📝 Planned | `/specs/FN/FE/ONB/001-First_Time_User_Experience/` | First-Time User Experience - User agreement + password reset on first login (V1.0 #12, depends on AUTH/001) |
| └─ FN/FE/ONB/001-01 | ⬜ Not Started | | Create Terms of Service page |
| └─ FN/FE/ONB/001-02 | ⬜ Not Started | | Implement agreement acceptance flow |
| └─ FN/FE/ONB/001-03 | ⬜ Not Started | | Add first-login detection |
| └─ FN/FE/ONB/001-04 | ⬜ Not Started | | Build password reset wizard |
| └─ FN/FE/ONB/001-05 | ⬜ Not Started | | Integration testing |
| **FN/FE/ONB/002** | 📝 Planned | `/specs/FN/FE/ONB/002-Initial_Roster_Entry/` | Initial Roster Entry - Manager enters last 3 days of previous month (V1.0 #13) |
| └─ FN/FE/ONB/002-01 | ⬜ Not Started | | Design manual roster entry form |
| └─ FN/FE/ONB/002-02 | ⬜ Not Started | | Implement roster data import API |
| └─ FN/FE/ONB/002-03 | ⬜ Not Started | | Build onboarding wizard step |
| └─ FN/FE/ONB/002-04 | ⬜ Not Started | | Validate and persist initial data |

---

## 🐛 Bug Index

### BG - Bugs

*No bugs documented yet.*

---

## 🔧 Chore Index

### CH - Chores

*No chores documented yet.*

---

## Version 1.0 Requirements Mapping

| V1.0 # | Description | JIRA ID | Status |
|--------|-------------|---------|--------|
| 1 | Last month duty interaction | FN/ADM/RST/002 | 📝 Planned |
| 2 | SL/PH/AL leave handling | FN/ADM/LVE/001 | 📝 Planned |
| 3 | Staff dayoff/no-Night requests | FN/ADM/REQ/001 | 📝 Planned |
| 4 | IC role mapping + SRN/RN balance | FN/ADM/ROL/001 | 📝 Planned |
| 5 | Customize special shifts | FN/ADM/STF/007 | ✅ Completed |
| 6 | Hybrid duty generation | FN/ADM/SHF/001 | 📝 Planned |
| 7 | Show shift counts/hours | FN/BE/ENG/002 | ✅ Completed |
| 8 | Manager/Member accounts | FN/ADM/AUTH/001 | 📝 Planned |
| 9 | Role-based permissions | FN/ADM/AUTH/002 | 📝 Planned |
| 10 | Staff active/inactive toggle | FN/ADM/STF/004 | ✅ Completed |
| 11 | Language Support (CN/TC/EN) | FN/FE/I18N/001 | 📝 Planned |
| 12 | First-time user experience | FN/FE/ONB/001 | 📝 Planned |
| 13 | Enter last 3 days of last month | FN/FE/ONB/002 | 📝 Planned |
| 14 | Annual Weekend/PH/Night balance | FN/BE/ENG/003 | 📝 Planned |

---

## Status Legend

- 🚧 **In Progress** - Actively being worked on
- ✅ **Completed** - Feature fully implemented and tested
- 📝 **Planned** - Spec folder created, awaiting implementation
- ⬜ **Not Started** - Subtask not yet begun
- 🔄 **Ready** - Ready for testing
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
- `tasks.md` - Jira subtask breakdown with checkboxes
