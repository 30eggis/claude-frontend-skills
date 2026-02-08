# UI Pattern Examples for Detection

Reference document for `ui-pattern-detector` agent.

## Approach

**Example-based, not criteria-based.**

Use these examples as templates to recognize similar patterns. The goal is to find repeated UI structures that should be extracted into reusable components.

---

## Pattern Category 1: Loop-based Repetition

### 1.1 Map Callback Bodies

Look for JSX inside `.map()` callbacks:

```tsx
// PATTERN: User list item
{users.map(user => (
  <div className="flex items-center gap-2">
    <div className="w-8 h-8 rounded-full bg-blue-500">{user.name[0]}</div>
    <div>
      <p className="font-medium">{user.name}</p>
      <p className="text-sm text-gray-500">{user.department}</p>
    </div>
  </div>
))}
```

**Extract as**: `UserListItem` or `UserBadge`

### 1.2 Table Row Patterns

```tsx
// PATTERN: Status cell with conditional styling
<td className={`px-4 py-2 ${
  status === 'approved' ? 'text-green-600' :
  status === 'rejected' ? 'text-red-600' : 'text-yellow-600'
}`}>
  {status}
</td>
```

**Extract as**: `StatusCell` or extend `Badge`

---

## Pattern Category 2: Manual Copy-Paste

### 2.1 Sibling Elements with Same Structure

```tsx
// PATTERN: Stat cards (identical structure, different values)
<div className="bg-white rounded-lg p-4 shadow">
  <p className="text-sm text-gray-500">출근 인원</p>
  <p className="text-2xl font-bold">287</p>
</div>
<div className="bg-white rounded-lg p-4 shadow">
  <p className="text-sm text-gray-500">지각 인원</p>
  <p className="text-2xl font-bold text-red-600">3</p>
</div>
<div className="bg-white rounded-lg p-4 shadow">
  <p className="text-sm text-gray-500">미출근</p>
  <p className="text-2xl font-bold text-yellow-600">12</p>
</div>
```

**Extract as**: `StatCard`

### 2.2 Button Pairs

```tsx
// PATTERN: Cancel/Confirm buttons
<div className="flex gap-2">
  <button className="px-4 py-2 border rounded">취소</button>
  <button className="px-4 py-2 bg-blue-600 text-white rounded">확인</button>
</div>
```

**Extract as**: `ButtonPair` or `DialogActions`

---

## Pattern Category 3: Cross-File Repetition

### 3.1 Modal/Card Headers

Same header pattern across multiple files:

```tsx
// File: leave-management/page.tsx
<div className="flex justify-between items-center mb-4">
  <h2 className="text-xl font-semibold">휴가 관리</h2>
  <button onClick={onClose}>×</button>
</div>

// File: overtime-management/page.tsx
<div className="flex justify-between items-center mb-4">
  <h2 className="text-xl font-semibold">연장근무 관리</h2>
  <button onClick={onClose}>×</button>
</div>
```

**Extract as**: `PageHeader` or `CardHeader`

### 3.2 Filter Bars

```tsx
// PATTERN: Filter bar (repeated in table pages)
<div className="flex gap-4 mb-4">
  <input type="date" className="border rounded px-3 py-2" />
  <input type="date" className="border rounded px-3 py-2" />
  <select className="border rounded px-3 py-2">
    <option>전체</option>
    <option>승인</option>
    <option>반려</option>
  </select>
  <button className="bg-blue-600 text-white px-4 py-2 rounded">검색</button>
</div>
```

**Extract as**: `FilterBar` or `DateRangeFilter`

---

## Pattern Category 4: Info Display Blocks

### 4.1 Label + Value Pairs

```tsx
// PATTERN: Info row
<div className="flex justify-between py-2 border-b">
  <span className="text-gray-500">신청일</span>
  <span className="font-medium">2024-01-15</span>
</div>
```

**Extract as**: `InfoRow` or `LabelValue`

### 4.2 Avatar + Text

```tsx
// PATTERN: User with avatar
<div className="flex items-center gap-3">
  <img src={user.avatar} className="w-10 h-10 rounded-full" />
  <div>
    <p className="font-medium">{user.name}</p>
    <p className="text-sm text-gray-500">{user.role}</p>
  </div>
</div>
```

**Extract as**: `UserAvatar` or `AvatarWithInfo`

---

## Pattern Category 5: Status Displays

### 5.1 Colored Status Badges

```tsx
// PATTERN: Status with conditional color
<span className={`px-2 py-1 rounded text-sm ${
  status === '승인' ? 'bg-green-100 text-green-800' :
  status === '반려' ? 'bg-red-100 text-red-800' :
  'bg-yellow-100 text-yellow-800'
}`}>
  {status}
</span>
```

**Extract as**: Extend existing `Badge` with status variants

### 5.2 Progress Indicators

