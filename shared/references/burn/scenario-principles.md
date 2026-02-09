# Scenario Generation Principles

> Source: tna Page Contracts (39 pages, user actions), Sprint Packet scenario system

## 1. Scenario = User Action Sequence
All scenarios follow "When user {trigger}, then {expected result}" format.
Abstract descriptions prohibited. Concrete actions and results only.

## 2. Scenario Classification
| Type | Description | Example |
|------|-------------|---------|
| Happy Path | Normal flow | View attendance records → list displays |
| Validation | Input validation | Empty reason for rejection → error message |
| Error | API/system error | Server 500 → error banner + retry |
| Empty | No data | Zero results → guidance message |
| Permission | Insufficient access | Regular employee approves → button disabled |
| Edge | Boundary conditions | 1000-item pagination, concurrent approval |

## 3. Scenario Completeness Checklist
Minimum scenarios per page:
- [ ] Initial load (happy path)
- [ ] Loading state
- [ ] Error state (API failure)
- [ ] Empty state (no data)
- [ ] Each interactive element's click/submit result
- [ ] Each form's validation failure case
- [ ] Permission guard (if applicable)

## 4. API-UI Binding → Scenario Mapping
One line in DEV.md API-UI Binding = 1+ scenarios:
```
on-loaded => GET /api/stats => render(cards)
→ Scenario: "Page load shows 5 stat cards"
→ Scenario: "API failure shows error banner"
→ Scenario: "Zero data shows empty state message"
```

## 5. Scenario → Test Conversion Rules
| Scenario Type | Test Level | Priority |
|--------------|-----------|----------|
| Happy Path | Unit + E2E | P0 |
| Validation | Unit | P0 |
| Error | Integration | P1 |
| Empty | Unit | P1 |
| Permission | Integration | P1 |
| Edge | E2E | P2 |
