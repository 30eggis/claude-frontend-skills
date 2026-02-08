# Phases (P1-P16) — onboard

## P1: Mockup Analysis

> **MANDATORY**: P1 MUST use Playwright MCP for all analysis.
> The agent MUST visit every screen in the browser and **recursively** click ALL interactive elements to FULL DEPTH.
> - Click every button, link, tab, dropdown, and role="button" element
> - Click elements that have onClick handlers or appear clickable in the snapshot
> - Click center of button-like visual elements even if not properly componentized as `<button>`
> - **RECURSIVE DEPTH**: When clicking an element reveals new content (sub-menu panels, nested tabs, new forms), the agent MUST explore that new content fully — click all its interactive elements, tabs, and sub-panels too
> - Example: On /management, clicking "조직 인사관리" shows a new panel → explore ALL tabs/buttons/forms inside that panel
> - Record all state changes: modals, dropdowns, navigation, toasts, expanded content
> - Track visited states to avoid infinite loops (max depth: 5)
> - Code-only analysis without browser interaction is NOT acceptable

```
Bash: status-update.sh {sessionDir} agent-start mockup-analyzer

Task(mockup-analyzer, sonnet):
  Input: projectPath, devServerUrl
  Tools: Playwright MCP (browser_navigate, browser_snapshot, browser_click, browser_hover, browser_wait_for, browser_take_screenshot, browser_evaluate, browser_press_key)
  REQUIRED: Use browser_snapshot on every screen, browser_click ALL interactive elements found in snapshot
  Output:
    - 00-analysis/_index.md
    - 00-analysis/navigation-structure.md
    - 00-analysis/screens/*.md (MUST include Interactive Exploration Results + Hidden UI Discovered sections)

Bash: status-update.sh {sessionDir} agent-complete mockup-analyzer "" 1.1
Bash: meta-checkpoint.sh {sessionDir} 1.1

AskUserQuestion: "Analysis complete. {screenCount} screens detected, {interactionCount} interactive elements explored. Review?"
Options: [Approve, Revise]
```

---

## P2: Persona Definition

```
Bash: status-update.sh {sessionDir} agent-start persona-architect

Task(persona-architect, sonnet):
  Input: 00-analysis/_index.md, 00-analysis/navigation-structure.md
  Output: 01-personas/*.md

Bash: status-update.sh {sessionDir} agent-complete persona-architect "" 2.1
Bash: meta-checkpoint.sh {sessionDir} 2.1

AskUserQuestion: "Personas defined. Review?"
Options: [Approve, Revise]
```

---

## P3: Multi-Critic (Parallel)

```
Bash: status-update.sh {sessionDir} agent-start "critic-logic,critic-feasibility,critic-frontend"

Task(critic-logic, sonnet, parallel):
  Input: 00-analysis/, 01-personas/
  Output: 02-critique/critique-logic.md

Task(critic-feasibility, sonnet, parallel):
  Input: 00-analysis/, 01-personas/
  Output: 02-critique/critique-feasibility.md

Task(critic-frontend, sonnet, parallel):
  Input: 00-analysis/, 01-personas/
  Output: 02-critique/critique-frontend.md

WAIT for all 3

Bash: status-update.sh {sessionDir} agent-complete "critic-logic,critic-feasibility,critic-frontend"
Bash: status-update.sh {sessionDir} agent-start critic-analytics

Task(critic-analytics, sonnet):
  Input: 02-critique/critique-logic.md, critique-feasibility.md, critique-frontend.md
  Output: 02-critique/critique-synthesis.md

Bash: status-update.sh {sessionDir} agent-complete critic-analytics "" 3.1
Bash: meta-checkpoint.sh {sessionDir} 3.1
```

---

## P4: Critique Resolution (User Input)