```tsx
// PATTERN: Progress with label
<div>
  <div className="flex justify-between text-sm mb-1">
    <span>사용</span>
    <span>12/15일</span>
  </div>
  <div className="w-full bg-gray-200 rounded h-2">
    <div className="bg-blue-600 rounded h-2" style={{width: '80%'}} />
  </div>
</div>
```

**Extract as**: `ProgressBar` or `UsageIndicator`

---

## Detection Heuristics

1. **Occurrence threshold**: >= 3 times
2. **Structural similarity**: Ignore literal values, match tag structure
3. **ClassName patterns**: Similar utility classes = similar component
4. **Prop patterns**: Same prop names across occurrences

## Priority Scoring

| Factor | Weight |
|--------|--------|
| Occurrences | 40% |
| Files affected | 30% |
| Complexity saved | 20% |
| Maintenance risk | 10% |

## Output Format

For each detected pattern, generate:
1. Pattern name (proposed component name)
2. Occurrence count and locations
3. Structural template (with placeholders)
4. Proposed props interface
5. Generated component code

---

## Onboard Mode: Verbatim Extraction

> **When used in onboard mode, extraction MUST preserve the exact original HTML.**
> See `shared/references/onboard/visual-preservation.md` for full rules.

### Key Difference from Default Mode

| Aspect | Default Mode | Onboard Mode |
|--------|-------------|--------------|
| Goal | Design optimal component | Extract existing code as-is |
| HTML structure | May redesign | Must preserve verbatim |
| Tailwind classes | May optimize | Must keep exactly |
| Props | Design ideal API | Only parameterize differing values |
| New features | Encouraged | Forbidden |

### Onboard Extraction Example

**Detected pattern** (3 occurrences in hr-dashboard):

```tsx
// Occurrence 1 (line 45)
<div className="bg-white rounded-2xl p-6 border border-slate-100 text-center">
  <div className="w-12 h-12 bg-blue-50 rounded-xl flex items-center justify-center mx-auto mb-3">
    <span className="text-blue-600 text-xl">👤</span>
  </div>
  <div className="text-sm text-slate-500 mb-1">출근 인원</div>
  <div className="text-2xl font-bold text-slate-800">287<span className="text-sm font-normal text-slate-400 ml-1">명</span></div>
</div>

// Occurrence 2 (line 55)
<div className="bg-white rounded-2xl p-6 border border-slate-100 text-center">
  <div className="w-12 h-12 bg-red-50 rounded-xl flex items-center justify-center mx-auto mb-3">
    <span className="text-red-600 text-xl">⚠️</span>
  </div>
  <div className="text-sm text-slate-500 mb-1">지각 인원</div>
  <div className="text-2xl font-bold text-red-600">3<span className="text-sm font-normal text-slate-400 ml-1">명</span></div>
</div>
```

**Correct onboard extraction:**

```tsx
// Only parameterize the values that differ between occurrences
interface StatCardProps {
  icon: string;           // "👤" vs "⚠️"
  iconBg: string;         // "bg-blue-50" vs "bg-red-50"
  iconColor: string;      // "text-blue-600" vs "text-red-600"
  label: string;          // "출근 인원" vs "지각 인원"
  value: string | number; // "287" vs "3"
  valueColor?: string;    // "text-slate-800" vs "text-red-600"
  unit?: string;          // "명"
}

export function StatCard({ icon, iconBg, iconColor, label, value, valueColor = 'text-slate-800', unit }: StatCardProps) {
  return (
    // EXACT same structure - every className preserved
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

**Wrong onboard extraction (DO NOT):**

```tsx
// WRONG - redesigned with new abstractions, different classes, shadcn/ui
import { Card, CardContent } from '@/components/ui/card';

export function StatCard({ label, value, variant = 'default' }: StatCardProps) {
  return (
    <Card className="p-4">
      <CardContent className="flex flex-col items-center">
        <Typography variant="caption">{label}</Typography>
        <Typography variant="h3" color={variant}>{value}</Typography>
      </CardContent>
    </Card>
  );
}
```

### Onboard Output Format for duplicates/*.md

```markdown
# Pattern: {PatternName}

## Occurrences: {count}

| # | File | Lines | Differs |
|---|------|-------|---------|
| 1 | src/app/page.tsx | 45-53 | icon=👤, iconBg=bg-blue-50, label=출근 인원, value=287 |
| 2 | src/app/page.tsx | 55-63 | icon=⚠️, iconBg=bg-red-50, label=지각 인원, value=3 |

## Verbatim Template
```tsx
{exact original JSX with variable parts marked as {prop_name}}
```

## Variable Values
| Prop | Type | Occurrences |
|------|------|-------------|
| icon | string | "👤", "⚠️", "🏖️" |
| iconBg | string | "bg-blue-50", "bg-red-50", "bg-green-50" |

## Constraint
Extracted component MUST render identical HTML to the original inline code.
```
