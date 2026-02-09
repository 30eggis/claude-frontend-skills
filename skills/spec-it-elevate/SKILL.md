---
name: spec-it-elevate
description: "HTML mockup to production-ready spec + IA.md + DEV.md per route. Phases P0-P8."
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Task, AskUserQuestion
argument-hint: "<project-path> [--port <port>] [--openapi <path>] [--resume <sessionId>]"
permissionMode: bypassPermissions
---

# spec-it-elevate

Transform a running Next.js mockup into production-ready specs, IA.md, and per-route DEV.md files for burn execution.

## Quick Start

```
/spec-it:spec-it-elevate /path/to/nextjs-project
/spec-it:spec-it-elevate /path/to/project --port 3001
/spec-it:spec-it-elevate /path/to/project --openapi /path/to/openapi.yaml
/spec-it:spec-it-elevate --resume <sessionId>
```

## Requirements

- Running Next.js dev server (required)
- OpenAPI spec (optional — used in P7 for API resolution)

## Phase Overview

| Phase | Name | Agents | Gate |
|-------|------|--------|------|
| P0 | Init | - | Session created |
| P1 | Explore | mockup-analyzer → persona-architect | All click-todo items clicked |
| P2 | Critique | 3 critics (parallel) → critic-analytics | All critical issues resolved |
| P3 | PRD Generate | prd-generator → prd-screen-writer | All screens have FR + spec |
| P4 | Extract | component-auditor → ui-pattern-detector → component-builder | All patterns cataloged |
| P5 | Analyze | gap-analyzer → taxonomy-normalizer | All components have states, enum map complete |
| P6 | Elevate | component-elevator | All components have Verbatim + Production |
| P7 | Review | context-synthesizer → critical-review → api-resolver | All issues resolved, API map done |
| P8 | Deliver | ia-generator → dev-md-generator → test-spec-generator | IA.md valid, all DEV.md self-contained, tests correct |

---

## P0: INIT

```
projectPath = args[0]
port = args.port or 3000
openApiPath = args.openapi or null
sessionId = args.resume or generateId()

sessionDir = "tmp/{sessionId}"
mkdir -p {sessionDir}

Write: {sessionDir}/_meta.json
{
  "sessionId": sessionId,
  "projectPath": projectPath,
  "port": port,
  "openApiPath": openApiPath,
  "framework": detectFramework(projectPath),
  "createdAt": now(),
  "currentPhase": "P0"
}
```

---

## P1: EXPLORE

Browser crawl + static code extraction + persona definition.

### Step 1: Mockup Analysis

```
Task(agent: mockup-analyzer, model: sonnet)
  Input: projectPath, port
  Output: {sessionDir}/00-exploration/
    - click-todo.yaml (all interactions mapped)
    - screenshots/ (per-page)
    - screens/*.md (per-screen analysis)
    - navigation-structure.md
    - route-map.md
```

### Step 2: Static Code Extraction

```
Glob: {projectPath}/src/components/**/*.{tsx,jsx,vue}
Extract: component-inventory.md, design-tokens.json
Write: {sessionDir}/00-exploration/component-inventory.md
Write: {sessionDir}/00-exploration/design-tokens.json
```

### Step 3: Persona Definition

```
Task(agent: persona-architect, model: sonnet)
  Input: navigation-structure.md, screens/*.md
  Output: {sessionDir}/00-exploration/personas/*.md
```

**Gate**: All click-todo items `clicked: true`, all screens documented.

---

## P2: CRITIQUE

3 critics debate in parallel + analytics synthesis.

### Step 1: Parallel Critics

```
Task(agent: critic-logic, model: sonnet)      ─┐
Task(agent: critic-feasibility, model: sonnet) ─┼─ parallel
Task(agent: critic-frontend, model: sonnet)    ─┘
  Input: 00-exploration/
  Output: critique results
```

### Step 2: Synthesis

```
Task(agent: critic-analytics, model: sonnet)
  Input: 3 critic outputs
  Output: {sessionDir}/01-critique/critique-synthesis.md
```

### Step 3: Resolution (if critical issues)

```
IF critique-synthesis has CRITICAL issues:
  Present to user via AskUserQuestion
  Write: {sessionDir}/01-critique/resolutions/*.md
```

**Gate**: All critical issues resolved.

---

## P3: PRD GENERATE

Generate production-quality PRD from exploration data.

### Step 1: Generate FR + Supporting Docs

```
Task(agent: prd-generator, model: opus)
  Input: 00-exploration/, 01-critique/
  Output: {sessionDir}/02-prd/
    - README.md, DOCUMENT-INDEX.md
    - 00-executive-summary/
    - 01-user-requirements/
    - 02-functional-requirements/FR-NNN-*.md
    - 04-business-rules/
    - 05-data-requirements/
    - 06-non-functional-requirements/
```

### Step 2: Generate Screen Specifications

```
Task(agent: prd-screen-writer, model: sonnet)
  Input: 00-exploration/, 02-prd/02-functional-requirements/
  Output: {sessionDir}/02-prd/03-screen-specifications/{portal}/*.md
```

**Gate**: Every screen has a corresponding FR + screen spec.

---

## P4: EXTRACT

Component catalog + pattern detection.

### Step 1: Component Audit

