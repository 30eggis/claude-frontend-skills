---
name: spec-it-burn
description: "Bottom-up, DEV.md-driven multi-agent execution engine. B0-B4 phases: scaffold, API, pages, component scale, documentation."
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Task, AskUserQuestion, mcp__plugin_spec-it_chrome-devtools__navigate_page, mcp__plugin_spec-it_chrome-devtools__take_snapshot, mcp__plugin_spec-it_chrome-devtools__take_screenshot, mcp__plugin_spec-it_chrome-devtools__wait_for, mcp__plugin_spec-it_chrome-devtools__click, mcp__plugin_spec-it_chrome-devtools__fill, mcp__plugin_spec-it_chrome-devtools__evaluate_script, mcp__plugin_spec-it_chrome-devtools__list_pages
argument-hint: "<elevate-output-dir> [--output <output-dir>] [--resume <sessionId>] [--mode <generative|original>]"
permissionMode: bypassPermissions
---

# spec-it-burn

Bottom-up, DEV.md-driven multi-agent execution engine. Replaces spec-it-fire with a page-centric approach.

## Quick Start

```
/spec-it:spec-it-burn /path/to/elevate-output
/spec-it:spec-it-burn /path/to/elevate-output --mode generative
/spec-it:spec-it-burn /path/to/elevate-output --output /path/to/project
/spec-it:spec-it-burn --resume <sessionId>
```

## Input

- spec-it-elevate output directory (contains `07-ia/`, `08-dev/`)
- Running dev server (will start if not running)

## Phase Overview

| Phase | Name | Purpose | Gate |
|-------|------|---------|------|
| B0 | Scaffold | IA-based folder creation + DEV.md placement | All route folders created |
| B1 | API Layer | Domain-based API clients | All API modules implemented |
| B2 | Pages | Bottom-up TDD page implementation | All pages pass tests + browser verify |
| B3 | Component Scale | Extract inline patterns to shared | All tests pass, 0 regressions |
| B4 | Documentation | AGENT.md, HISTORY.md, cleanup | All docs written, DEV.md deleted |

---

## B0: SCAFFOLD

### Step 1: UI Approach Selection

```
uiMode = args.mode or null

IF uiMode is null:
  AskUserQuestion(
    questions: [{
      question: "Which UI implementation approach?",
      header: "UI Approach",
      options: [
        {
          label: "Generative UI (Recommended)",
          description: "Reference wireframes as design guide, generate new production code with modern patterns."
        },
        {
          label: "Original UI Preservation",
          description: "Extract existing HTML verbatim, refactor to match spec. Pixel-identical fidelity."
        }
      ]
    }]
  )
```

### Step 2: Load Context + Create Session

```
elevateDir = args[0]
outputDir = args.output or Read({elevateDir}/_meta.json).projectPath
sessionId = args.resume or generateId()

Read: {elevateDir}/_meta.json
Read: {elevateDir}/07-ia/IA.md
Read: {elevateDir}/08-dev/dev-index.md

sessionDir = "tmp/burn-{sessionId}"
mkdir -p {sessionDir}

Write: {sessionDir}/_meta.json
{
  "sessionId": sessionId,
  "elevateDir": elevateDir,
  "outputDir": outputDir,
  "uiMode": uiMode,
  "currentPhase": "B0",
  "status": "in_progress"
}
```

### Step 3: Create Project Structure

```
Read: IA.md → parse navigation tree

Create scaffold folders (code-quality-principles §6):
  mkdir -p {outputDir}/src/types/
  mkdir -p {outputDir}/src/constants/
  mkdir -p {outputDir}/src/hooks/queries/
  mkdir -p {outputDir}/src/hooks/mutations/
  mkdir -p {outputDir}/src/components/ui/
  mkdir -p {outputDir}/src/components/composed/
  mkdir -p {outputDir}/src/components/layout/
  mkdir -p {outputDir}/src/lib/api/

FOR each route in IA.md:
  mkdir -p {outputDir}/app/{route-group}/{page}/
  Copy: DEV.md from {elevateDir}/08-dev/{route-group}/{page}/DEV.md
  Copy: WIREFRAME.md from {elevateDir}/08-dev/{route-group}/{page}/WIREFRAME.md
  Copy: TDD.md from {elevateDir}/08-dev/{route-group}/{page}/TDD.md
```

### Step 4: Dev Server Check

