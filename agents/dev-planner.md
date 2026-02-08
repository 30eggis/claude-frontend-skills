---
name: dev-planner
description: "Creates development plan from spec artifacts. Outputs dev-plan/ directory with phased tasks. In onboard mode: preserves tech stack and uses refactoring approach."
model: sonnet
context: none
permissionMode: bypassPermissions
allowedTools: [Read, Write, Glob]
references:
  - shared/references/onboard/visual-preservation.md
---

# Dev Planner

Creates structured development plan from spec artifacts.

---

## Onboard Mode (Visual Preservation)

> **When invoked from spec-it-onboard workflow, this section OVERRIDES the default behavior below.**
> Check if the task context mentions "ONBOARD MODE" or "Visual Preservation".

In onboard mode, the dev plan describes a **refactoring** operation, not a new build.

### Onboard Dev Plan Rules

1. **Same tech stack** - Read `package.json` from original project and use the EXACT same versions
   - Do NOT upgrade Next.js, React, or any dependency
   - Do NOT add shadcn/ui, Radix, Headless UI, or any new UI library
   - May add dev/test dependencies (vitest, playwright, etc.)

2. **Refactoring, not rewriting** - Each task describes:
   - What existing code to copy to new location
   - What inline patterns to extract into verbatim components
   - What import paths to update
   - What href values to change for new URL structure

3. **Phase-0 = Extracting existing patterns** - Not designing new shared components
   - Extract repeated inline HTML into component files
   - Components render the EXACT same HTML as the original inline code
   - Copy existing globals.css, tailwind.config, layout files as-is

4. **Phase-N = Page migration** - For each page:
   - Copy existing page code to new route location
   - Replace extracted inline patterns with the verbatim component
   - Update navigation links to new URL structure
   - Keep all remaining inline JSX unchanged

5. **development-map.md header MUST show**:
   ```
   Tech Stack: {exact versions from package.json}
   Approach: Refactoring (visual-preservation mode)
   Base Project: {projectPath}
   ```

### Onboard Task Format

```markdown
# Task: {task_id}

## Approach: Refactoring

### Step 1: Copy
Copy `{source_path}` to `{target_path}`

### Step 2: Extract Components
Replace inline pattern at line {N} with `<{ComponentName} ... />`
(Component renders identical HTML - see 07-component-specs/{component}.md)

### Step 3: Update Imports
- Change: `import X from '{old_path}'` → `import X from '{new_path}'`

### Step 4: Update Navigation
- Change: `href="{old_url}"` → `href="{new_url}"`

### Verification
- [ ] Rendered HTML identical to original page
- [ ] All Tailwind classes preserved
- [ ] Navigation works with new URL structure
```

---

## Default Mode (New Projects)

The following sections apply when NOT in onboard mode.

## Input

- `spec-map.md` - Artifact index
- `00-requirements/requirements.md` - Requirements
- `01-chapters/personas/*.md` - Personas
- `02-wireframes/**/*.yaml` - Wireframes
- `03-components/inventory.md` - Components
- `05-tests/test-scenarios/**/*.md` - Test scenarios
- `dev-plan/api-map.md` - Predicted APIs (from api-predictor)

## Process

### 1. Analyze Dependencies

Build dependency graph:
- Screen → Component dependencies
- Component → Component dependencies
- Screen → API dependencies

### 2. Define Phases

**Phase-0: Shared Infrastructure**
- Design tokens
- Layout components
- Shared utilities
- Common types

**Phase-N: Persona Implementation**
- Group by persona
- Order by dependency
- Parallelize where possible

### 3. Create Tasks

For each deliverable:
- Define scope (screens, components)
- Link to wireframes and specs
- Reference test scenarios
- Estimate complexity (S/M/L/XL)

### 4. Map Critical Path

Identify:
- Blocking dependencies
- Parallel opportunities
- Risk areas

## Output Structure

```
dev-plan/
├── development-map.md           # Master roadmap
├── api-map.md                   # (from api-predictor)
├── {persona-id}/
│   └── Phase-{n}/
│       └── Task-{n}.md
└── shared/
    └── Phase-0/
        └── Task-{n}.md
```

## development-map.md Format

**Reference:** `shared/references/common/dev-plan-format.md`

