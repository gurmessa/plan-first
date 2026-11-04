# Status — FlightFinder Agent

**Single source of truth for implementation progress.**
Any agent or human picking up work must read this file first and update it before finishing.

**Companion docs:** [`IMPLEMENTATION_PLAN.md`](./IMPLEMENTATION_PLAN.md) (milestone definitions) ·
[`PRD.md`](./PRD.md) · [`ARCHITECTURE.md`](./ARCHITECTURE.md) · [`WORKFLOW.md`](./WORKFLOW.md) · [`TECH_STACK.md`](./TECH_STACK.md)

**Last updated:** 2025 — M7 complete on `milestone/m7-browser-e2e` (all milestones done)
**Updated by:** pi agent

---

## 1. Status Legend

| Symbol | Meaning |
|--------|---------|
| ⬜ `not started` | Nothing written; no files exist |
| 🟨 `in progress` | Partially implemented; **not** runnable/verifiable |
| 🟦 `review` | Implemented and self-verified; awaiting review |
| 🟩 `done` | Acceptance criteria met and verified |
| 🟥 `blocked` | Cannot proceed; reason recorded in Notes |
| ⚪ `deferred` | Intentionally postponed (recorded decision) |

**Rules**
- Only mark 🟩 when the milestone's **acceptance criteria** (in `IMPLEMENTATION_PLAN.md`) pass.
- Never mark a milestone 🟩 if a dependent milestone is still ⬜/🟨.
- Always fill **Updated by** and **Notes** when changing a row.
- If you stop mid-milestone, leave the row 🟨 and write exactly what remains in Notes.

---

## 2. Milestone Overview

| ID | Milestone | Status | Depends on | Owner | Updated by | Notes |
|----|-----------|--------|-----------|-------|-----------|-------|
| M0 | Scaffold & configuration | 🟩 done | — | pi agent | pi agent | `uv sync`, config load, imports, ruff, tracing toggle all verified |
| M1 | Kiwi MCP tools | 🟩 done | M0 | pi agent | pi agent | Live search verified; one-way adapter + unit tests |
| M2 | `user_ask` tools + HITL interrupts | 🟩 done | M0 | pi agent | pi agent | `interrupt_on` + `MemorySaver`; resume/prune helpers tested |
| M3 | Deep agent + system prompt | 🟩 done | M1, M2 | pi agent | pi agent | `build_agent()` + full prompt; scripted clarify→search→refine passes |
| M4 | Gradio UI + interrupt resume | 🟩 done | M3 | pi agent | pi agent | Chat UI + lazy agent; HTTP launch smoke passed; full browser E2E in M7 |
| M5 | Agent tests + model A/B | 🟩 done | M4 | pi agent | pi agent | Shared fixtures, clarification suite, live opt-in, compare script |
| M6 | Hardening + docs sync | 🟩 done | M5 | pi agent | pi agent | Search cap, retry/timeout, logging, docs synced |
| M7 | Browser E2E (Playwright MCP) | 🟩 done | M6 | pi agent | pi agent | 5/5 scenarios pass headed; LangSmith traces verified |

---

## 3. Detailed Task Tracking

### M0 — Scaffold & Configuration ⬜

| Task | Status | Owner | Notes |
|------|--------|-------|-------|
| `pyproject.toml` + `uv.lock` via `uv init`/`uv add` | 🟩 | pi agent | hatchling src layout; runtime + dev deps locked |
| `src/flight_finder/` package skeleton | 🟩 | pi agent | `config/models/agent/session/mcp/tools/ui` |
| `.env.example` + `.gitignore` | 🟩 | pi agent | `.env`, `.venv`, caches, `e2e-artifacts/` |
| `config.py` (dotenv + typed settings) | 🟩 | pi agent | Frozen dataclass `Settings` + `load_settings()` |
| `models.py` (`ChatOpenAI` factory) | 🟩 | pi agent | `build_model()` / cached `get_model()` |
| Module stubs: `agent.py`, `session.py`, `mcp/kiwi.py`, `tools/user_ask.py`, `ui/app.py`, `main.py` | 🟩 | pi agent | Stubs raise clear `NotImplementedError` per milestone |

