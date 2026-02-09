# Elevated Component Spec Template

Dual-structure component specification: Verbatim Base (frozen) + Production Layer (additions).

---

## Template

```markdown
# Component: {ComponentName}

> Category: {data-display | navigation | forms | layout | feedback}
> Source: {file_path}:{line_range}
> Occurrences: {N} times across {M} files
> Elevated: {date}

---

## Section 1: Verbatim Base

> **FROZEN** — Do not modify the HTML structure below.
> This is the pixel-identical source of truth from the mockup.

### Original HTML

```{tsx|html}
{Exact copy of original inline HTML/JSX with all Tailwind classes preserved.
No additions, no removals, no replacements.}
```

### Props Interface

```typescript
interface {ComponentName}Props {
  // Only values that differ between occurrences
  {propName}: {type};         // {description}
  {propName}?: {type};        // default: {defaultValue}
}
```

### Usage Contexts

| Location | Variation | Props Used |
|----------|-----------|-----------|
| {file}:{line} | {what differs} | {prop values} |

---

## Section 2: Production Layer

> **ADDITIONS ONLY** — These items layer on top of the Verbatim Base.
> Never replace verbatim markup. Wrap or extend it.

### Loading State

```{tsx|html}
{Skeleton or loading indicator that matches the verbatim layout shape.
Must use same dimensions and spacing as the original.}
```

### Error State

```{tsx|html}
{Error display within the same container.
Include retry mechanism.}
```

### Empty State

```{tsx|html}
{Empty data message within the same container.
Contextual to what data is missing.}
```

### Accessibility (ARIA)

| Element | Attribute | Value |
|---------|-----------|-------|
| {element} | role | {role} |
| {element} | aria-label | {label} |
| {element} | aria-live | polite | assertive |

### Design Token Bindings

| Original Class | Token Binding | Notes |
|---------------|---------------|-------|
| {tailwind-class} | {token-name} | {why mapped} |

### Enum References

| Prop/Field | Enum Ref | Values |
|-----------|----------|--------|
| {prop} | canonical-enum-map.yaml#enums.{name} | {values} |

### Permission Guards

| Action | Required Role | Guard |
|--------|--------------|-------|
| {view/click/edit} | {ADMIN/MANAGER/...} | {implementation hint} |

### Keyboard Navigation

| Key | Action | Context |
|-----|--------|---------|
| Enter | {action} | {when focused} |
| Escape | {action} | {when open} |

---

## Verification Checklist

- [ ] Verbatim HTML matches original exactly
- [ ] All Tailwind classes preserved
- [ ] No new HTML elements in Verbatim Base
- [ ] Loading state matches original dimensions
- [ ] Error state fits within same container
- [ ] All ARIA attributes added
- [ ] Enum references link to canonical-enum-map.yaml
- [ ] Permission guards documented
```

---

## Category Guidelines

| Category | Focus Areas |
|----------|------------|
| **data-display** | Loading skeletons, empty states, number formatting |
| **navigation** | Active state, keyboard nav, route guards |
| **forms** | Validation states, error messages, submit loading |
| **layout** | Responsive breakpoints, sidebar collapse, scroll |
| **feedback** | Auto-dismiss timing, stacking, screen reader announcements |
