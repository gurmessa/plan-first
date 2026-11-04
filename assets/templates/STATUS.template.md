# Status — <Project Name>

> **Purpose:** the single source of truth for where the project is right now.
> **Owner:** <name> · **Last updated:** <date>

This file is living. Update it whenever a task changes state, and always when you stop
work, so the next person or agent can resume without asking questions.

## 1. Status Legend

| Symbol | Meaning |
|--------|---------|
| ⬜ | Not started |
| 🟨 | In progress |
| 🟩 | Done |
| 🟥 | Blocked |
| ⏸️ | Paused / deferred |

## 2. Milestone Overview

| # | Milestone | Status | Depends on | Owner | Notes |
|---|-----------|--------|------------|-------|-------|
| M0 | <name> | ⬜ | — | <owner> | |

## 3. Detailed Task Tracking

### M0 — <name> ⬜

| Task | Status | Owner | Notes |
|------|--------|-------|-------|
| <task> | ⬜ | <owner> | |

<!-- Repeat per milestone. -->

## 4. Handoff Protocol

When handing off, a new contributor should be able to resume from this file alone:

1. Read the milestone overview and the in-progress tasks.
2. Read the linked plan section and relevant architecture.
3. Claim the lowest unblocked task (set 🟨 and add your name).
4. On stop, update status, notes, and the changelog below.

## 5. Verification Commands

```bash
# <command that proves the current milestone works>
```

## 6. Change Log

| Date | Who | What changed |
|------|-----|--------------|
| <date> | <name> | <change> |

## 7. Known Blockers & Open Decisions

| Item | Type | Owner | Status |
|------|------|-------|--------|
| <item> | blocker/decision | <name> | open/resolved |

## Exit checklist

- [ ] Legend present and used consistently
- [ ] Every in-progress task has an owner
- [ ] Change log is current
- [ ] Blockers and open decisions have owners
- [ ] A newcomer could resume from this file alone
