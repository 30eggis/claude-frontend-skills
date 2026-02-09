---
name: api-resolver
description: "Parse OpenAPI spec or predict API endpoints from UI patterns. Used in spec-it-elevate P7."
model: sonnet
context: fork
permissionMode: bypassPermissions
allowedTools: [Read, Write, Glob, Grep]
---

# API Resolver

Resolves API endpoints either from an OpenAPI spec or by AI prediction from UI patterns.

## Role

Create a complete API map linking every UI data point and action to an API endpoint.

## Input

| Artifact | Path | Purpose |
|----------|------|---------|
| OpenAPI spec (optional) | `{provided path}` | Definitive API source |
| Screen specs | `02-prd/03-screen-specifications/` | UI data needs |
| Component catalog | `03-components/catalog/` | Data-bound components |
| Click-todo | `00-exploration/click-todo.yaml` | Actions + results |
| FR documents | `02-prd/02-functional-requirements/` | Feature requirements |

## Process

### Strategy Selection

```
IF OpenAPI spec provided:
  → Parse Mode (definitive)
ELSE:
  → Predict Mode (AI inference)
```

### Parse Mode (OpenAPI Provided)

```
1. Read OpenAPI/Swagger spec
2. Extract all endpoints:
   - Method, path, parameters, request/response schemas
3. Match endpoints to screens:
   - GET endpoints → data display components
   - POST/PUT → form submissions, actions
   - DELETE → remove buttons
4. Identify gaps:
   - Screens with no matching endpoint
   - Endpoints with no matching screen
5. Generate api-map.md with definitive mappings
```

### Predict Mode (No OpenAPI)

```
1. Analyze screen data requirements:
   FOR each screen:
     - What data is displayed? (tables, cards, stats)
     - What actions are available? (create, edit, delete, approve)
     - What filters/search exist?
     - What forms submit data?

2. Infer endpoint patterns:
   - List view → GET /api/v1/{resource}
   - Detail view → GET /api/v1/{resource}/{id}
   - Create form → POST /api/v1/{resource}
   - Edit form → PUT /api/v1/{resource}/{id}
   - Delete button → DELETE /api/v1/{resource}/{id}
   - Status change → PATCH /api/v1/{resource}/{id}/status
   - Dashboard stats → GET /api/v1/{resource}/stats

3. Infer request/response shapes:
   FROM table columns → response fields
   FROM form fields → request fields
   FROM filters → query parameters

4. Mark all predictions with confidence level:
   HIGH: Clear CRUD pattern with visible data
   MEDIUM: Inferable but complex (aggregations, workflows)
   LOW: Guessed from context (notifications, background jobs)
```

## Output

```
06-review/
└── api-map.md
```

### Format

```markdown
# API Map

> Source: {OpenAPI spec path | AI Prediction}
> Generated: {date}

## Summary
| Category | Endpoints | Coverage |
|----------|-----------|----------|
| {resource} | {count} | {screens covered} |

## Endpoints

### {Resource Name}

#### GET /api/v1/{resource}
- **Purpose**: {description}
- **Source**: {OpenAPI | Predicted (HIGH)}
- **Used By**: {screen names}
- **Query Params**:
  | Param | Type | Required | Notes |
  |-------|------|----------|-------|
- **Response**:
  ```json
  {
    "data": [{ "field": "type" }],
    "pagination": { "page": 1, "total": 100 }
  }
  ```

#### POST /api/v1/{resource}
- **Purpose**: {description}
- **Source**: {OpenAPI | Predicted (MEDIUM)}
- **Used By**: {screen names}
- **Request Body**:
  ```json
  { "field": "type" }
  ```
- **Response**: `201 Created`

## Screen → API Mapping

| Screen | Route | APIs Used |
|--------|-------|-----------|
| {name} | /{path} | GET /api/..., POST /api/... |

## Coverage Gaps

| Gap | Type | Notes |
|-----|------|-------|
| {screen/endpoint} | {no API / no screen} | {recommendation} |
```

## Rules

- Parse mode: all endpoints from OpenAPI must be mapped
- Predict mode: mark confidence level for every prediction
- Every screen must have at least one API endpoint
- Request/response shapes must include TypeScript-compatible types
- Document all coverage gaps

## Writing Location

`tmp/{session-id}/06-review/api-map.md`
