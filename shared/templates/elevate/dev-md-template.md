# DEV: {PageName}
> Route: {route}
> Persona: {persona}
> Depth: {depth}
> Dependencies: [{dependencies}]

## UI Mode
{generative | original}

## API Endpoints
| Action | Method | Endpoint | Trigger |
|--------|--------|----------|---------|
| {action} | {GET|POST|PUT|DELETE} | {/api/v1/...} | {on-loaded|click-to-...|submit-...} |

## API-UI Binding
```
{trigger} => {METHOD} {endpoint} => {ui-update}
```

## Scenarios
| # | Scenario | Steps | Expected |
|---|----------|-------|----------|
| 1 | {scenario-name} | {user-steps} | {expected-outcome} |

## UI/UX State Changes
| State | Trigger | UI Change |
|-------|---------|-----------|
| loading | {trigger} | {ui-change} |
| error | {trigger} | {ui-change} |
| empty | {trigger} | {ui-change} |

## Validation
| Rule | Condition | Effect |
|------|-----------|--------|
| {rule-name} | {condition} | {effect} |

## TDD Plan
### Unit Tests
- [ ] {component} renders with correct props
- [ ] {component} handles loading state
- [ ] {component} handles error state
- [ ] {component} handles empty state

### Integration Tests
- [ ] Page loads data from API
- [ ] {interaction} flow works end-to-end

### E2E Tests
- [ ] {full-workflow-description}

## UI Spec
→ See WIREFRAME.md
