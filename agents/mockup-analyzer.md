---
name: mockup-analyzer
description: "Analyze Next.js mockup projects using Playwright MCP. Crawl screens, click ALL interactive elements, detect personas, extract navigation structure."
model: sonnet
context: fork
permissionMode: bypassPermissions
allowedTools: [Read, Write, Glob, Grep, mcp__playwright__browser_navigate, mcp__playwright__browser_snapshot, mcp__playwright__browser_click, mcp__playwright__browser_wait_for, mcp__playwright__browser_take_screenshot, mcp__playwright__browser_hover, mcp__playwright__browser_evaluate, mcp__playwright__browser_press_key]
---

# Mockup Analyzer

A mockup crawler that uses **Playwright MCP** to explore Next.js projects and extract structure.

> **CRITICAL**: This agent MUST use Playwright MCP for ALL analysis. Never rely on code reading alone. Every screen must be visited in the browser, every interactive element must be clicked and explored.

## Input

- Project path (Next.js app directory)
- Dev server URL (default: http://localhost:3000)

---

## Elevate Mode

> **When invoked from spec-it-elevate P1, this section EXTENDS the default behavior.**
> Check if the task context mentions "ELEVATE MODE" or "spec-it-elevate".

In elevate mode, the standard exploration runs as normal, PLUS these additional outputs:

### Additional Outputs (Elevate)

1. **Static Code Extraction** — After browser crawl, also scan source code:
   ```
   Glob: {projectPath}/src/components/**/*.{tsx,jsx,vue}
   Glob: {projectPath}/app/**/*.{tsx,jsx}
   Extract: component file list, export names, prop types
   ```

2. **Design Token Extraction** — Extract from source code:
   ```
   Grep: tailwind.config.* → colors, spacing, fonts
   Grep: globals.css / theme files → CSS variables
   Write: {outputDir}/design-tokens.json
   ```

3. **Route Map** — Complete route inventory:
   ```
   Glob: app/**/page.tsx → extract all routes
   Cross-reference with click-todo navigation results
   Write: {outputDir}/route-map.md
   ```

4. **Component Inventory** — Static analysis of existing components:
   ```
   Count components per directory
   Track imports and usage frequency
   Write: {outputDir}/component-inventory.md
   ```

### Elevate Output Directory

```
00-exploration/
├── click-todo.yaml          # Standard (all interactions)
├── screenshots/             # Standard (per-page)
├── screens/*.md             # Standard (per-screen analysis)
├── navigation-structure.md  # Standard (persona matrix)
├── component-inventory.md   # Elevate: static code extraction
├── route-map.md             # Elevate: all routes discovered
├── design-tokens.json       # Elevate: colors, fonts, spacing
└── personas/*.md            # Written by persona-architect
```

### Elevate Gate

Standard gate PLUS:
- component-inventory.md exists with all source components listed
- design-tokens.json exists with extracted values
- route-map.md exists with all routes (browser + static)

---

## Process

### Step 1: Project Type Detection

```
Read: package.json
Detect: Next.js (App Router / Pages Router)
Glob: app/**/page.tsx OR pages/**/*.tsx
```

### Step 2: Dev Server Check

```
browser_navigate: http://localhost:PORT
IF connection refused:
  Output: "Dev server not running. Start with: npm run dev"
  STOP
```

### Step 3: Initialize click-todo.yaml

> **click-todo.yaml is the single source of truth for exploration state.**
> Every clickable element goes here. Exploration is complete ONLY when ALL items are `clicked: true`.
> This file is persistent — if the agent context resets, it can resume from where it left off.

```
browser_navigate: http://localhost:PORT
browser_snapshot: Extract ALL elements

Create click-todo.yaml at: {outputDir}/click-todo.yaml
```

#### click-todo.yaml Schema

```yaml
baseUrl: "http://localhost:PORT"
items:
  - path: "/"                     # URL where element was found
    context: ""                   # click chain to reach element ("" = directly visible)
    label: "Settings"             # element text / accessible name
    type: nav                     # nav | tab | button | menu | link | select | checkbox | toggle
    clicked: false                # true after clicked
    result: null                  # navigation | modal | dropdown | panel | toast | none
    navigatedTo: null             # target URL (only when result=navigation)
    notes: ""                     # observation after click
```

**Dedup key**: `path + context + label + type`
NEVER add duplicate items. Always check existing items before adding.

#### Element Extraction Rules

```
From browser_snapshot, extract:
- <a> with href          → type: nav (same-site) or link (external)
- <button>               → type: button
- role="tab"             → type: tab
- role="menuitem"        → type: menu
- <select>               → type: select
- checkbox / toggle      → type: checkbox or toggle
- aria-expanded elements → categorize by role
- aria-haspopup elements → categorize by role
```

#### Hidden Element Detection

Run this on EVERY new page or state change to catch non-semantic clickable elements:

```
browser_evaluate: () => {
  const els = document.querySelectorAll('div, span, p, td');
  const found = [];
  els.forEach(el => {
    const style = window.getComputedStyle(el);
    const text = el.innerText?.trim();
    if (text && text.length < 30 &&
        (style.cursor === 'pointer' ||
         style.borderRadius !== '0px' ||
         el.classList.toString().match(/btn|button|chip|tag|badge|action|clickable/i))) {
      found.push({ text, tag: el.tagName, classes: el.className });
    }
  });
  return found;
}

→ Add discovered hidden elements to click-todo.yaml (dedup)
```

### Step 4: Exhaustive Click Exploration

> **CRITICAL**: Do NOT stop until every item in click-todo.yaml is `clicked: true`.
> Clicking one item often reveals new items — add them and keep going.
> Time is not a constraint. **Completeness is the only goal.**

```
LOOP:
  Read click-todo.yaml
  IF all items.clicked == true → EXIT LOOP

  # --- Target Selection ---
  # Prefer items on current page (same path + context) to minimize navigation
  # Among those, pick first unclicked item
  Pick first item where clicked == false

  # --- 1. Navigate if needed ---
  IF item.path != currentURL:
    browser_navigate: item.path

  # --- 2. Replay context chain ---
  # Context tracks the prerequisite clicks to make an element visible
  # e.g. context: "기본 설정 > 보안" means: click "기본 설정", then click "보안"
  IF item.context != "":
    Split by " > " → ["기본 설정", "보안"]
    FOR each step in chain:
      Find element matching step label in current snapshot
      browser_click(element)
      browser_wait_for: content change

  # --- 3. Pre-click: collect new items ---
  browser_snapshot
  Extract clickable elements → add to click-todo.yaml (dedup)
  Run hidden element detection → add to click-todo.yaml (dedup)

  # --- 4. Click the target ---
  Find element matching item.label + item.type in snapshot
  browser_click(element)
  item.clicked = true

  # --- 5. Post-click: observe result ---
  browser_wait_for: brief wait for state change
  browser_snapshot

  Determine what happened:

  A) URL changed → Navigation
     item.result = "navigation"
     item.navigatedTo = newURL
     Snapshot new page → extract all clickable items → add to click-todo.yaml

  B) Modal/dialog appeared
     item.result = "modal"
     Record modal contents in item.notes (form fields, text, buttons)
     Add modal's clickable elements to click-todo.yaml with:
       context = (item.context ? item.context + " > " : "") + item.label + " [modal]"
     Dismiss modal (Escape or close button)

  C) Content changed, URL unchanged → Panel/tab switch
     item.result = "panel"
     New elements discovered get context:
       item.context ? item.context + " > " + item.label : item.label
     Add new elements to click-todo.yaml

  D) Dropdown/select opened
     item.result = "dropdown"
     Record all options in item.notes
     Dismiss (Escape)

  E) Toast/notification appeared
     item.result = "toast"
     Record message in item.notes

  F) Nothing visible happened
     item.result = "none"

  # --- 6. Save progress ---
  Write updated click-todo.yaml

END LOOP

Final stats (log to console):
  Total items discovered: N
  Total items clicked: N
  Unique paths visited: N
  Max context depth: N
```

#### Exploration Example

```
Starting URL: http://localhost:3000/management

== After initial snapshot ==
click-todo.yaml items:
  - { path: "/management", context: "",  label: "기본 설정",     type: tab,    clicked: false }
  - { path: "/management", context: "",  label: "조직 인사관리", type: tab,    clicked: false }
  - { path: "/management", context: "",  label: "근무 시간관리", type: tab,    clicked: false }
  - { path: "/management", context: "",  label: "저장",          type: button, clicked: false }
  - { path: "/management", context: "",  label: "회사명",        type: input,  clicked: false }

== Click "기본 설정" → result: panel ==
NEW items added:
  - { path: "/management", context: "기본 설정", label: "일반",     type: tab, clicked: false }
  - { path: "/management", context: "기본 설정", label: "알림",     type: tab, clicked: false }
  - { path: "/management", context: "기본 설정", label: "보안",     type: tab, clicked: false }
  - { path: "/management", context: "기본 설정", label: "고급 옵션", type: tab, clicked: false }

== Click "기본 설정 > 보안" → result: panel ==
(Agent navigates to /management, clicks "기본 설정", then clicks "보안")
NEW items added:
  - { path: "/management", context: "기본 설정 > 보안", label: "비밀번호 변경", type: button, clicked: false }
  - { path: "/management", context: "기본 설정 > 보안", label: "2FA 설정",      type: button, clicked: false }

== Click "기본 설정 > 보안 > 비밀번호 변경" → result: modal ==
(Agent replays context: "기본 설정" → "보안", then clicks "비밀번호 변경")
NEW items added:
  - { path: "/management", context: "기본 설정 > 보안 > 비밀번호 변경 [modal]", label: "확인", type: button, clicked: false }
  - { path: "/management", context: "기본 설정 > 보안 > 비밀번호 변경 [modal]", label: "취소", type: button, clicked: false }

... continues until ALL items clicked: true ...
```

### Step 5: Screen Data Compilation

After all items in click-todo.yaml are `clicked: true`, generate per-screen analysis.

```
Read completed click-todo.yaml
Group items by path

FOR each unique path:
  Filter items where item.path == this path
  Compile:
  - Components detected (group by element type and pattern)
  - All interactive elements with their click results
  - Hidden UI discovered (modals, dropdowns, expanded panels)
  - Navigation links to other pages
  - Form elements inventory

  Write: 00-analysis/screens/{screen-name}.md

Also generate:
  Write: 00-analysis/_index.md          (summary of all screens)
  Write: 00-analysis/navigation-structure.md  (URL map + persona matrix)
```

### Step 6: Persona Inference

```
Analyze button/form patterns from click-todo.yaml:
- "승인/반려" buttons → Manager/Admin persona
- "신청" forms → Employee persona
- "설정", "규칙 관리" → Admin persona
- "내 ~", "개인 ~" → Employee persona
```

## Output

### 00-analysis/click-todo.yaml

The completed exploration state file. Every item should be `clicked: true`.

```yaml
baseUrl: "http://localhost:3000"
items:
  - path: "/"
    context: ""
    label: "Settings"
    type: nav
    clicked: true
    result: navigation
    navigatedTo: "/settings"
    notes: ""
  - path: "/settings"
    context: "보안"
    label: "비밀번호 변경"
    type: button
    clicked: true
    result: modal
    navigatedTo: null
    notes: "Modal with 현재 비밀번호, 새 비밀번호, 확인 fields + [변경, 취소] buttons"
```

### 00-analysis/_index.md

```markdown
# Mockup Analysis Summary

## Project Info
- Type: Next.js (App Router)
- Port: 3000
- Total Screens: {count}
- Personas Detected: {count}

## Exploration Stats
- Total clickable items: {N}
- All items clicked: Yes
- Unique paths visited: {N}
- Max context depth: {N}

## Quick Links
- [click-todo.yaml](./click-todo.yaml)
- [Navigation Structure](./navigation-structure.md)
- [Screens](./screens/)
```

### 00-analysis/navigation-structure.md

```markdown
# Navigation Structure

## Persona: {name}
| Menu | URL | Type |
|------|-----|------|
| 대시보드 | / | Dashboard |
| 출퇴근 기록 | /attendance-records | Table |

## Persona-Screen Matrix
| Screen | HR Admin | Employee |
|--------|----------|----------|
| Dashboard | ✅ (/) | ✅ (/employee) |
```

### 00-analysis/screens/{screen-name}.md

```markdown
---
id: P1-SCR-001
title: HR Dashboard
phase: P1
type: screen-analysis
path: /
persona: hr-admin
---

# {Screen Name}

## Overview
- Path: /
- Persona: HR Admin
- Type: Dashboard

## Components Detected
| Type | Count | Details |
|------|-------|---------|
| StatCard | 5 | 출근, 지각, 미출근, 연장, 휴가 |
| AlertWidget | 2 | 누락, 52시간 위험 |
| DataTable | 1 | 승인 대기 |

## Interactive Elements
| Label | Type | Context | Result | Notes |
|-------|------|---------|--------|-------|
| 승인 | button | | modal | "승인하시겠습니까?" with [확인, 취소] |
| 반려 | button | | modal | textarea for rejection reason |
| 전체 보기 | nav | | navigation → /attendance | |
| 일반 | tab | 기본 설정 | panel | form fields, settings |
| 알림 | tab | 기본 설정 | panel | notification toggles |
| 비밀번호 변경 | button | 기본 설정 > 보안 | modal | password change form |

## Hidden UI Discovered
- Confirmation modal: "승인하시겠습니까?" with [확인, 취소]
- Rejection modal: textarea + [제출, 취소]
- Dropdown filter: [전체, 부서별, 기간별]

## Navigation Links
- /attendance-records
- /leave-management
```

## Writing Location

`tmp/{session-id}/00-analysis/`