```
Read: 02-critique/critique-synthesis.md
Extract: must_resolve_count

IF must_resolve_count > 0:
  Bash: status-update.sh {sessionDir} agent-start critique-resolver

  Skill(critique-resolver):
    Input: critique-synthesis.md
    Output: 02-critique/critique-solve/*.md

  Bash: status-update.sh {sessionDir} agent-complete critique-resolver "" 4.1
ELSE:
  Write: 02-critique/critique-solve/merged-decisions.md (no issues)

Bash: meta-checkpoint.sh {sessionDir} 4.1

AskUserQuestion: "Critique resolution complete. Review?"
Options: [Approve, Revise]
```

---

## P5: Restructure Plan (onboard exclusive)

> complex의 chapter-planner와 달리, onboard에서는 **기존 코드의 리팩토링 구조를 확정**합니다.
> "뭘 만들지"가 아니라 "이미 있는 걸 어떻게 재구성할지" 결정합니다.
>
> **Visual Preservation**: 구조 변경은 파일 위치, import 경로, 라우트 구조에만 적용합니다.
> 페이지의 렌더링 결과(HTML/CSS)는 변경하지 않습니다.

```
Bash: status-update.sh {sessionDir} agent-start chapter-planner

Task(chapter-planner, opus):
  Input: 00-analysis/, 01-personas/, 02-critique/critique-synthesis.md, 02-critique/critique-solve/
  Context: This is an ONBOARD project - existing code restructuring, NOT new development.
           Focus on: what to keep, what to refactor, what to merge, what to split.
           CRITICAL: Visual Preservation applies. See shared/references/onboard/visual-preservation.md
           - Do NOT plan any visual/UI changes
           - Restructure = reorganize files and routes, NOT redesign pages
           - The existing tech stack (framework version, CSS approach) MUST be preserved
           - Plan should describe what code moves where, not what code gets rewritten
  Output: 03-chapters/restructure-plan-final.md

Bash: status-update.sh {sessionDir} agent-complete chapter-planner "" 5.1
Bash: meta-checkpoint.sh {sessionDir} 5.1

AskUserQuestion: "Restructure plan confirmed. {N} chapters. Proceed with path design?"
Options: [Proceed, Modify Plan, Review Details]
```

---

## P6: Path Restructure

```
Bash: status-update.sh {sessionDir} agent-start path-architect

Read: shared/references/onboard/path-patterns.md

Task(path-architect, sonnet):
  Input: 01-personas/*.md, 03-chapters/restructure-plan-final.md, 00-analysis/navigation-structure.md
  Output:
    - 04-restructure/_index.md
    - 04-restructure/{persona}-paths.md

Bash: status-update.sh {sessionDir} agent-complete path-architect "" 6.1
Bash: meta-checkpoint.sh {sessionDir} 6.1

AskUserQuestion: "Path restructure planned. Review?"
Options: [Approve, Revise]
```

---

## P7: Component Audit

```
Bash: status-update.sh {sessionDir} agent-start component-auditor

Task(component-auditor, sonnet):
  Input: projectPath/components/
  Output:
    - 05-components/_index.md
    - 05-components/inventory.md

Bash: status-update.sh {sessionDir} agent-complete component-auditor "" 7.1
Bash: meta-checkpoint.sh {sessionDir} 7.1
```

---

## P8: Pattern Detection

> **Visual Preservation**: 패턴 감지 시 원본 HTML/Tailwind 클래스를 정확히 기록해야 합니다.
> `extraction-plan.md`에는 원본 코드의 verbatim 복사본이 포함되어야 하며,
> 추출 후 컴포넌트가 원본과 동일한 HTML을 렌더링해야 한다는 제약조건을 명시해야 합니다.

