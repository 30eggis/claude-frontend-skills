# Code Quality Principles (AI Native)

> Source: FRONTEND-COMPARISON-REPORT §7.2 "Architecture principles for higher AI code generation quality",
> ARCHITECTURE-REVIEW with-mock-server issues (632-line file, Prop Drilling 16+)

## 1. 200-Line File Limit
- Single file exceeding 200 lines prohibited (ensures AI context window comprehension)
- Exceeding → mandatory component extraction
- Page files handle composition only (<100 lines target)

## 2. Centralized Types
- `src/types/api.ts`: API response/request types (auto-generated from OpenAPI)
- `src/types/ui.ts`: UI component Props types (shared)
- `src/types/enums.ts`: Status values, role enumerations (generated from canonical-enum-map)
- Inline interface definitions in pages prohibited

## 3. No Prop Drilling
- Props passing beyond 3 levels prohibited
- Solutions: URL params, Context, or component-internal fetch
- with-mock-server anti-pattern: 16+ props passing → unmaintainable

## 4. Centralized Constants
- `src/constants/status-config.ts`: Status badge colors/labels
- `src/constants/routes.ts`: Route paths
- `src/constants/api-endpoints.ts`: API paths
- Hardcoding prohibited (especially colors, URLs, status values)

## 5. Input Validation Required
- All form submits require client-side validation (zod schema)
- Rules defined in DEV.md Validation section = implemented as zod schema
- Server response errors also displayed in UI (toast or inline error)

## 6. B0 Scaffold Auto-Generated Folders
```
src/
├── types/          # Centralized types
├── constants/      # Centralized constants
├── hooks/
│   ├── queries/    # GET hooks
│   └── mutations/  # POST/PUT hooks
├── components/
│   ├── ui/         # Primitives (Button, Input, Badge)
│   ├── composed/   # Composed (DataTable, FilterBar)
│   └── layout/     # AppShell, Sidebar
└── lib/api/        # API client
```
