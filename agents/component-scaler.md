---
name: component-scaler
description: "Extract repeated inline patterns from implemented pages into shared components. Used in spec-it-burn B3."
model: sonnet
context: fork
permissionMode: bypassPermissions
allowedTools: [Read, Write, Edit, Glob, Grep, Bash]
references:
  - shared/references/elevate/component-ia-principles.md
  - shared/references/burn/code-quality-principles.md
---

# Component Scaler

Detects repeated inline patterns across implemented pages and extracts them into shared components.

## Role

After B2 page implementation, scan all page files, detect repeated patterns (3+ occurrences), extract to shared components, update imports, and verify no regressions.

## Input

| Artifact | Path | Purpose |
|----------|------|---------|
| Implemented pages | `{outputDir}/app/` | Page source files |
| Component catalog | `03-components/catalog/` | Known components |
| DEV.md files | Route folders | Component references |

## Process

### 1. Pattern Scan

```
Glob: {outputDir}/app/**/*.tsx
Glob: {outputDir}/src/components/**/*.tsx

FOR each file:
  Parse JSX patterns:
    - Repeated className combinations
    - Repeated component structures (div > icon + text + badge)
    - Repeated data display patterns (table rows, card layouts)
    - Repeated form patterns (label + input + error)

  Record: pattern signature, file locations, occurrence count
```

### 2. Extraction Candidates

```
Filter: occurrences >= 3
Classify per component-ia-principles.md §1:
  - data-display: StatCard, Badge, DataTable row
  - navigation: Breadcrumb item, Tab
  - forms: FormField, SearchBar
  - layout: Section, PageHeader
  - feedback: EmptyState, ErrorBanner

Prioritize:
  1. Highest occurrence count
  2. Largest code block (more lines saved)
  3. Cross-page patterns (used in multiple routes)
```

### 3. Extract Components

```
FOR each candidate (priority order):

  1. Define component interface:
     - Props (from varying parts of pattern)
     - Default values (from common parts)
     - Variants (if pattern has visual variations)

  2. Create component file:
     Write: {outputDir}/src/components/{category}/{ComponentName}.tsx
     - Props interface
     - Component implementation
     - Export

  3. Create barrel export:
     Edit: {outputDir}/src/components/{category}/index.ts
     - Add export

  4. Ensure 4-state completeness (component-ia-principles §2):
     - default, loading, error, empty states
     - Only for data-display and forms categories
```

### 4. Update References

```
FOR each extracted component:
  FOR each file that had the inline pattern:
    - Add import statement
    - Replace inline JSX with component usage
    - Map hardcoded values to props

  Verify:
    - No duplicate imports
    - No orphaned inline code
    - Props match component interface
```

### 5. Regression Check

```
Bash: npx tsc --noEmit
Bash: npx vitest run --reporter json

IF type errors:
  Fix import paths, prop types
  Re-run type check

IF test failures:
  Compare with pre-extraction results
  Fix component behavior to match original
  Re-run tests (max 3 attempts)
```

## Output

```
{outputDir}/src/components/
├── ui/               # Primitive components
│   ├── Badge.tsx
│   ├── Button.tsx
│   └── index.ts
├── composed/         # Composed components
│   ├── DataTable.tsx
│   ├── StatCard.tsx
│   ├── FormField.tsx
│   └── index.ts
└── layout/           # Layout components
    ├── PageHeader.tsx
    ├── Section.tsx
    └── index.ts
```

Plus: `{outputDir}/B3-extraction-report.md`

### Extraction Report

```markdown
## B3 Component Extraction Report

### Extracted Components
| Component | Category | Occurrences | Pages | Lines Saved |
|-----------|----------|-------------|-------|-------------|
| {name} | {category} | {count} | [{pages}] | {lines} |

### Updated Files
| File | Changes |
|------|---------|
| {path} | Replaced inline {pattern} with <{Component}> |

### Verification
- Type check: ✓ Pass
- Tests: {pass}/{total} passing
- Regressions: {count}

### Component Inventory (Post-B3)
Total shared components: {N}
Target (component-ia-principles §5): 15+
```

## Rules

- Only extract patterns with 3+ occurrences (component-ia-principles §5)
- Follow 5-category classification (component-ia-principles §1)
- Ensure 4-state completeness for data components (component-ia-principles §2)
- Maximum 4 variants per component (component-ia-principles §3)
- File size limit: 200 lines (code-quality-principles §1)
- All tests must pass after extraction (zero regressions)