**Exit criteria:** `uv sync` works; `uv run python -c "from flight_finder.config import settings; print(settings)"` prints config; tracing toggles via env only.

### M1 — Kiwi MCP Tools ⬜

| Task | Status | Owner | Notes |
|------|--------|-------|-------|
| `MultiServerMCPClient` HTTP wiring | 🟩 | pi agent | `KIWI_MCP_URL` fallback to `https://mcp.kiwi.com` |
| Tool filtering (`search-flight` in, `feedback-to-devs` gated) | 🟩 | pi agent | `KiwiToolset` + `select_kiwi_tools` |
| Trip-context → `search-flight` arg adapter | 🟩 | pi agent | `TripQuery` + `to_search_args`; strips return args |
| Live smoke test | 🟩 | pi agent | Both tools listed; real PRG→BCN search returned results |

**Exit criteria:** Live search returns itineraries; adapter never emits return-related args; `KIWI_MCP_URL` fallback works.

### M2 — `user_ask` Tools + HITL Interrupts ⬜

| Task | Status | Owner | Notes |
|------|--------|-------|-------|
| `ask_text` / `ask_choice` as `@tool` (defensive raise) | 🟩 | pi agent | Bodies raise if ever executed |
| `interrupt_on` config (`respond`) | 🟩 | pi agent | `INTERRUPT_ON` / `build_interrupt_on()` in `agent.py` |
| `Session` (thread_id, pending interrupts) | 🟩 | pi agent | `PendingQuestion` + `pending_questions()` |
| Checkpointer wiring (`MemorySaver`) | 🟩 | pi agent | `build_checkpointer()`; in-memory only |
| Thread pruning helper | 🟩 | pi agent | `prune_session()` deletes thread; blocks resume |
| `resume_with` test helper | 🟩 | pi agent | `run_turn` / `resume_with` / `build_decisions` |

**Exit criteria:** `ask_*` yields `result.interrupts`; resume continues the run; pruning prevents a later resume.

### M3 — Deep Agent + System Prompt ⬜

| Task | Status | Owner | Notes |
|------|--------|-------|-------|
| `create_deep_agent` wiring with all tools | 🟩 | pi agent | `async build_agent()`; Kiwi search + ask tools + `interrupt_on` + checkpointer |
| System prompt per WORKFLOW §8 | 🟩 | pi agent | Clarify-before-search, one-way, formatting, safety, errors |
| Thread-scoped trip context / refinement | 🟩 | pi agent | Prior turns persist via `MemorySaver` checkpoints |
| Interrupt dialogue verification | 🟩 | pi agent | Scripted ask → resume → search → refine test |

**Exit criteria:** No `search-flight` without required args; output has table + recommendation + `bookingUrl`.

### M4 — Gradio UI + Interrupt Resume ⬜

| Task | Status | Owner | Notes |
|------|--------|-------|-------|
| Chat interface with markdown rendering | 🟩 | pi agent | `gr.Blocks` + messages `Chatbot` |
| Per-session `gr.State` (`thread_id`, `pending`) | 🟩 | pi agent | `gr.State(new_session, delete_callback=...)` |
| Handler: `invoke` vs. `Command(resume)` | 🟩 | pi agent | `respond()` routes via `run_turn`/`resume_with` |
| Render interrupts as questions | 🟩 | pi agent | `pending_questions()` → `format_questions()` |
| Prune thread on session teardown | 🟩 | pi agent | Gradio delete callback calls `prune_session` |
| Hero-image marker block | 🟩 | pi agent | Hidden HTML marker block |

**Exit criteria:** Browser end-to-end flow works; refresh clears chat; errors are friendly.

### M5 — Agent Tests + Model A/B ⬜

| Task | Status | Owner | Notes |
|------|--------|-------|-------|
| `conftest.py` (interrupt harness + Kiwi fixtures) | 🟩 | pi agent | `tests/support.py` + fixtures |
| `test_agent_flow.py`, `test_clarification.py` | 🟩 | pi agent | Clarify/search/refine + missing-field/choice suites |
| Kiwi mocked by default; `RUN_KIWI_LIVE=1` opt-in | 🟩 | pi agent | Fake `search-flight`; live test separate |
| Model comparison script | 🟩 | pi agent | `scripts/compare_models.py` |

