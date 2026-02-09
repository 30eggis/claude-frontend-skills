# PRD Document Templates

Templates for generating production-quality PRD documents in the elevate workflow.

---

## Functional Requirement Template (FR-NNN)

```markdown
# FR-{NNN}: {Feature Name}

> Version: {X.X}
> Last Modified: {YYYY-MM-DD}
> Priority: P0 | P1 | P2

## 1. Overview
{Feature description derived from mockup exploration data.
What this feature does and why it matters.}

## 2. Functional Requirements

### 2.1 {Sub-area Name}

| ID | Requirement | Priority | Notes |
|----|------------|----------|-------|
| FR-{NNN}-01 | {Requirement from exploration data} | P0 | {notes} |
| FR-{NNN}-02 | {Requirement} | P1 | |

## 3. Business Rules
{Validation rules, constraints, calculations inferred from UI patterns.}

| Rule ID | Condition | Validation | UI Impact |
|---------|-----------|-----------|-----------|
| BR-{NNN}-01 | {when this condition} | {validate this} | {show/hide/disable} |

## 4. User Stories

- **{Role}**:
  - As a {role}, I want to {action}, so that {value}

## 5. Screen References

| Screen | Route | Persona | Mockup |
|--------|-------|---------|--------|
| {Screen Name} | /{path} | {persona} | {screenshot ref} |

## 6. Acceptance Criteria

- [ ] {Testable, binary criterion from mockup capabilities}
- [ ] {Another criterion}

## 7. Related FRs

| FR | Relationship | Notes |
|----|-------------|-------|
| FR-{MMM} | {depends on / extends / conflicts} | {details} |
```

---

## Screen Specification Template

```markdown
# {Screen Title}

> Version: {X.X}
> Last Modified: {YYYY-MM-DD}
> Route: /{path}
> Portal: {portal-name}
> Persona: {primary persona}

## 1. Screen Overview
{Purpose and context from exploration data.}

## 2. Access Permissions

| Role | Access | Notes |
|------|--------|-------|
| {role} | {RW/R/-} | {conditions} |

## 3. Layout Structure

```
{ASCII diagram derived from mockup screenshot analysis}
```

## 4. Component Details

### 4.1 {Component Name}
- **Type**: {card / table / form / chart}
- **Data**: {displayed data fields}
- **Actions**: {available user actions}
- **Component Ref**: {component spec reference}

## 5. Interactions

### 5.1 {Interaction Name}
- **Trigger**: {user action}
- **Result**: {navigation / modal / panel / toast}
- **Details**: {from click-todo.yaml exploration}

## 6. Related Functional Requirements

| FR | Description | Screen Element |
|----|------------|----------------|
| FR-{NNN}-{XX} | {requirement} | {which UI element implements it} |

## 7. Error Handling

| Scenario | Display | Recovery |
|----------|---------|----------|
| Data load failure | Error banner | Retry button |
| Empty data | Empty state message | Action link |
| Permission denied | Access denied page | Back navigation |
```

---

## Executive Summary Template

```markdown
# Executive Summary

> Version: {X.X}
> Last Modified: {YYYY-MM-DD}

## Project Overview
{What this project is, derived from exploration data analysis.}

## Key Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| {metric} | {target} | {how to measure} |

## Scope Summary

| Area | Screens | Features | Priority |
|------|---------|----------|----------|
| {area} | {count} | {count} | P0/P1/P2 |

## Persona Summary

| Persona | Role | Key Screens |
|---------|------|-------------|
| {name} | {description} | {screens} |

## Technical Boundaries
{Framework, constraints, integrations detected from project.}
```

---

## Document Index Template

```markdown
# Document Index

> Total Documents: {count}
> Last Updated: {YYYY-MM-DD}

## Status Legend
- DRAFT: Initial generation, needs review
- REVIEW: Under stakeholder review
- APPROVED: Finalized

## Documents

| Section | Document | Version | Status | Last Modified |
|---------|----------|---------|--------|---------------|
| 00 | project-scope.md | 1.0 | DRAFT | {date} |
| 00 | success-metrics.md | 1.0 | DRAFT | {date} |
| 01 | user-groups.md | 1.0 | DRAFT | {date} |
| 02 | FR-{NNN}-{name}.md | 1.0 | DRAFT | {date} |
| 03 | {portal}/{screen}.md | 1.0 | DRAFT | {date} |
```

---

## ID Conventions

| ID Type | Format | Example |
|---------|--------|---------|
| Feature Area | FR-NNN | FR-001, FR-002 |
| Sub-requirement | FR-NNN-XX | FR-001-01, FR-001-02 |
| Business Rule | BR-NNN-XX | BR-001-01 |
| Screen | SCR-{portal}-NNN | SCR-MGR-001 |

## Priority Definitions

| Priority | Meaning | Guideline |
|----------|---------|-----------|
| P0 | Must-have | Core workflow, visible in main navigation |
| P1 | Important | Important but deferrable to next phase |
| P2 | Nice-to-have | Enhancement, future consideration |
