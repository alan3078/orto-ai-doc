# Plan - FN/BE/ENG/002 - Roster Validator (Audit Mode)

## 1. Overview

This feature adds a "Quality Assurance" layer to the engine. It allows users to submit an existing roster (generated or manually edited) along with a set of constraints, and receive a detailed "Report Card" indicating which constraints are met and which are violated.

## 2. Goals

- **Transparency**: Explain why a roster is good or bad with specific constraint-level feedback
- **Manual Edit Safety**: If a manager manually changes Amy's shift from "Day" to "Night", they can press "Check Rules" to see if they broke the "Max 3 Nights" rule
- **Detailed Feedback**: Instead of just "Infeasible", return "Constraint #4 (Min 2 Seniors) failed on June 5th"
- **Trust Building**: Help users understand the solver's decisions and validate manual changes

## 3. Scope (MVP)

### Backend (Python Engine)

- **New API Endpoint**: `POST /api/v1/validate` in Python Engine
- **Validation Logic**: Imperative Python functions to check each constraint type (Point, VerticalSum, HorizontalSum, SlidingWindow, PatternBlock, AttributeVerticalSum, ResourceStateCount) against a provided matrix
- **Reporting**: A JSON response format listing every rule with `status: "PASS" | "FAIL"` and `details` explaining violations

### Frontend (Next.js Web App)

- **Service Integration**: Extend `SolverIntegrationService` with `validateRoster()` method
- **Server Action**: Create `validateRosterAction(rosterId)` to orchestrate DB → Python → Response
- **UI Components**: 
  - "Check Rules" button on Roster Management and Test Roster pages
  - `ValidationDrawer` component (side panel) showing validation results
  - `ConstraintReportCard` component for individual constraint pass/fail/details
- **Visual Feedback**: Pass/Fail badges, violation counts, detailed error messages

## 4. Out of Scope

- **Auto-fixing violations**: The Solver does this; the Validator only reports
- **Real-time checking**: Check while dragging is deferred to post-MVP
- **Partial validation**: MVP validates all constraints; selective validation deferred
- **Validation history**: No persistence (`ValidationLog` table) - results computed on-demand
- **Constraint suggestions**: No AI-powered fix recommendations

## 5. Technical Decisions

### 1. Caching Strategy
**Decision**: On-demand computation (no persistence)  
**Rationale**: 
- Rosters don't change frequently once generated
- Validation is fast (< 1s for typical 30-day roster)
- Simpler architecture for MVP
- Add `ValidationLog` table later if performance degrades

### 2. Validation Scope
**Decision**: Always validate all constraints  
**Rationale**: 
- System still in active development
- Comprehensive audit builds trust
- Simple UX (one button, full report)
- Partial validation can be added post-MVP via constraint filtering

### 3. Validation Trigger
**Decision**: Only validate rosters saved in DB  
**Rationale**: 
- Validates both generated rosters (COMPLETED status) and manually edited ones
- No real-time validation during UI drag-and-drop (out of scope)
- Clear separation: Solver creates, Validator audits

## 6. Success Criteria

### Functional
- [ ] User clicks "Check Rules" button on a roster page
- [ ] System validates roster against all active constraints
- [ ] Validation drawer opens showing:
  - Overall summary (X/Y constraints passed)
  - Individual constraint cards with PASS/FAIL badges
  - Detailed violation messages (e.g., "Amy has 4 consecutive nights on June 3-6, violates max 3")
- [ ] Failed constraints show specific time slots and resources involved
- [ ] Validation completes in < 2 seconds for 30-day roster with 20 constraints

### Technical
- [ ] Python `/api/v1/validate` endpoint returns structured JSON
- [ ] All 7 constraint types have validator functions
- [ ] Frontend correctly handles validation errors (API down, invalid roster ID)
- [ ] UI is responsive and accessible (keyboard navigation, screen readers)

### UX
- [ ] Clear visual distinction between PASS (green) and FAIL (red) states
- [ ] Violation messages are human-readable (no technical jargon)
- [ ] Drawer is scrollable for long constraint lists
- [ ] Loading state shown during validation

## 7. Dependencies

- **Prerequisite**: FN/BE/ENG/001 (Core Universal Solver) must be completed
- **Prerequisite**: FN/BE/DATA/003 (Data Modeling) must have Roster and Constraint tables
- **Prerequisite**: FN/FE/UI/004 (MVP Dashboard) must have roster display pages

## 8. Timeline Estimate

- **Backend (Python)**: 2-3 days
  - Validator service: 1.5 days
  - API endpoint + schemas: 0.5 day
  - Unit tests: 1 day
- **Frontend (Next.js)**: 2-3 days
  - Service + action: 0.5 day
  - UI components: 1.5 days
  - Integration + testing: 1 day
- **Total**: 4-6 days for complete feature

## 9. Future Enhancements (Post-MVP)

- **Validation History**: Persist results in `ValidationLog` table with timestamps
- **Partial Validation**: Filter constraints by type, priority, or custom groups
- **Real-time Validation**: WebSocket-based live feedback during manual edits
- **Fix Suggestions**: AI-powered recommendations to resolve violations
- **Bulk Validation**: Validate multiple rosters simultaneously
- **Severity Levels**: Categorize violations as CRITICAL, WARNING, INFO
- **Export Reports**: Download validation results as PDF/CSV