**Exit criteria:** `uv run pytest` green offline; live run green; model swap needs no code change.

### M6 — Hardening + Docs Sync ⬜

| Task | Status | Owner | Notes |
|------|--------|-------|-------|
| Bounded retries, timeouts, search cap | 🟩 | pi agent | 1 retry + 30 s/120 s timeouts; 3 searches/turn |
| Empty-result suggestions | 🟩 | pi agent | Explicit prompt rules + test |
| Structured logging | 🟩 | pi agent | `ToolCallLogger` + turn start/end logs |
| Docs sync (README/PRD/ARCHITECTURE/WORKFLOW) | 🟩 | pi agent | README, ARCHITECTURE, STATUS updated |

**Exit criteria:** No loops; no tracebacks to user; docs match behavior; clean-clone quickstart works.

### M7 — Browser E2E with Playwright MCP ⬜

| Task | Status | Owner | Notes |
|------|--------|-------|-------|
| Add Playwright MCP to `.mcp.json` (`npx @playwright/mcp@latest`) | 🟩 | pi agent | Fixed placement under `mcpServers` |
| Document E2E scenarios + how to run | 🟩 | pi agent | `docs/VERIFICATION.md` + `scripts/e2e_browser.py` |
| Scenario 1: happy path (complete inputs) | 🟩 | pi agent | Table + Kiwi booking links |
| Scenario 2: clarification → answer → search | 🟩 | pi agent | Question → resume → results |
| Scenario 3: refinement (direct) | 🟩 | pi agent | `max_sector_stopovers=0` search |
| Scenario 4: empty results → suggestions | 🟩 | pi agent | "No nonstop flights…" + budget/date/stop advice |
| Scenario 5: refresh resets chat | 🟩 | pi agent | Empty chat; no prior prices |
| Capture screenshots/traces on failure | 🟩 | pi agent | `e2e-artifacts/fail-*.png/.txt` |

**Exit criteria:** All 5 scenarios pass against the live app + real model + live Kiwi MCP; no
console errors or user-visible tracebacks; refresh prune verified; results recorded here.

---

## 4. Cross-Agent Handoff Protocol

When starting work:

1. Read `docs/STATUS.md` and the milestone section in `docs/IMPLEMENTATION_PLAN.md`.
2. Pick the **lowest unblocked** milestone (⬜ or 🟨 with all dependencies 🟩).
3. Set its row to 🟨, set **Owner** and **Updated by** to your agent name.
4. Work only within that milestone's scope; do not silently expand scope.

When finishing or stopping:

1. Update the milestone row and its task checkboxes.
2. Set **Updated by** and write a factual **Notes** entry:
   - what was completed, what remains, and any deviations from the plan.
3. Append an entry to §6 Change Log.
4. If you hit a blocker, set 🟥 and state exactly what is needed to unblock.

**Do not** mark work done without running its acceptance checks. **Do not** rewrite
milestone definitions here — those belong in `IMPLEMENTATION_PLAN.md`.

---

## 5. Verification Commands

Run these to check current state before claiming a milestone:

```bash
uv sync                                                     # M0
uv run python -c "from flight_finder.config import settings; print(settings)"   # M0
uv run pytest                                               # M5
uv run pytest -k clarification                              # M2/M5
RUN_KIWI_LIVE=1 uv run pytest                               # M1/M5 live
uv run python scripts/compare_models.py --models A,B       # M5 model A/B
uv run python -m flight_finder.main                         # M4 smoke
uv run python scripts/e2e_browser.py --headed                # M7 browser E2E (visible)
uv run python scripts/e2e_browser.py                        # M7 browser E2E (headless)
```

---

## 6. Change Log

Newest first. Keep entries short and factual.

