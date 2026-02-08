# Visual Preservation Principle (Onboard Mode)

Reference document for all agents operating in **onboard** mode.

## Core Principle

> **Onboard = Refactoring, NOT Rewriting.**
>
> The rendered HTML output of every page MUST be pixel-identical to the original mockup.
> Internal code structure may change (componentization, route reorganization, import paths),
> but the user-visible result MUST NOT change.

---

## Rules

### Rule 1: Preserve Tech Stack

```
DO NOT upgrade or change:
- Framework version (e.g., Next.js 14 stays Next.js 14)
- CSS approach (e.g., Tailwind stays Tailwind, no shadcn/ui addition)
- State management (keep existing or add minimal if none exists)
- Build tooling

DO:
- Keep the same package.json dependencies (unless adding test/dev tools)
- Use the same React/Next.js patterns already in the codebase
```

### Rule 2: Verbatim HTML Extraction

When extracting inline JSX into components:

```
DO NOT:
- Redesign the HTML structure
- Replace Tailwind classes with different ones
- Add new design tokens, color variables, or spacing systems
- Add animations or micro-interactions not present in original
- Create new prop APIs that change rendering logic

DO:
- Copy the exact JSX structure into the component file
- Replace hardcoded values with props
- Keep all original className strings exactly as-is
- Keep the same HTML element types (div stays div, not Card)
```

#### Example: StatCard Extraction

**Original inline code:**
```tsx
<div className="bg-white rounded-2xl p-6 border border-slate-100 text-center">
  <div className="w-12 h-12 bg-blue-50 rounded-xl flex items-center justify-center mx-auto mb-3">
    <span className="text-blue-600 text-xl">👤</span>
  </div>
  <div className="text-sm text-slate-500 mb-1">출근 인원</div>
  <div className="text-2xl font-bold text-slate-800">287<span className="text-sm font-normal text-slate-400 ml-1">명</span></div>
</div>
```

**Correct extraction (verbatim):**
```tsx
interface StatCardProps {
  icon: string;
  iconBg: string;    // "bg-blue-50"
  iconColor: string; // "text-blue-600"
  label: string;
  value: string | number;
  unit?: string;
  valueColor?: string; // "text-slate-800" (default)
}

export function StatCard({ icon, iconBg, iconColor, label, value, unit, valueColor = 'text-slate-800' }: StatCardProps) {
  return (
    <div className="bg-white rounded-2xl p-6 border border-slate-100 text-center">
      <div className={`w-12 h-12 ${iconBg} rounded-xl flex items-center justify-center mx-auto mb-3`}>
        <span className={`${iconColor} text-xl`}>{icon}</span>
      </div>
      <div className="text-sm text-slate-500 mb-1">{label}</div>
      <div className={`text-2xl font-bold ${valueColor}`}>
        {value}
        {unit && <span className="text-sm font-normal text-slate-400 ml-1">{unit}</span>}
      </div>
    </div>
  );
}
```

**Wrong extraction (redesigned):**
```tsx
// DO NOT do this - changed HTML structure, different classes, new features
export function StatCard({ label, value, variant = 'default', trend }: StatCardProps) {
  return (
    <Card className="p-4">
      <CardHeader>{label}</CardHeader>
      <CardContent>
        <span className={cn('text-2xl', variantStyles[variant])}>{value}</span>
        {trend && <TrendIndicator value={trend} />}
      </CardContent>
    </Card>
  );
}
```

### Rule 3: Page Migration = Copy + Reorganize

When moving pages to new route structure:

```
DO NOT:
- Rewrite page content
- Replace inline elements with new component abstractions
- Change layout structure or spacing
- Add new features or UI elements

DO:
- Copy the entire page component to new route location
- Replace extracted inline patterns with the verbatim component
- Update import paths to match new file locations
- Update navigation links (href) to match new URL structure
- Keep all remaining inline JSX exactly as-is
```

### Rule 4: CSS Preservation

```
DO NOT:
- Remove or rename CSS classes from globals.css
- Replace custom CSS with Tailwind equivalents
- Add new CSS framework (shadcn/ui, Radix, etc.)
- Change CSS custom properties or variables

DO:
- Keep all existing globals.css content
- Keep all existing Tailwind config customizations
- Add new classes only if truly needed for new functionality
```

### Rule 5: Navigation Update Only

URL structure changes are allowed, but must not affect visual output:

```
ALLOWED:
- Change href values in links/buttons
- Add App Router route groups: (hr), (employee)
- Add layout.tsx files for shared layouts
- Update usePathname/useRouter calls

NOT ALLOWED:
- Change sidebar visual design
- Change header visual design
- Add/remove navigation items
- Change active state styling
```

---

## Verification Checklist

Before completing any onboard artifact, verify:

```yaml
visual_preservation_check:
  - Same HTML element types as original: true
  - Same Tailwind classes as original: true
  - Same CSS files preserved: true
  - No new UI library added: true
  - No framework version upgrade: true
  - Extracted components render identical HTML: true
  - Page layout unchanged: true
  - No new design tokens introduced: true
  - No new animations added: true
  - Props only parameterize existing variable values: true
```

---

## When This Applies

This document applies to ALL agents when the workflow mode is `onboard`:

| Agent | What to Preserve |
|-------|------------------|
| component-builder (P9) | Extract existing HTML verbatim, do not redesign |
| component-migrator (P10) | No structure improvements, copy as-is |
| dev-planner (P15) | Same tech stack, refactor-only approach |
| chapter-planner (P5) | Restructure organization, not visuals |
| ui-pattern-detector (P8) | Record exact HTML for extraction |
| context-synthesizer (P11) | Document preservation constraints |
