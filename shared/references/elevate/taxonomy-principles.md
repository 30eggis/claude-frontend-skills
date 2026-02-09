# Taxonomy Principles

> Source: tna Canonical Enum Map (38 concepts, ~180 values), Mockup↔PRD↔API cross-validation

## 1. Single Source of Truth
- canonical-enum-map.yaml = sole definition for all enum/status/type values
- Mockup shows "출근/지각/결근", PRD says "present/late/absent", API uses "PRESENT/LATE/ABSENT"
  → canonical-enum-map declares these 3 as the same concept

## 2. 3-Way Cross-Validation
| Source | Extraction Target | Method |
|--------|-------------------|--------|
| Mockup | Labels/badges/filter options displayed | Browser crawl + code analysis |
| PRD | States/types defined in business rules | FR document scan |
| API | OpenAPI enum definitions, response fields | OpenAPI parsing |

Mismatch found → record as ENUM_MISMATCH in gap-report

## 3. Enum Structure
```yaml
concept: attendance-status
display:
  ko: [출근, 지각, 조퇴, 결근, 휴가]
  en: [Present, Late, Early Leave, Absent, On Leave]
api:
  enum: [PRESENT, LATE, EARLY_LEAVE, ABSENT, ON_LEAVE]
  field: status
  source: GET /api/v1/attendance/records
prd:
  reference: FR-002-01
  business-rules: [BR-AT-01, BR-AT-02]
```

## 4. Korean↔English Mapping Required
- Korean UI projects with English APIs → mapping table mandatory
- display.ko → screen display
- api.enum → API communication
- Conversion functions are auto-generation targets

## 5. Enum Change Impact Analysis
- Adding/removing enum values → auto-track affected components/APIs
- Record reference locations in canonical-enum-map usage field