| Date | Agent | Change |
|------|-------|--------|
| 2025 | pi agent | **M7 done** on `milestone/m7-browser-e2e`: headed Playwright MCP runner (`scripts/e2e_browser.py`), `.mcp.json` fixed, `docs/VERIFICATION.md`. **5/5 scenarios pass** (happy path 14.8s, clarification 21.2s, refinement 8.1s, empty 11.2s, refresh 2.1s). Also fixed async invocation for Kiwi MCP tools. LangSmith tracing verified (project `flightfinder-agent`). |
| 2025 | pi agent | **M6 done** on `milestone/m6-hardening`: per-turn search budget middleware, Kiwi retry + HTTP/SSE timeouts, `ToolCallLogger` + turn logs, empty-result prompt hardening, docs sync. Offline 31 passed / 1 skipped; live Kiwi test green. |
| 2025 | pi agent | **M5 done** on `milestone/m5-agent-tests`: `tests/support.py` + `conftest.py` shared doubles/fixtures; `test_clarification.py` (missing-field ask-before-search, defaults, ask_choice); offline suite 25 passed / 1 skipped; live Kiwi test green; `scripts/compare_models.py` for manual model A/B. |
| 2025 | pi agent | **M4 done** on `milestone/m4-gradio-ui`: `gr.Blocks` chat UI, per-session `gr.State` with `delete_callback` pruning, `respond()` routing new turns vs. resume, question rendering, friendly errors, lazy agent build. Verified with helper tests and an HTTP launch smoke (200, page served). Real browser flow remains M7. |
| 2025 | pi agent | **M3 done** on `milestone/m3-deep-agent`: async `build_agent()` wiring `create_deep_agent` with Kiwi `search-flight` + ask tools, `interrupt_on`, `MemorySaver`; expanded system prompt. `feedback-to-devs` intentionally not wired. Tests script clarify→search→refine and assert the required-arg / one-way invariants. |
| 2025 | pi agent | **M2 done** on `milestone/m2-user-ask-interrupts`: `INTERRUPT_ON` for both ask tools, `Session`/`PendingQuestion`, `MemorySaver` factory, `run_turn`/`resume_with`/`build_decisions`, pruning. Tests cover interrupt-without-execution, resume, and prune-blocks-resume. Note: LangGraph returns `result["__interrupt__"]`; `interrupts_of()` normalises that and `.interrupts`. |
| 2025 | pi agent | **M1 done** on `milestone/m1-kiwi-tools`: Kiwi toolset selection (`search-flight` vs gated `feedback-to-devs`), one-way `TripQuery` → `to_search_args` adapter (strips return-only args), unit tests + opt-in live test. Verified live search (USD, results returned). |
| 2025 | pi agent | **M0 done** on `milestone/m0-scaffold`: uv project, deps, src layout, `.env.example`, `.gitignore`, `config.py`, `models.py`, and stubs. Verified `uv sync`, settings load, imports, `ruff`, and env-only tracing toggle. |
| 2025 | docs agent | Added M7: final browser E2E via Playwright MCP (after M0–M6). Updated IMPLEMENTATION_PLAN, STATUS, ARCHITECTURE, TECH_STACK, PRD, AGENTS, README. No code exists yet. |
| 2025 | docs agent | Design change: replaced `QuestionBroker` with LangGraph `interrupt_on` + `MemorySaver` + `Command(resume)`. Updated ARCHITECTURE, PRD, WORKFLOW, TECH_STACK, IMPLEMENTATION_PLAN, STATUS. All milestones still ⬜; no code exists yet. |
| 2025 | docs agent | Created `STATUS.md`. All milestones M0–M6 recorded as ⬜ not started; no code exists yet. |

---

## 7. Known Blockers & Open Decisions

| # | Item | Impact | Status |
|---|------|--------|--------|
| B1 | Kiwi MCP not reachable from some networks during doc authoring | Live tests may be flaky; mocks required for offline runs | Mitigated: retry + timeout; live test opt-in |
| D1 | Render destination hero images inline in chat? | M4 optional task | Undecided |
| D2 | Search cap per turn (e.g. ≤ 5) | M6 hardening | Resolved: 3 per turn via `SearchBudgetMiddleware` |
| D3 | Aggressiveness of flexible-date widening | M3/M6 behavior | Undecided |
