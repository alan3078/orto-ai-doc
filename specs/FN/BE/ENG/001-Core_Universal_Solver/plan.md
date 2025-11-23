# Plan - FN/BE/ENG/001 - Core Universal Solver (MVP)

## 1. Overview

This feature implements the foundational "Math Engine" using Google OR-Tools. The goal is to create a generic API that accepts a standardized JSON input defining resources, time slots, and basic mathematical constraints, and returns a valid schedule.

## 2. Goals

**Decoupling**: The engine must not contain domain logic (no "Nurse", "Shift", "Patient"). It uses abstract terms: Resource, TimeSlot, State.

**Security**: Use a "Safe Parser" approach (Math JSON). We will not use `exec()` or dynamic code injection to ensure this can be safely exposed as a SaaS API.

**Extensibility**: The JSON schema must support future constraint types without breaking existing logic.

## 3. Scope (MVP)

For this first version, we will support:

- **Basic Configuration**: Defining Resources (Rows), Time Slots (Columns), and States (Cell Values).
- **Constraint Type A - Point**: "Resource X at Time Y must be State Z" (e.g., Day Off).
- **Constraint Type B - Summation**: "Sum of State Z > Target" (e.g., Min 3 people working).

## 4. Out of Scope

- Sliding Window / Sequence Patterns (Reserved for FN-002).
- User Authentication (Reserved for ADM module).
- Frontend UI.

## 5. Success Criteria

- We can send a JSON payload via Postman/cURL.
- The system returns a JSON matrix where all constraints are satisfied.
- If constraints are impossible, the system returns a "Infeasible" error status.