```
Bash: status-update.sh {sessionDir} agent-start ui-pattern-detector

Read: shared/references/onboard/pattern-examples.md
Read: shared/references/onboard/visual-preservation.md

Task(ui-pattern-detector, sonnet):
  Input: projectPath/**/*.tsx, 05-components/inventory.md
  Context: CRITICAL - Visual Preservation Mode.
           - Record the EXACT original HTML/JSX for each detected pattern
           - Include verbatim code snippets in duplicates/*.md
           - extraction-plan.md MUST specify that extracted components render identical HTML
           - Do NOT propose HTML restructuring or class name changes
           - Props should only parameterize values that differ between occurrences
  Output:
    - 06-patterns/_index.md
    - 06-patterns/existing-components.md
    - 06-patterns/duplicates/*.md (MUST include verbatim original code)
    - 06-patterns/extraction-plan.md (MUST include visual preservation constraints)

Bash: status-update.sh {sessionDir} agent-complete ui-pattern-detector "" 8.1
Bash: meta-checkpoint.sh {sessionDir} 8.1

AskUserQuestion: "{patternCount} patterns detected. Review extraction plan?"
Options: [Approve, Revise]
```

---

## P9: Component Specs

> **Visual Preservation**: 컴포넌트 스펙은 기존 인라인 HTML을 그대로 추출한 것이어야 합니다.
> 새로운 디자인 토큰, 애니메이션, 또는 shadcn/ui 기반 컴포넌트를 설계하지 않습니다.
> Props는 원본 코드에서 인스턴스 간 다른 값만 파라미터화합니다.

```
Bash: status-update.sh {sessionDir} agent-start component-builder

Read: shared/references/onboard/visual-preservation.md

Task(component-builder, sonnet):
  Input: 06-patterns/duplicates/*.md
  Context: ONBOARD MODE - Visual Preservation applies.
           - Extract existing HTML verbatim into component files
           - Do NOT design new component APIs or add design tokens
           - Do NOT use shadcn/ui or any new UI library
           - Props should parameterize only values that differ between occurrences
           - The component MUST render the exact same HTML structure and Tailwind classes as the original
           - See shared/references/onboard/visual-preservation.md Rule 2
  Output: 07-component-specs/{component}.md

Bash: status-update.sh {sessionDir} agent-complete component-builder "" 9.1
Bash: meta-checkpoint.sh {sessionDir} 9.1

AskUserQuestion: "Component specs created. Review?"
Options: [Approve, Revise]
```

---

## P10: Migration Plan

> **Visual Preservation**: 마이그레이션 계획은 렌더링 결과가 원본과 pixel-identical 해야 한다는
> 제약조건을 포함해야 합니다. 컴포넌트 교체 시 원본 HTML 구조가 보존되어야 합니다.
> "구조 개선(structure improvements)"은 적용하지 않습니다.

```
Bash: status-update.sh {sessionDir} agent-start component-migrator

Read: shared/references/onboard/visual-preservation.md

Task(component-migrator, sonnet):
  Input: 05-components/, 06-patterns/, 07-component-specs/
  Context: ONBOARD MODE - Visual Preservation applies.
           - Migration = move files + update imports, NOT rewrite
           - Do NOT apply "structure improvements" (no forwardRef addition, no API redesign)
           - Each migration step must preserve identical rendered output
           - migration-plan.md MUST include pixel-identical constraint per migration item
           - Pages are migrated by copying existing code to new route, then replacing
             extracted inline patterns with the verbatim component (same HTML output)
           - See shared/references/onboard/visual-preservation.md Rule 3
  Output:
    - 08-migration/_index.md
    - 08-migration/migration-plan.md

Bash: status-update.sh {sessionDir} agent-complete component-migrator "" 10.1
Bash: meta-checkpoint.sh {sessionDir} 10.1

AskUserQuestion: "Migration plan ready. Review?"
Options: [Approve, Revise]
```

---

## P11: Context Synthesis

```
Bash: status-update.sh {sessionDir} agent-start context-synthesizer

Task(context-synthesizer, sonnet):
  Input: All artifacts from P1-P10
  Output: spec-map.md

Bash: status-update.sh {sessionDir} agent-complete context-synthesizer "" 11.1
Bash: meta-checkpoint.sh {sessionDir} 11.1
```

