# Browser E2E (M7)

End-to-end checks for the real Gradio app: **real OpenRouter model + live Kiwi MCP +
Playwright MCP** driving a real browser. This is a verification tool, not part of the app
runtime or the offline test suite.

## Prerequisites

- `OPENROUTER_API_KEY` in `.env` (plus optional LangSmith vars).
- Network access to `https://mcp.kiwi.com`.
- Node/npm available (`npx @playwright/mcp@latest`).
- A browser Chrome/Chromium installed. Chrome-for-Testing is only needed if you force
  `--browser chromium`.

## Run

```bash
# Visible browser (recommended when debugging)
uv run python scripts/e2e_browser.py --headed

# Headless (CI-style)
uv run python scripts/e2e_browser.py

# Options
uv run python scripts/e2e_browser.py --port 7867 --timeout 220 --headed
```

The runner:

1. Starts `python -m flight_finder.main` on `127.0.0.1:<port>` (logs to
   `e2e-artifacts/app.log`).
2. Starts the Playwright MCP server over stdio and drives Chrome.
3. Sends chat messages with `browser_type`, reads the page with `browser_evaluate`, and
   waits on rendered text.
4. Writes `e2e-artifacts/results.json`; on failure also saves `fail-<scenario>.png` and
   `.txt`.

## Scenarios

| # | Scenario | Input | Expected |
|---|----------|-------|----------|
| 1 | Happy path | Complete request with origin, destination, date | Results table with price + Kiwi booking link |
| 2 | Clarification/resume | "cheap flight from London to Lisbon" (no date) → answer the date | Question appears first; after the answer, results render |
| 3 | Refinement | "Only direct flights, please." after results | New search with `max_sector_stopovers=0`; direct options |
| 4 | Empty results | Nonstop London→Tokyo under 1 USD | Friendly "no flights" message + suggestions (budget/dates/stops) |
| 5 | Refresh reset | Reload the page after a conversation | Chat is empty; prior messages/prices are gone |

## Latest run

- **Date:** 2026-11-01
- **Model:** `openai/gpt-6-luna` (from `.env`; swap via `OPENROUTER_MODEL`)
- **Result:** **5/5 PASS**
- **LangSmith:** `LANGCHAIN_TRACING_V2=true`, project `flightfinder-agent`; agent runs
  (LLM + HITL middleware + tools) visible in LangSmith.
- **Evidence:** `e2e-artifacts/app.log` shows `search-flight` calls with `flyFrom`,
  `flyTo`, `departureDate` (no return args), plus the refinement (`max_sector_stopovers=0`)
  and empty-result (`price_to=1`) searches.

| Scenario | Result | Seconds |
|----------|--------|---------|
| Happy path | ✅ PASS | 14.8 |
| Clarification/resume | ✅ PASS | 21.2 |
| Refinement | ✅ PASS | 8.1 |
| Empty results | ✅ PASS | 11.2 |
| Refresh reset | ✅ PASS | 2.1 |

> Use a **future departure date** when running manually. The repository's environment
> clock is ahead; `15/07/2027` is used by the runner.