```
devServerUrl = "http://localhost:" + _meta.port
navigate_page(devServerUrl)
IF connection refused:
  Bash: cd {outputDir} && npm run dev &
  wait_for("page loaded", timeout: 15000)
```

**Gate**: All route folders exist, all DEV.md files placed.

---

## B1: API LAYER

### Step 1: Extract API Domains

```
Read: all DEV.md files → extract API Endpoints tables
Group endpoints by domain:
  /api/v1/attendance/* → attendance
  /api/v1/leave/*      → leave
  /api/v1/dashboard/*  → dashboard
  etc.
```

### Step 2: Implement API Clients

```
FOR each domain (parallel where independent):
  Task(agent: dev-executor, model: sonnet)
    Input:
      - Domain name
      - API endpoints (method, path, request/response shapes from DEV.md)
      - Canonical enum map
    Output:
      - {outputDir}/src/lib/api/{domain}.ts (API client methods)
      - {outputDir}/src/lib/api/types/{domain}.ts (TypeScript types)

After all domains:
  Write: {outputDir}/src/lib/api/client.ts (base fetch wrapper with auth, error handling)
  Write: {outputDir}/src/lib/api/index.ts (barrel export)

Generate types:
  Write: {outputDir}/src/types/api.ts (shared API types)
  Write: {outputDir}/src/types/enums.ts (from canonical-enum-map)
```

### Step 3: API Unit Tests

```
Bash: npx vitest run src/lib/api/ --reporter json
IF FAIL: fix → re-run (max 3)
```

**Gate**: All API modules compile, unit tests pass.

---

## B2: PAGES (Main Burn Loop)

### Depth Algorithm

```
1. Read all DEV.md files → extract Depth and Dependencies
2. Sort by depth DESC (deepest pages first)
3. Within same depth, group independent pages (no mutual dependencies)
4. Dispatch parallel agents per independent group

ORDER:
  Depth 3 → Depth 2 → Depth 1 → Depth 0
  Within same depth: pages with no cross-dependencies → parallel
```

### Page Implementation Loop

```
FOR each depth level (DESC):
  FOR each independent group at this depth (parallel):
    FOR each page in group:

      ┌─ STEP 1: PRE-TESTS ──────────────────────────┐
      │                                                │
      │ Task(agent: test-spec-generator, model: sonnet)│
      │   Input: DEV.md                                │
      │   Output: page.test.tsx (failing tests)        │
      │                                                │
      │ Skip if TDD.md already has generated tests     │
      └────────────────────────────────────────────────┘

      ┌─ STEP 2: IMPLEMENT ───────────────────────────┐
      │                                                │
      │ Task(agent: dev-executor, model: sonnet)       │
      │   Input: DEV.md + WIREFRAME.md (self-contained)│
      │                                                │
      │   IF uiMode == "generative":                   │
      │     - Reference WIREFRAME.md as design guide   │
      │     - Generate new production code             │
      │     - Modern patterns + design tokens          │
      │     - ARIA, loading/error/empty states         │
      │     - Permission guards                        │
      │                                                │
      │   IF uiMode == "original":                     │
      │     - Extract existing HTML verbatim           │
      │     - Refactor into components (same markup)   │
      │     - Add production layer on top              │
      │     - Preserve pixel-identical appearance      │
      │                                                │
      │   Both modes read:                             │
      │     - DEV.md (APIs, scenarios, validation)     │
      │     - WIREFRAME.md (UI spec or HTML ref)       │
      │     - src/lib/api/{domain}.ts (API client)     │
      │     - src/types/ (shared types)                │
      │                                                │
      │   Output: page.tsx (<200 lines)                │
      │   Code quality: code-quality-principles        │
      └────────────────────────────────────────────────┘

      ┌─ STEP 3: UNIT TESTS ──────────────────────────┐
      │                                                │
      │ Bash: npx vitest run {page.test.tsx}           │
      │                                                │
      │ IF FAIL:                                       │
      │   Read failing output                          │
      │   Fix implementation                           │
      │   Re-run (max 3 attempts)                      │
      │                                                │
      │ IF 3x exceeded:                                │
      │   Record as PARTIAL in burn report             │
      │   Continue to next page                        │
      └────────────────────────────────────────────────┘

      ┌─ STEP 4: BROWSER VERIFY ──────────────────────┐
      │                                                │
      │ Chrome DevTools MCP:                           │
      │                                                │
      │ 1. navigate_page(devServerUrl + route)         │
      │ 2. wait_for(key element from DEV.md)           │
      │ 3. take_snapshot() → a11y tree                 │
      │ 4. take_screenshot()                           │
      │    → {sessionDir}/screenshots/{route}/         │
      │                                                │
      │ 5. Verify a11y tree has expected elements      │
      │ 6. Click interactive elements from DEV.md      │
      │    → screenshot each interaction               │
      │                                                │
      │ IF visual mismatch or missing elements:        │
      │   → fix → re-verify (max 3)                    │
      └────────────────────────────────────────────────┘

      ┌─ STEP 5: MARK COMPLETE ───────────────────────┐
      │                                                │
      │ Append to HISTORY.md (in route folder):        │
      │   [{date}] Implemented: {summary}              │
      │                                                │
      │ Update burn report                             │
      └────────────────────────────────────────────────┘
```

