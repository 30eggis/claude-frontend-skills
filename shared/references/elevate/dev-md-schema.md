# DEV.md Schema

DEV.md is a self-contained development specification per page/route. It serves as the single source of truth during implementation, replacing sprint packets.

---

## Header (Required)

```markdown
# DEV: {Page Name}
> Route: /{path}
> Persona: {persona}
> Depth: {N}
> Dependencies: [{dep1}, {dep2}] or []
```

| Field | Type | Description |
|-------|------|-------------|
| Page Name | string | Human-readable page name |
| Route | string | Full route path (e.g., /(admin)/attendance/records) |
| Persona | string | Primary persona for this page |
| Depth | number | URL segment count (0=dashboard, 1=list, 2=detail, 3=sub-action) |
| Dependencies | string[] | Routes that must be implemented first |

---

## Sections (Required)

### 1. UI Mode
```
{generative | original}
```
- `generative`: Reference wireframe, generate new production code
- `original`: Extract existing HTML verbatim, refactor to match spec

### 2. API Endpoints
| Column | Description |
|--------|-------------|
| Action | What this API call does |
| Method | HTTP method |
| Endpoint | API path |
| Trigger | What triggers this call (on-loaded, click-to-{btn}, etc.) |

### 3. API-UI Binding
Pseudo-code showing trigger → API call → UI update chain:
```
{trigger} => {METHOD} {endpoint} => {ui-action}
```
Each line represents one user interaction flow.

### 4. Scenarios
| Column | Description |
|--------|-------------|
| # | Sequential number |
| Scenario | Short description |
| Steps | User actions |
| Expected | Expected outcome |

Minimum: happy path, loading, error, empty states per page.

### 5. UI/UX State Changes
| Column | Description |
|--------|-------------|
| State | UI state name |
| Trigger | What causes this state |
| UI Change | What the UI shows |

Must cover: loading, error, empty, submitting states.

### 6. Validation
| Column | Description |
|--------|-------------|
| Rule | Validation rule name |
| Condition | When validation triggers |
| Effect | UI effect (disable button, show error, etc.) |

### 7. TDD Plan
Checklist format organized by test level:
- Unit Tests: Component rendering, props, state assertions
- Integration Tests: API integration, data flow
- E2E Tests: Full user scenarios

---

## Companion Files

| File | Purpose | Created By |
|------|---------|------------|
| WIREFRAME.md | UI spec (generative) or HTML reference (original) | dev-md-generator |
| TDD.md | Detailed test plan with code templates | test-spec-generator |
| page.test.tsx | Pre-generated test file | test-spec-generator |

---

## Lifecycle

1. **Created**: elevate P8 (dev-md-generator)
2. **Consumed**: burn B2 (dev-executor reads DEV.md as sole input)
3. **Archived**: burn B4 (essential info preserved in AGENT.md, DEV.md deleted)

---

## Rules

- DEV.md must be self-contained (developer needs no other files)
- One DEV.md per route/page
- All API endpoints must reference api-map.md
- All scenarios must be testable
- Dependencies must match IA.md depth rules
