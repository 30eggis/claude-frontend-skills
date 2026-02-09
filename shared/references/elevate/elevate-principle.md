# Elevate Principle: Verbatim Base + Production Layer

The core principle of the elevate workflow: **preserve what exists, then layer production concerns on top.**

---

## Philosophy

Mockup HTML is the single source of visual truth. Never redesign. Instead:

1. **Verbatim Base** — Extract and preserve the exact HTML/JSX/Tailwind from the mockup
2. **Production Layer** — Add missing production concerns as a separate, clearly marked layer

This dual-structure ensures pixel-identical fidelity to the mockup while achieving production readiness.

---

## Verbatim Base Rules

1. **Copy, don't redesign** — The component's markup must be a verbatim copy of the original
2. **Same HTML elements** — `<div>` stays `<div>`, do not replace with `<Card>` or semantic wrappers
3. **Same Tailwind classes** — Keep all original utility classes exactly as-is
4. **No new UI libraries** — Do not reference libraries not in the original code
5. **Props = only variable values** — Only parameterize values that differ between occurrences

---

## Production Layer Additions

The Production Layer adds ONLY what is missing for production readiness:

| Category | What to Add | Example |
|----------|------------|---------|
| **Loading State** | Skeleton matching verbatim layout | `<Skeleton className="h-4 w-24" />` |
| **Error State** | Error message + retry in same container | `<ErrorBanner onRetry={refetch} />` |
| **Empty State** | Contextual empty message | `"No records found for this period"` |
| **ARIA** | Accessibility attributes | `aria-label`, `role`, `aria-live` |
| **Design Tokens** | Map hardcoded values to tokens | `text-blue-600` → `text-primary` |
| **Enum References** | Link to canonical-enum-map.yaml | `status: _ref:enums.attendanceStatus` |
| **Permission Guards** | Role-based visibility | `{can('view:dashboard') && <Component />}` |
| **Keyboard Navigation** | Focus management, shortcuts | `onKeyDown`, `tabIndex` |

---

## Dual-Structure Format

Every elevated component spec has two clearly separated sections:

```markdown
# Component: {Name}

## Section 1: Verbatim Base
> Source: {file}:{line_range}
> This section is FROZEN. Do not modify the HTML structure.

### Original HTML
{exact copy of mockup HTML/JSX with Tailwind classes}

### Props Interface
{only values that vary between occurrences}

## Section 2: Production Layer
> This section ADDS to the verbatim base. Never replaces.

### Loading State
### Error State
### Empty State
### Accessibility (ARIA)
### Design Token Bindings
### Enum References
### Permission Guards
```

---

## Anti-Patterns

| Do NOT | Instead |
|--------|---------|
| Replace `<div>` with `<Card>` | Keep `<div>` with same classes |
| Add animations not in mockup | Only add loading/error states |
| Introduce new spacing/colors | Map existing values to tokens |
| Add features not in the mockup | Only add production infrastructure |
| Merge verbatim + production into one block | Keep them as separate sections |

---

## When to Deviate

Deviation from verbatim is allowed ONLY for:

1. **Accessibility compliance** — WCAG 2.1 AA requirements
2. **Security** — XSS prevention, input sanitization
3. **Performance** — Virtualization for large lists (>100 items)

Each deviation must be documented with rationale in the Production Layer section.
