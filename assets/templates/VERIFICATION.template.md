# Verification — <Project Name>

> **Purpose:** state how we prove the system works, at every level.
> **Owner:** <name> · **Last reviewed:** <date>

## 1. Strategy

| Layer | Scope | Runs when | Cost |
|-------|-------|-----------|------|
| Unit | Pure logic and helpers | Every change | Fast |
| Integration | Components with real or fake dependencies | Every change | Medium |
| End-to-end | The whole system as a user | Before release | Slow |
| Manual | Exploratory checks | Occasionally | Human |

Rules:
- Prefer fast tests; keep slow tests opt-in.
- Never let a test depend on a paid or live service by default.
- A bug fix adds a test that fails without the fix.

## 2. Unit Tests

- <What is covered>
- <What is deliberately not covered>

## 3. Integration Tests

| Dependency | Default | Live mode |
|------------|---------|-----------|
| <dependency> | mocked | enabled by `<ENV_FLAG>=1` |

## 4. End-to-End Scenarios

| # | Scenario | Input | Expected |
|---|----------|-------|----------|
| 1 | Happy path | <input> | <expected> |
| 2 | Failure path | <input> | <expected> |

## 5. How to Run

```bash
# fast suite
<command>

# live / opt-in suite
<ENV_FLAG>=1 <command>
```

## 6. Latest Results

- **Date:** <date>
- **Commit:** <sha>
- **Result:** <n> passed, <n> failed, <n> skipped

## 7. Failure Artifacts

<Where logs, screenshots, or traces are written when a test fails.>

## Exit checklist

- [ ] Every requirement in the PRD maps to a test or a scenario
- [ ] Live dependencies are opt-in
- [ ] Results are recorded with a date and commit
- [ ] Failure artifacts are preserved for debugging
