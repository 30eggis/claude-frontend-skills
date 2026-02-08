---
name: spec-it-onboard
description: "Onboard existing Next.js mockup into spec-it workflow. Transform mockups to production-ready specifications."
allowed-tools: Read, Write, Edit, Bash, Task, AskUserQuestion
argument-hint: "[project-path] [--session <sessionId>] [--resume <sessionId>]"
permissionMode: bypassPermissions
---

# spec-it-onboard: Mockup to Spec Workflow

Transform existing Next.js mockup projects into production-ready specifications.

**Input**: Next.js codebase (*.tsx)
**Output**: Complete spec-it artifacts ready for spec-it-execute

## Core Principle: Visual Preservation

> **Onboard = Refactoring, NOT Rewriting.**
>
> The rendered HTML of every page MUST remain pixel-identical to the original mockup.
> Internal code may be reorganized (componentized, re-routed, restructured),
> but the user-visible output MUST NOT change.

See [shared/references/onboard/visual-preservation.md](../../shared/references/onboard/visual-preservation.md) for detailed rules.

Key constraints enforced across ALL phases:
1. **Preserve tech stack** - Do NOT upgrade framework versions or add UI libraries (e.g., no shadcn/ui)
2. **Verbatim HTML extraction** - Components must render the exact same HTML/Tailwind classes as the original inline code
3. **Copy + Reorganize** - Pages migrate by copying existing code to new routes, not by rewriting
4. **CSS preservation** - All existing globals.css, Tailwind config, and custom classes must be kept as-is
5. **Navigation-only URL changes** - Route restructuring changes href values and file locations, not visual design

## Rules

See [shared/references/common/output-rules.md](../../shared/references/common/output-rules.md).
See [shared/references/common/context-rules.md](../../shared/references/common/context-rules.md).

## Execution Order

Run each phase strictly in order. Do NOT start the next phase until the current phase is complete and approved (where applicable).

| Phase | Agent | Input | Output | ★ |
|-------|-------|-------|--------|---|
| P1 | mockup-analyzer | `projectPath`, `devServerUrl` | `00-analysis/_index.md`, `00-analysis/navigation-structure.md`, `00-analysis/screens/*.md` | ★ |
| P2 | persona-architect | `00-analysis/_index.md`, `00-analysis/navigation-structure.md` | `01-personas/*.md` | ★ |
| P3 | critic-logic, critic-feasibility, critic-frontend, critic-analytics | `00-analysis/`, `01-personas/` | `02-critique/critique-*.md`, `02-critique/critique-synthesis.md` | - |
| P4 | critique-resolver | `02-critique/critique-synthesis.md` | `02-critique/critique-solve/*.md` | ★ |
| P5 | chapter-planner (restructure mode) | P1~P4 전체 output | `03-chapters/restructure-plan-final.md` | ★ |
| P6 | path-architect | `01-personas/*.md`, `03-chapters/restructure-plan-final.md`, `00-analysis/navigation-structure.md` | `04-restructure/_index.md`, `04-restructure/{persona}-paths.md` | ★ |
| P7 | component-auditor | `projectPath/components/` | `05-components/_index.md`, `05-components/inventory.md` | - |
| P8 | ui-pattern-detector | `projectPath/**/*.tsx`, `05-components/inventory.md` | `06-patterns/_index.md`, `06-patterns/existing-components.md`, `06-patterns/duplicates/*.md`, `06-patterns/extraction-plan.md` | ★ |
| P9 | component-builder | `06-patterns/duplicates/*.md` | `07-component-specs/{component}.md` | ★ |
| P10 | component-migrator | `05-components/`, `06-patterns/`, `07-component-specs/` | `08-migration/_index.md`, `08-migration/migration-plan.md` | ★ |
| P11 | context-synthesizer | P1~P10 전체 output | `spec-map.md` | - |
| P12 | critical-review | `spec-map.md` | `09-review/` (FAIL → P5 regression) | ★ |
| P13 | test-spec-writer | `01-personas/`, `spec-map.md` | `10-tests/` | ★ |
| P14 | api-predictor | `spec-map.md`, `projectPath` | `dev-plan/api-map.md` | - |
| P15 | spec-assembler + dev-planner | `spec-map.md`, `10-tests/`, all artifacts | `11-final/`, `dev-plan/development-map.md`, `dev-plan/{persona}/Phase-{n}/` | ★ |
| P16 | docs-hub-curator | `outputDir` | `README-DOC/index.md` | - |

---

## Phase 0: Init

### Step 0.1: Detect Project

```
IF project-path provided:
  projectPath = args[0]
ELSE:
  projectPath = pwd

Read: {projectPath}/package.json
Detect: Next.js version, App Router vs Pages
Glob: {projectPath}/app/**/page.tsx

IF not Next.js:
  Output: "Not a Next.js project"
  STOP
```

### Step 0.2: Session Init

```
docsDir = {pwd}/spec-it-onboard          # pwd = skill execution location (not the target project)

IF --session {sessionId} in args:
  # Reuse session created by router
  sessionDir = .spec-it/{sessionId}/plan
  Read: {sessionDir}/_meta.json
  # Update _meta.json with mode-specific values
  Update _meta.json:
    mode = "onboard"
    uiMode = "onboard"
    projectPath = {projectPath}
ELSE:
  # Direct invocation — create new session (pwd = skill execution location)
  result = Bash: session-init.sh "" onboard
  sessionId = extract SESSION_ID
  sessionDir = extract SESSION_DIR

# All output goes to {pwd}/spec-it-onboard/{sessionId}/
outputDir = {docsDir}/{sessionId}
```

> **IMPORTANT**: Output files MUST be created in `spec-it-onboard/` under the current working directory (`pwd`).
> Do NOT create files inside the target project (`projectPath`).

### Step 0.R: Resume

```
IF --resume in args:
  Read: .spec-it/{sessionId}/plan/_meta.json
  IF reexecuteFromP5: GOTO P5
  ELSE: GOTO _meta.currentStep
```

---

## Doc Index

| Doc | Description |
|-----|-------------|
| [docs/01-phases.md](docs/01-phases.md) | P1-P16 phase details + Final Approval |
| [docs/02-output-structure.md](docs/02-output-structure.md) | Output directory tree |

## Resume

`/spec-it-onboard --resume {sessionId}`
