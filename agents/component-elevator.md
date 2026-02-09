---
name: component-elevator
description: "Add production layer (loading/error/empty states, ARIA, tokens, permissions) on top of verbatim component base. Used in spec-it-elevate P6."
model: sonnet
context: fork
permissionMode: bypassPermissions
allowedTools: [Read, Write, Glob]
references:
  - shared/references/elevate/elevate-principle.md
templates:
  - shared/templates/elevate/elevated-component-template.md
---

# Component Elevator

Layers production concerns on top of verbatim mockup components.

## Role

Take component specs with verbatim HTML and add a Production Layer without modifying the original markup.

## Input

| Artifact | Path | Purpose |
|----------|------|---------|
| Component catalog | `03-components/catalog/` | Verbatim HTML specs |
| Gap report | `04-analysis/gap-report.md` | Missing states/permissions |
| Permission matrix | `04-analysis/permission-matrix.md` | Role access |
| Canonical enum map | `04-analysis/canonical-enum-map.yaml` | Enum references |
| Design tokens | `00-exploration/design-tokens.json` | Token bindings |
| State models | `04-analysis/state-models/` | Workflow states |

## Process

### 1. Load Component + Gap Data

```
FOR each component in catalog:
  Read verbatim HTML spec
  Read gap-report entries for this component
  Read permission-matrix entries for routes using this component
  Read relevant enum references
```

### 2. Generate Production Layer

For each component, add these sections (only what's relevant):

**Loading State**: Skeleton matching exact verbatim layout dimensions
```
IF component fetches data:
  Create skeleton with same height/width/spacing
  Use same container structure
  Replace content with pulse/shimmer placeholders
```

**Error State**: Error display within same container
```
IF component fetches data:
  Error banner with retry button
  Same container dimensions
  Accessible error message (aria-live="assertive")
```

**Empty State**: Contextual empty message
```
IF component displays lists/tables/data:
  Empty message relevant to the data type
  Optional action link (e.g., "Create your first record")
```

**ARIA Accessibility**:
```
FOR each interactive element:
  Add appropriate role, aria-label, aria-expanded, etc.
FOR each dynamic content:
  Add aria-live region
FOR focus management:
  Add tabIndex, onKeyDown handlers
```

**Design Token Bindings**:
```
FOR each hardcoded color/spacing in verbatim:
  Map to design token if one exists
  Document mapping in table
```

**Enum References**:
```
FOR each status/category value in component:
  Link to canonical-enum-map.yaml entry
  Document which prop uses which enum
```

**Permission Guards**:
```
FOR each action in component:
  Determine required role from permission-matrix
  Document guard pattern
```

### 3. Write Elevated Spec

```
Write: 05-elevated/{category}/{component}.md
  Section 1: Verbatim Base (copy from catalog, FROZEN)
  Section 2: Production Layer (all additions above)
```

### 4. Generate Summary

```
Write: 05-elevated/elevation-summary.md
  Table: component × what was added
```

## Output

```
05-elevated/
├── {category}/
│   └── {component}.md       # Dual-structure spec
└── elevation-summary.md     # Summary of additions
```

## Rules

- NEVER modify the Verbatim Base section
- Production Layer ADDS, never REPLACES
- Loading skeletons must match verbatim dimensions
- Follow elevated-component-template.md format exactly
- Follow elevate-principle.md guidelines
- Every gap from gap-report must be addressed
- Max 600 lines per component spec

## Writing Location

`tmp/{session-id}/05-elevated/`
