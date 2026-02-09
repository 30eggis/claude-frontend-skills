# State Model Principles

> Source: tna 5 State Machines + 86 Business Rules + Permission Matrix

## 1. State Machine Definition Criteria
Create State Machine document when:
- Entity has 3+ states (e.g., attendance record, leave request)
- State transitions have business rules
- Transitions require permission checks

## 2. State Machine Structure
```
[StateA] --{event, condition, role}--> [StateB]
```
| Field | Description |
|-------|-------------|
| states | All possible states (defined as enum) |
| transitions | List of state transitions |
| transition.from | Source state |
| transition.to | Target state |
| transition.event | Trigger (user action or system event) |
| transition.guard | Transition condition (business rule) |
| transition.role | Roles that can perform |

## 3. Business Rule Structure
| Field | Description |
|-------|-------------|
| id | BR-{domain}-NN |
| condition | Rule application condition |
| validation | Validation logic |
| ui-effect | UI impact (error message, disable, etc.) |
| api-effect | API impact (400 response, field validation, etc.) |

## 4. Permission Matrix
| Route | Admin | Manager | Employee |
|-------|-------|---------|----------|
| /dashboard | CRUD | Read | Read |
| /attendance | CRUD | Read+Approve | Read+Create |

- C = Create, R = Read, U = Update, D = Delete
- Complex permissions: Approve, Export as separate columns

## 5. DEV.md State Reflection
DEV.md "UI/UX State Changes" section:
- Each State Machine state → mapped to UI state
- Transition events → mapped to triggers
- Guard conditions → mapped to validation rules
