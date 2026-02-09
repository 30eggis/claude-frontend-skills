# Gap Analysis Guide

How to find what's MISSING in a mockup for production readiness.

---

## 1. State Coverage Gaps

Every data-displaying component must have these states:

| State | Check | Common Gaps |
|-------|-------|-------------|
| **Loading** | Does the mockup show a loading indicator? | Usually missing — needs skeleton |
| **Error** | Does the mockup show error handling? | Usually missing — needs error banner |
| **Empty** | Does the mockup show an empty data state? | Often missing — needs empty message |
| **Partial** | What if only some data loads? | Rarely covered — needs graceful degradation |
| **Stale** | What if data is outdated? | Needs refresh indicator or auto-refetch |

### Detection Method

```
FOR each component with data binding:
  1. Check mockup: does it show loading/error/empty variants?
  2. If NO → add to gap-report as "missing state: {component}.{state}"
  3. Recommend skeleton shape matching the verbatim layout
```

---

## 2. Permission Gaps

Every action and route must have permission checks:

| Element | Check | Gap If Missing |
|---------|-------|----------------|
| **Routes** | Who can access this page? | Needs role-based route guard |
| **Buttons** | Who can click this? | Needs conditional rendering |
| **Data** | Who can see this data? | Needs field-level filtering |
| **Actions** | Who can perform this? | Needs API-level authorization |

### Detection Method

```
FOR each screen:
  1. List all actions (buttons, links, forms)
  2. Cross-reference with persona definitions
  3. If action has no persona restriction → add to gap-report
  4. Build permission-matrix.md: Role x Route grid
```

### Permission Matrix Format

```markdown
| Route/Action | Admin | Manager | Employee |
|-------------|-------|---------|----------|
| /dashboard | RW | R | - |
| /settings | RW | - | - |
| Approve leave | Yes | Yes | - |
| Edit own record | - | - | Yes |
```

---

## 3. Business Rule Gaps

Mockups show the UI but rarely encode business rules:

| Rule Type | Where to Look | Gap If Missing |
|-----------|--------------|----------------|
| **Validation** | Form fields, input masks | Min/max, required, format rules |
| **Calculation** | Displayed numbers, totals | Formula, rounding, timezone |
| **Workflow** | Status badges, action buttons | State machine transitions |
| **Constraints** | Date pickers, dropdowns | Range limits, mutual exclusions |
| **Conditional UI** | Show/hide sections | Trigger conditions |

### Detection Method

```
FOR each form/input:
  1. What validation rules apply? (length, format, range)
  2. Are there dependent fields? (select A changes options in B)
  3. What happens on submit? (success/error flows)

FOR each status/badge:
  1. What are ALL possible values?
  2. What transitions are allowed?
  3. Who can trigger each transition?
```

---

## 4. Enum Normalization Gaps

Mockups often use inconsistent labels for the same concept:

| Problem | Example | Fix |
|---------|---------|-----|
| **Synonym variation** | "Active", "active", "ACTIVE" | Canonical: `ACTIVE` |
| **Language mixing** | "승인", "Approved" | Pick one, map the other |
| **Implicit enums** | Badge colors without labels | Define explicit enum values |
| **Status inconsistency** | "pending" vs "대기" vs "Waiting" | Single canonical value |

### Detection Method

```
1. Scan all status badges, select options, filter values
2. Group by semantic meaning
3. If same concept has 2+ labels → add to canonical-enum-map.yaml
4. Choose canonical value, list all aliases
```

### Canonical Enum Map Format (YAML)

```yaml
enums:
  attendanceStatus:
    canonical: [PRESENT, LATE, ABSENT, LEAVE, HALF_DAY]
    aliases:
      PRESENT: ["출근", "Present", "정상"]
      LATE: ["지각", "Late", "Tardy"]
      ABSENT: ["결근", "Absent", "미출근"]
    display:
      PRESENT: { label: "출근", color: "green", icon: "check" }
      LATE: { label: "지각", color: "amber", icon: "clock" }
    usedIn:
      - component: StatusBadge
        prop: status
      - screen: /dashboard
        element: stats-card
```

---

## 5. State Machine Gaps

Workflows implied by mockup UI elements:

```
FOR each entity with status:
  1. List all visible statuses from mockup
  2. Identify transition triggers (buttons, actions)
  3. Draw state machine diagram
  4. Check: are all transitions accounted for?
  5. Check: can you reach every state from initial?
  6. Check: are there dead-end states?
```

### State Model Format

```markdown
# State Model: Leave Request

## States
| State | Display | Entry Condition |
|-------|---------|-----------------|
| DRAFT | "임시저장" | Created by employee |
| PENDING | "대기" | Submitted for approval |
| APPROVED | "승인" | Manager approved |
| REJECTED | "반려" | Manager rejected |
| CANCELLED | "취소" | Employee cancelled |

## Transitions
| From | To | Trigger | Guard |
|------|-----|---------|-------|
| DRAFT | PENDING | Submit | All required fields filled |
| PENDING | APPROVED | Approve | role=MANAGER |
| PENDING | REJECTED | Reject | role=MANAGER, reason required |
| PENDING | CANCELLED | Cancel | owner only, before approval |

## Diagram
DRAFT → PENDING → APPROVED
                → REJECTED
                → CANCELLED
```

---

## Output Checklist

After gap analysis, ensure these artifacts exist:

- [ ] `gap-report.md` — All identified gaps with severity
- [ ] `permission-matrix.md` — Complete Role x Route/Action grid
- [ ] `state-models/{workflow}.md` — For each entity with status
- [ ] `business-rule-overlay.md` — Condition → validation → UI mapping
- [ ] `canonical-enum-map.yaml` — Single source of truth for all enums