### Integration Tests (after each depth level)

```
After all pages at depth N are implemented:
  Bash: npx vitest run --reporter json (integration tests)
  IF FAIL: identify failing pages → re-implement (max 2 regressions)
```

### User Checkpoint (after each depth level)

```
AskUserQuestion(
  questions: [{
    question: "Depth {N} pages complete ({count} pages). How to proceed?",
    header: "Checkpoint",
    options: [
      {label: "Continue to depth {N-1}", description: "Proceed with next depth level"},
      {label: "Review and fix issues", description: "Address issues before continuing"},
      {label: "Stop here", description: "Pause execution, resume later"}
    ]
  }]
)
```

**Gate**: All pages pass unit tests + browser verification.

---

## B3: COMPONENT SCALE-UP

```
Task(agent: component-scaler, model: sonnet)
  Input: {outputDir}/app/, {outputDir}/src/components/
  Process:
    1. Scan all page.tsx files for repeated patterns
    2. Extract patterns with 3+ occurrences to shared components
    3. Update all import paths
    4. Verify type check + all tests pass

  Output:
    - {outputDir}/src/components/{category}/{Component}.tsx
    - B3-extraction-report.md

Gate: npx tsc --noEmit clean, npx vitest run all pass, 0 regressions
```

**Gate**: All tests pass, no regressions.

---

## B4: DOCUMENTATION + CLEANUP

```
FOR each implemented page:

  Write: {route-folder}/AGENT.md
    Content:
      - Route, persona, depth, implementation date
      - Wireframe summary (key layout decisions)
      - Components used (with import paths)
      - API endpoints used
      - Key decisions made during implementation

  Append to: {route-folder}/HISTORY.md
    [{date}] B4 complete: AGENT.md created, dev artifacts cleaned

  Delete: {route-folder}/DEV.md
  Delete: {route-folder}/WIREFRAME.md
  Delete: {route-folder}/TDD.md

Write: {outputDir}/CLAUDE.md
  - Project overview
  - Architecture summary (folder structure)
  - Route map (from IA.md, simplified)
  - Component inventory (from B3 report)
  - API domain map (from B1)
  - Development conventions

Write: {outputDir}/HISTORY.md (project-level)
  - Full development log
  - Phase summaries
  - Files created/modified per phase
```

**Gate**: All AGENT.md files created, all DEV.md files deleted, CLAUDE.md written.

---

## Hard Gates + Auto Regression

```
Page-level (B2):
  Unit FAIL      → fix → re-test (max 3)
  Visual mismatch → fix → re-verify (max 3)
  3x exceeded    → PARTIAL, continue to next page

Depth-level (B2):
  Integration FAIL → identify pages → re-implement (max 2 regressions)

B3:
  Type error     → fix imports/props → re-check (max 3)
  Test regression → fix component → re-run (max 3)

Overall:
  Depth FAIL (3x regression) → User intervention
    AskUserQuestion: "Depth {N} failed 3 times. What to do?"
    Options: [Skip depth, Manual fix, Abort]
```

---

## Resume

`/spec-it:spec-it-burn --resume <sessionId>` → reads `tmp/burn-{sessionId}/_meta.json`, resumes from last completed phase/depth.

## Critical Rules

1. Ask UI approach before starting (generative vs original)
2. Bottom-up: deepest pages first (depth DESC)
3. DEV.md = sole input for each page implementation
4. Browser verification required for every page
5. TDD: tests before implementation
6. User checkpoint after every depth level
7. Max 3 retries/page, max 2 regressions/depth
8. 200-line file limit (code-quality-principles)
9. No prop drilling — URL = State, SearchParams for filter/sort/pagination
10. B4 cleans up dev artifacts (DEV.md, WIREFRAME.md, TDD.md)
