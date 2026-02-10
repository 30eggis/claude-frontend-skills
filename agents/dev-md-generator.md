---
name: dev-md-generator
description: "Generate DEV.md per page/route from elevated specs, screen specs, API map, and gap report. Used in spec-it-elevate P8."
model: opus
context: fork
permissionMode: bypassPermissions
allowedTools: [Read, Write, Glob, Grep]
references:
  - shared/references/elevate/dev-md-schema.md
  - shared/references/burn/scenario-principles.md
  - shared/references/elevate/state-model-principles.md
templates:
  - shared/templates/elevate/dev-md-template.md
---

# DEV.md Generator

Generates self-contained DEV.md files for each page/route, replacing sprint packets as the primary implementation artifact.

## Role

Transform elevated specs, screen specifications, API map, and analysis artifacts into one DEV.md per page that contains ALL context needed for implementation.

## Input

| Artifact | Path | Purpose |
|----------|------|---------|
| IA.md | `07-ia/IA.md` | Route structure + depths + dependencies |
| Screen specs | `02-prd/03-screen-specifications/` | Page details |
| Elevated components | `05-elevated/` | Component specs |
| API map | `06-review/api-map.md` | API endpoints |
| Gap report | `04-analysis/gap-report.md` | Missing items |
| Permission matrix | `04-analysis/permission-matrix.md` | Access control |
| Canonical enum map | `04-analysis/canonical-enum-map.yaml` | Enum references |
| Business rules | `04-analysis/business-rule-overlay.md` | Validations |
| State models | `04-analysis/state-models/` | State machines |
| Spec map | `06-review/spec-map.md` | Master index |

## Process

### 1. Parse IA.md

```
Read IA.md → extract route list with:
  - route path
  - title
  - persona
  - depth
  - dependencies
```

### 2. Generate DEV.md per Route

```
FOR each route in IA.md:
  1. Find matching screen spec
  2. Find matching elevated components
  3. Find matching API endpoints from api-map
  4. Find matching business rules
  5. Find matching permission requirements
  6. Find matching state models
  7. Find matching enum references
  8. IF exploration data has mockup HTML mapping:
     Find matching HTML file from 00-exploration/route-map.md
     Add to DEV.md header: **Source HTML:** {absolute path to .html file}

  Compile DEV.md following dev-md-schema.md
```

> **Source HTML** 필드는 mockup HTML 파일의 절대경로.
> 이 필드가 있으면 burn B2에서 원본 대비 1:1 비교가 활성화됨.
> Mockup이 제공되지 않은 경우 이 필드는 생략되며, WIREFRAME.md만 참조.

### 3. Build Sections

#### API Endpoints
```
FROM api-map.md:
  Filter endpoints used by this page
  Map each to: Action, Method, Endpoint, Trigger
```

#### API-UI Binding
```
FOR each API endpoint:
  Determine trigger (on-loaded, click-to-{btn}, submit-{form})
  Determine UI update (render, toast, refetch, navigate)
  Write pseudo-code binding line
```

#### Scenarios
```
Apply scenario-principles.md:
  - Happy path for each API-UI binding
  - Error state for each API call
  - Empty state for data displays
  - Validation failure for each form
  - Permission guard for restricted actions
  - Edge cases from gap-report
```

#### UI/UX State Changes
```
FROM component states (component-ia-principles §2):
  - loading: skeleton/spinner per data section
  - error: error banner/message per API
  - empty: empty state per data section
  - submitting: disabled buttons during API calls
```

#### Validation
```
FROM business rules + gap report:
  - Form field validations
  - Submit preconditions
  - Confirmation dialogs
```

#### TDD Plan
```
FROM scenarios:
  Unit: component rendering + state assertions
  Integration: API flow + data binding
  E2E: full user workflow scenarios
```

### 4. Generate WIREFRAME.md

```
FOR each route:
  IF UI mode is generative:
    Extract key UI elements from screen spec + elevated components
    Write wireframe description (layout, components, hierarchy)
  IF UI mode is original:
    Reference original HTML from mockup exploration
    Include verbatim HTML snippets
```

## Output

```
{sessionDir}/08-dev/
├── {route-group}/
│   └── {page}/
│       ├── DEV.md
│       └── WIREFRAME.md
└── dev-index.md          # Index of all DEV.md files with routes/depths
```

## DEV.md Self-Containment Rules

Each DEV.md must include:
- ALL API endpoints the page uses (inline, not references)
- ALL scenarios with concrete steps and expected results
- ALL validation rules with conditions and effects
- ALL state changes with triggers and UI effects
- TDD plan with specific test assertions

The developer (or dev-executor agent) should NEVER need to read other files to understand what to implement.

## Quality Checks

```
FOR each generated DEV.md:
  ✓ All API endpoints exist in api-map.md
  ✓ All scenarios follow scenario-principles.md types
  ✓ All data components have 4 states (default, loading, error, empty)
  ✓ Dependencies match IA.md
  ✓ Permission roles match permission-matrix.md
  ✓ Enum values match canonical-enum-map.yaml
```

## Writing Location

`tmp/{session-id}/08-dev/`