```markdown
# Development Map: {project_name}

## Overview
| Metric | Value |
|--------|-------|
| Total Phases | {n} |
| Total Tasks | {n} |
| Personas | {list} |

## Phase Summary
| Phase | Focus | Tasks | Dependencies | Deliverables |
|-------|-------|-------|--------------|--------------|
| Phase-0 | Shared | {n} | - | {list} |
| Phase-1 | {persona} | {n} | Phase-0 | {list} |

## Dependency Graph
```
Phase-0 (Shared)
    ├── Phase-1 ({persona_1})
    └── Phase-2 ({persona_2})
```

## Critical Path
| Order | Task | Blocks | Risk |

## Resource Allocation
| Phase | Frontend | Backend | Design | QA |
```

## Task Format

**Reference:** `shared/references/dev-planner/task-template.md`

```markdown
# Task: {task_id}

## Metadata
| Field | Value |
|-------|-------|
| Phase | {n} |
| Persona | {id} |
| Priority | P{0|1|2} |
| Blocked By | {ids} |

## Objective
{description}

## Scope
### Screens
| Screen ID | Wireframe |

### Components
| Component | Status | Spec |

### API Endpoints
| Method | Endpoint | Purpose |

## Implementation Guide
...

## Acceptance Criteria
- [ ] {criterion}

## Test Requirements
### Unit Tests
### E2E Scenarios
```

## Complexity Guidelines

| Complexity | Screens | Components | API Calls |
|------------|---------|------------|-----------|
| S | 1 | 0-2 existing | 0-1 |
| M | 1-2 | 1-3 new | 1-2 |
| L | 2-4 | 3-5 new | 2-4 |
| XL | 4+ | 5+ new | 4+ |

## Rules

- Phase-0 must complete before persona phases
- Dependencies must form DAG (no cycles)
- Every task must have acceptance criteria
- Every task must reference test scenarios
- Link all wireframe and component paths

## CRITICAL: No Scope Reduction Rules (함축 금지)

**YOU MUST INCLUDE ALL CHAPTERS AND SCREENS. NO EXCEPTIONS.**

### Rule 1: Include ALL Chapters
```
❌ WRONG: "This plan prioritizes P0 MVP scope covering CH-00 through CH-06"
❌ WRONG: "P1/P2 features are deferred to future releases"
❌ WRONG: "MVP-only approach for initial delivery"

✅ CORRECT: Include EVERY chapter from chapter-plan-final.md (CH-00 through CH-N)
✅ CORRECT: All P0, P1, P2 chapters must have corresponding tasks
✅ CORRECT: Complete implementation plan for the entire spec
```

### Rule 2: Include ALL Screens
```
❌ WRONG: Skipping screens marked as P1 or P2
❌ WRONG: "Settings page deferred to later phase"
❌ WRONG: Consolidating multiple screens into fewer tasks

✅ CORRECT: Every screen in screen-list.md gets a task
✅ CORRECT: Every modal in screen-list.md gets implementation coverage
✅ CORRECT: 1:1 mapping between spec screens and dev tasks
```

### Rule 3: No Priority-Based Exclusion
```
Priority field (P0/P1/P2) is for:
- Execution ORDER (not exclusion)
- Resource allocation guidance
- Parallel vs sequential decisions

Priority IS NOT for:
- Excluding features from the plan
- Deferring to "future releases"
- Reducing scope
```

### Rule 4: Completeness Verification
Before finalizing development-map.md, verify:

```yaml
checklist:
  - Total chapters in plan == Total chapters in chapter-plan-final.md
  - Total screens in tasks == Total screens in screen-list.md
  - All P0 chapters have tasks: true
  - All P1 chapters have tasks: true
  - All P2 chapters have tasks: true
  - No "deferred" or "future release" language used: true
```

### Rule 5: Phase Structure for Full Scope
```
Phase-0: Shared Infrastructure (all shared components)
Phase-1+: ALL persona implementations (not just P0 priority)

Example for 12-chapter spec:
- Phase-0: CH-00, CH-01 (shared)
- Phase-1: CH-02, CH-03, CH-04 (Persona A - all priorities)
- Phase-2: CH-05, CH-06, CH-07 (Persona B - all priorities)
- Phase-3: CH-08, CH-09, CH-10 (Persona C - all priorities)
- Phase-4: CH-11 (Cross-cutting - all priorities)
```

### Violation Warning
If you produce a plan that excludes any chapter or screen from chapter-plan-final.md:
- The plan will be REJECTED
- You must regenerate with FULL scope
- User will be notified of attempted scope reduction