---

## P12: Critical Review (regression possible)

```
Skill(critical-review, --mode stepbystep):
  Input: spec-map.md
  Output: 09-review/

Read: 09-review/review-decisions.md
Check: Re-execution Trigger

IF reexecution_required:
  Output: "Returning to P5 for re-planning."
  _meta.reexecuteFromP5 = true
  GOTO P5

Bash: meta-checkpoint.sh {sessionDir} 12.1

AskUserQuestion: "Critical review complete. Proceed?"
Options: [Proceed, Review]
```

---

## P13: Test Specification

```
Bash: status-update.sh {sessionDir} agent-start test-spec-writer

Task(test-spec-writer, sonnet):
  Input: 01-personas/, spec-map.md
  Output: 10-tests/

Bash: status-update.sh {sessionDir} agent-complete test-spec-writer "" 13.1
Bash: meta-checkpoint.sh {sessionDir} 13.1

AskUserQuestion: "Test scenarios complete. Approve?"
Options: [Approve, Revise]
```

---

## P14: API Prediction

```
Bash: status-update.sh {sessionDir} agent-start api-predictor

Skill(api-predictor):
  Input: spec-map.md, projectPath (existing fetch/axios call analysis)
  Output: dev-plan/api-map.md

Bash: status-update.sh {sessionDir} agent-complete api-predictor "" 14.1
Bash: meta-checkpoint.sh {sessionDir} 14.1
```

---

## P15: Final Assembly + Dev Plan

> **Visual Preservation**: development-map.md는 기존 프로젝트와 동일한 기술 스택을 명시해야 합니다.
> "신규 빌드"가 아닌 "리팩토링" 접근 방식을 사용합니다.
> 각 Task는 "기존 코드를 복사 후 import만 변경"하는 방식으로 마이그레이션을 명세합니다.

```
Bash: status-update.sh {sessionDir} agent-start spec-assembler

Task(spec-assembler, haiku):
  Input: spec-map.md, 10-tests/
  Output: 11-final/

Read: shared/references/onboard/visual-preservation.md
Read: {projectPath}/package.json

Task(dev-planner, sonnet):
  Input: All artifacts
  Context: ONBOARD MODE - Visual Preservation applies.
           - development-map.md MUST specify the SAME tech stack as the original project
             (read package.json to detect exact versions)
           - Do NOT upgrade framework versions (e.g., Next.js 14 stays Next.js 14)
           - Do NOT add new UI libraries (no shadcn/ui, no Radix, no Headless UI)
           - Each task should describe a refactoring operation:
             "Copy existing page code to new route → extract inline patterns into
              verbatim components → update imports"
           - Phase-0 shared infrastructure = extracting existing inline patterns into
             component files (NOT designing new components)
           - See shared/references/onboard/visual-preservation.md
  Output:
    - dev-plan/development-map.md
    - dev-plan/{persona}/Phase-{n}/

Bash: status-update.sh {sessionDir} agent-complete spec-assembler "" 15.1
Bash: meta-checkpoint.sh {sessionDir} 15.1
```

---

## P16: Docs Hub

```
Bash: status-update.sh {sessionDir} agent-start docs-hub-curator

Task(docs-hub-curator, haiku):
  Input: outputDir = {outputDir}
  Output: README-DOC/index.md

Bash: status-update.sh {sessionDir} agent-complete docs-hub-curator "" 16.1
Bash: meta-checkpoint.sh {sessionDir} 16.1
```

---

## Final Approval

```
Read: 11-final/SPEC-SUMMARY.md

AskUserQuestion: "Spec complete. What next?"
Options: [
  {label: "Run /spec-it-execute", description: "Start implementation"},
  {label: "Archive", description: "Move to archive/"},
  {label: "Keep", description: "Keep in spec-it-onboard/"}
]

Bash: status-update.sh {sessionDir} complete
```
