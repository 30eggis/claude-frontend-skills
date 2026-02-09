---
name: test-spec-generator
description: "Generate TDD.md + pre-implementation test files from DEV.md. Used in spec-it-elevate P8 and spec-it-burn B2."
model: sonnet
context: fork
permissionMode: bypassPermissions
allowedTools: [Read, Write, Glob]
references:
  - shared/references/burn/tdd-principles.md
  - shared/references/burn/scenario-principles.md
  - shared/references/elevate/dev-md-schema.md
---

# Test Spec Generator

Generates pre-implementation test files (TDD) from DEV.md per page/route.

## Role

Create test files that define expected behavior BEFORE any code is written. Tests should fail initially and pass after implementation.

## Input

| Artifact | Path | Purpose |
|----------|------|---------|
| DEV.md | `{route-folder}/DEV.md` | Test targets (scenarios, API, validation) |
| WIREFRAME.md | `{route-folder}/WIREFRAME.md` | UI structure reference |
| Canonical enum map | `04-analysis/canonical-enum-map.yaml` | Valid values |

## Process

### 1. Parse DEV.md

```
Read DEV.md → extract:
  - Route, Persona, Depth
  - API Endpoints table
  - API-UI Binding
  - Scenarios table
  - UI/UX State Changes table
  - Validation rules
  - TDD Plan checklist
```

### 2. Generate TDD.md (Detailed Test Plan)

```
Write: {route-folder}/TDD.md

Structure:
  # TDD: {Page Name}
  > Route: {route}
  > Depth: {N}

  ## Unit Tests
  FOR each component in WIREFRAME.md:
    - renders without error
    - renders with required props
    - loading state renders skeleton
    - error state renders error message
    - empty state renders empty message
    - ARIA attributes present
    - enum values display correctly

  ## Integration Tests
  FOR each API-UI Binding:
    - API called with correct params on trigger
    - Response rendered correctly
    - Error response handled
    - Loading state shown during fetch

  ## E2E Tests (Depth-Based)
  Depth 3+: Scaffolded (page loads + key element)
  Depth 1-2: Incremental (step-by-step assertions)
  Depth 0: Full (complete workflows)
```

### 3. Generate page.test.tsx

```
Write: {route-folder}/page.test.tsx
```

### Unit Test Section

```typescript
import { render, screen } from '@testing-library/react';
import { describe, it, expect, vi } from 'vitest';

// Generated from DEV.md Scenarios + UI/UX State Changes

describe('{PageName}', () => {
  // Happy path (from Scenarios #1)
  it('renders initial state with expected elements', () => {
    render(<{PageName} />);
    expect(screen.getByTestId('{keyElement}')).toBeInTheDocument();
  });

  // Loading state (from UI/UX State Changes)
  it('shows loading skeleton on initial load', () => {
    // Mock API to delay
    render(<{PageName} />);
    expect(screen.getByTestId('{keyElement}-skeleton')).toBeInTheDocument();
  });

  // Error state (from UI/UX State Changes)
  it('shows error banner on API failure', async () => {
    // Mock API to reject
    render(<{PageName} />);
    await screen.findByText(/error/i);
  });

  // Empty state (from UI/UX State Changes)
  it('shows empty message when no data', async () => {
    // Mock API to return empty
    render(<{PageName} />);
    await screen.findByText(/no .* found/i);
  });

  // Validation (from Validation rules)
  it('validates {rule-name}: {condition}', async () => {
    render(<{PageName} />);
    // Trigger validation
    // Expect {effect}
  });
});
```

### Integration Test Section

```typescript
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

describe('{PageName} Integration', () => {
  // From API-UI Binding: on-loaded => GET {endpoint}
  it('loads data from API on mount', async () => {
    render(<{PageName} />);
    await waitFor(() => {
      expect(screen.getByTestId('{keyElement}')).toBeInTheDocument();
    });
  });

  // From API-UI Binding: click-to-{btn} => POST {endpoint}
  it('{action} triggers API call and updates UI', async () => {
    const user = userEvent.setup();
    render(<{PageName} />);
    await user.click(screen.getByRole('button', { name: '{btn-label}' }));
    // Verify API called + UI updated
  });
});
```

### E2E Test Section (Depth-Based)

**Depth 3+ (Scaffolded):**
```typescript
import { test, expect } from '@playwright/test';

test('{pageName} loads correctly', async ({ page }) => {
  await page.goto('{route}');
  await expect(page.locator('{keyElement}')).toBeVisible();
});
```

**Depth 1-2 (Incremental):**
```typescript
test('{pageName} interactive flow', async ({ page }) => {
  await page.goto('{route}');
  await expect(page.locator('{keyElement}')).toBeVisible();
  // Step-by-step from Scenarios
  await page.click('{trigger}');
  await expect(page.locator('{result}')).toBeVisible();
});
```

**Depth 0 (Full):**
```typescript
test('{workflow} complete flow', async ({ page }) => {
  await page.goto('{route}');
  // Full scenario from DEV.md
  // Navigate, fill forms, submit, verify results
});
```

## Scenario → Test Mapping

Follow scenario-principles.md §5:

| Scenario Type | Test Level | Priority |
|--------------|-----------|----------|
| Happy Path | Unit + E2E | P0 (always generate) |
| Validation | Unit | P0 (always generate) |
| Error | Integration | P1 |
| Empty | Unit | P1 |
| Permission | Integration | P1 |
| Edge | E2E | P2 |

## Output

```
{route-folder}/
├── TDD.md              # Detailed test plan
└── page.test.tsx        # Pre-generated failing tests
```

## Rules

- Tests must be syntactically valid TypeScript
- Tests must fail before implementation (TDD)
- Use data-testid selectors where possible
- Import paths use `@/` alias convention
- Progressive E2E depth based on route Depth (not sprint number)
- Every scenario in DEV.md = at least one test assertion
- Every validation rule = at least one test case
- Every UI state (loading, error, empty) = at least one test

## Writing Location

Same folder as DEV.md: `{route-folder}/`
