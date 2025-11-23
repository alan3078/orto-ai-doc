# Tasks - FN/BE/ENG/001 - Core Solver MVP

## Context

We are building the backend mathematical engine. No UI, no Database yet. Just API Input → Math → API Output.

## Jira Ticket Breakdown

### FN/BE/ENG/001-01: Project Init & Scaffold

- [x] Initialize Python project (Poetry or venv).
- [x] Install `fastapi`, `uvicorn`, `ortools`, `pydantic`.
- [x] Create basic folder structure `src/features/solver_engine`.

### FN/BE/ENG/001-02: Define Pydantic Models

- [x] Create `schemas.py`.
- [x] Define `ConfigModel`, `ConstraintModel`, `SolveRequest`, `SolveResponse`.
- [x] Ensure input validation (e.g., value cannot be negative).

### FN/BE/ENG/001-03: Implement Variable Initialization Logic

- [x] Create `service.py`.
- [x] Write function to initialize `cp_model` and the `shifts` variable matrix based on config.

### FN/BE/ENG/001-04: Implement "Point" & "Sum" Constraints

- [x] Write parser for `point` type constraint.
- [x] Write parser for `vertical_sum` constraint (handling the BoolVar conversion).
- [x] **Self-Check**: Does it handle ALL time slots vs specific time slot?

### FN/BE/ENG/001-05: API Endpoint & Integration

- [x] Create `router.py` with `POST /solve`.
- [x] Connect Router to Service.
- [x] **Manual Test**: Send a JSON to generate a 3-person, 2-day schedule where 1 person must be OFF.

### FN/BE/ENG/001-06: Unit Tests

- [x] Write a test case for an "Infeasible" scenario (Impossible rules) to ensure it returns proper error status, not a crash.
