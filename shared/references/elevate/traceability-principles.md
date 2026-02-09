# Traceability Principles

> Source: tna 3-way Traceability (Mockup↔PRD↔API), Phase 6 Validation

## 1. 3-Way Traceability
All features must be cross-traceable across 3 sources:
```
Screen (Mockup) ←→ FR (PRD) ←→ API Endpoint
```

## 2. ID System
| Artifact | ID Pattern | Example |
|----------|-----------|---------|
| Functional Requirement | FR-NNN-XX | FR-002-01 |
| Business Rule | BR-{domain}-NN | BR-AT-01 |
| Screen | SCR-{portal}-NNN | SCR-ADMIN-001 |
| Component | CMP-{category}-NNN | CMP-DATA-001 |
| API Endpoint | API-{method}-{path} | API-GET-attendance-records |

## 3. DEV.md as Traceability Hub
Each DEV.md section references other artifacts:
- API Endpoints → api-map.md, OpenAPI
- Scenarios → FR, Business Rules
- Validation → Business Rules
- UI Spec → WIREFRAME.md → Screen Spec

## 4. Gap = Broken Trace
Must record in gap-report:
- Screen has button but no corresponding API
- FR defines feature not present in any Screen
- API endpoint exists but no Screen calls it
- Enum value exists in only one source

## 5. Automated Verification
In elevate P7 review:
- Confirm all DEV.md API endpoints exist in api-map
- Confirm all Screen spec features map to FRs
- Confirm enum value 3-way consistency