```
Task(agent: component-auditor, model: haiku)
  Input: projectPath
  Output: {sessionDir}/03-components/inventory.md
```

### Step 2: Pattern Detection

```
Task(agent: ui-pattern-detector, model: sonnet)
  Input: projectPath, inventory.md
  Output: {sessionDir}/03-components/patterns/extraction-plan.md
```

### Step 3: Component Specs

```
Task(agent: component-builder, model: sonnet, mode: elevate)
  Input: inventory.md, extraction-plan.md, projectPath
  Output: {sessionDir}/03-components/catalog/{category}/{component}.md
  Write: {sessionDir}/03-components/catalog/catalog-index.md
  Write: {sessionDir}/03-components/migration-plan.md
```

**Gate**: All repeated patterns (3+) cataloged, verbatim HTML preserved.

---

## P5: ANALYZE

Find what's MISSING for production readiness.

### Step 1: Gap Analysis

```
Task(agent: gap-analyzer, model: opus)
  Input: 03-components/, 02-prd/, 00-exploration/
  Output: {sessionDir}/04-analysis/
    - gap-report.md
    - permission-matrix.md
    - state-models/*.md
    - business-rule-overlay.md
    - enum-candidates.yaml
```

### Step 2: Enum Normalization

```
Task(agent: taxonomy-normalizer, model: sonnet)
  Input: enum-candidates.yaml, 03-components/, projectPath
  Output: {sessionDir}/04-analysis/canonical-enum-map.yaml
```

**Gate**: Every component has states identified, enum map complete.

---

## P6: ELEVATE

Layer production concerns on verbatim base.

```
Task(agent: component-elevator, model: sonnet)
  Input: 03-components/catalog/, 04-analysis/
  Output: {sessionDir}/05-elevated/
    - {category}/{component}.md (dual-structure specs)
    - elevation-summary.md
```

**Gate**: Every component has both Verbatim Base + Production Layer.

---

## P7: REVIEW

Context synthesis + critical review + API resolution.

### Step 1: Context Synthesis

```
Task(agent: context-synthesizer, model: sonnet)
  Input: all phase outputs
  Output: {sessionDir}/06-review/spec-map.md
```

### Step 2: Critical Review (3 parallel)

```
Skill(critical-review)
  Input: all phase outputs
  Output: {sessionDir}/06-review/critical-review/
    - scenario-review.md
    - ia-review.md
    - exception-review.md
```

### Step 3: Review Resolution

```
IF critical/high issues found:
  Present to user → resolve
  Write: {sessionDir}/06-review/review-resolutions.md

  IF regression needed:
    GOTO P3 (PRD issues) or P5 (analysis issues)
```

### Step 4: API Resolution

```
Task(agent: api-resolver, model: sonnet)
  Input: openApiPath (if provided), screen specs, components
  Output: {sessionDir}/06-review/api-map.md
```

**Gate**: All critical/high issues resolved, API map complete.

---

## P8: DELIVER

Generate IA.md (navigation architecture) + DEV.md per route + pre-implementation tests.

### Step 1: IA Generation

```
Task(agent: ia-generator, model: opus)
  Input: 00-exploration/, 02-prd/, 04-analysis/, 06-review/
  Output: {sessionDir}/07-ia/
    - IA.md (navigation tree, depths, dependencies, permissions)
```

### Step 2: DEV.md Generation

```
Task(agent: dev-md-generator, model: opus)
  Input: 07-ia/IA.md, 05-elevated/, 02-prd/, 04-analysis/, 06-review/
  Output: {sessionDir}/08-dev/
    - {route-group}/{page}/DEV.md (one per route, self-contained)
    - {route-group}/{page}/WIREFRAME.md (UI spec)
    - dev-index.md (master index)
```

### Step 3: Test Spec Generation

```
Task(agent: test-spec-generator, model: sonnet)
  Input: 08-dev/{route-group}/{page}/DEV.md
  Output: {sessionDir}/08-dev/{route-group}/{page}/
    - TDD.md (test plan with code templates)
    - page.test.tsx (pre-generated failing tests)
```

**Gate**: IA.md has valid DAG, all DEV.md files self-contained, all test files syntactically correct.

---

## Output Summary

```
tmp/{sessionId}/
├── _meta.json
├── 00-exploration/        (P1)
├── 01-critique/           (P2)
├── 02-prd/                (P3)
├── 03-components/         (P4)
├── 04-analysis/           (P5)
├── 05-elevated/           (P6)
├── 06-review/             (P7)
├── 07-ia/                 (P8 Step 1)
│   └── IA.md
└── 08-dev/                (P8 Step 2-3)
    ├── dev-index.md
    └── {route-group}/{page}/
        ├── DEV.md
        ├── WIREFRAME.md
        └── TDD.md
```

## Critical Rules

1. No phase skipping — execute P0 through P8 in order
2. Gates must pass before advancing to next phase
3. Main orchestrator must not write files via Bash redirection (use Write tool)
4. All agent outputs go to `tmp/{sessionId}/` directory
5. Resume supported: `--resume <sessionId>` restarts from last incomplete phase
6. Regression: P7 can regress to P3 or P5 if critical issues found
7. DEV.md must be self-contained — developer needs no other files to implement
