# IA Design Principles

> Source: tna Route Map (37 routes), 5 roles × 31 routes permission matrix

## 1. Persona-Portal Separation
- Route group = Persona portal: `(admin)/`, `(employee)/`, `(manager)/`
- If one screen is visible to multiple personas, separate page.tsx per route group
- Shared logic goes in shared components; screens themselves are portal-independent

## 2. Route Depth = Dependency Indicator
- Depth 0: Dashboard (entry point, independent)
- Depth 1: List pages (data listings, independent)
- Depth 2: Detail pages (entered from List, depends on List)
- Depth 3: Sub-actions (actions within Detail, depends on Detail)
- Rule: URL segment count = Depth

## 3. URL = State (No Prop Drilling)
- Every screen must restore its state from URL alone
- `/attendance/records/123` → page fetches via params.id directly
- Parent → child props passing prohibited
- SearchParams for filter/sort/pagination state

## 4. Navigation Tree Structure
- IA.md must be tree-shaped (indented)
- Each node: route, depth, persona, dependencies
- Leaf node = implementation unit, Branch node = layout/group

## 5. Permission Matrix Integration
- All routes specify accessible roles
- CRUD-level permissions (Read-only vs Full access)
- IA.md includes per-route permission column
