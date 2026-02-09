# TDD Principles (Test-Driven Development)

> Source: tna Test-First (1500+ assertions), Progressive depth, 3-level system

## 1. Test-First Absolute Rule
- Tests must exist before implementation
- test-spec-generator creates .test.ts from DEV.md
- Implementation agents write "code that passes tests"
- "AI guesses then gets verified" PROHIBITED → "AI knows the target then codes"

## 2. 3-Level Test System
| Level | File Pattern | Verification Target | Tool |
|-------|-------------|---------------------|------|
| Unit | *.test.tsx | Component rendering, props, state | Vitest + RTL |
| Integration | *.integration.test.tsx | API integration, data flow | Vitest + MSW |
| E2E | *.spec.ts | Full user scenarios | Playwright |

## 3. Progressive Test Depth
| Phase | Unit | Integration | E2E |
|-------|------|-------------|-----|
| B1 (API) | API client methods | - | - |
| B2 early (Depth 3+) | Component render | API integration | Page loads |
| B2 mid (Depth 1-2) | + state transitions | + form submit | + Step-by-step |
| B2 late (Depth 0) | + edge cases | + cross-page | + Full workflow |

## 4. Test Coverage Criteria
- Unit: 80%+ line coverage
- Integration: Every API endpoint 1+ test
- Integration: RBAC matrix (role × endpoint × action) — suprema pattern
- E2E: All scenarios (DEV.md Scenarios section) covered
- Coverage deficit → supplement in next B2 iteration

## 4.1 RBAC Test Matrix (suprema pattern)
```
| Endpoint | Admin | Manager | Employee | Guest |
|----------|-------|---------|----------|-------|
| GET /records | ✅ all | ✅ team | ✅ self | ❌ 401 |
| POST /approve | ✅ | ✅ | ❌ 403 | ❌ 401 |
```
Auto-generated in conjunction with Permission Matrix (Principle 7)

## 5. Mock Strategy
- Unit: Mock all external dependencies
- Integration: API via MSW mock, components real
- E2E: Real dev server + mock API (or real API)
