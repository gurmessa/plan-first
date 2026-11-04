# PRD — FlightFinder Agent

**Status:** Draft v0.1
**Owner:** _(you)_
**Last updated:** 2025

---

## 1. Summary

FlightFinder Agent is a **chat-based assistant** that helps a user find **one-way flights**.
The user describes a trip in natural language, the agent fills in any missing details by
**asking clarifying questions**, then searches live flight data through the **Kiwi.com MCP
server** and presents ranked results with a recommendation and booking links.

The agent is built with **LangChain Deep Agents**, uses an **OpenRouter** model, exposes a
**Gradio** chat UI, and installs/manages everything via **uv**.

> **v1 is search only.** The agent never books, pays, or holds a fare — it hands the user a
> `bookingUrl` to complete the booking on Kiwi.com.

---

## 2. Goals

| # | Goal | Why it matters |
|---|------|----------------|
| G1 | Turn a vague trip request into a real search without a form | "Find me something cheap" should work |
| G2 | Never search with missing/ambiguous required inputs — **ask first** | Avoids wasted searches and wrong results |
| G3 | Present results clearly (cheapest / shortest / notable) plus a recommendation | User can decide in seconds |
| G4 | Support cheap, swappable LLMs via OpenRouter | Compare cost vs. quality per model |
| G5 | Keep architecture simple and testable | Small v1, fast iteration |

## 3. Non-Goals (v1)

- ❌ Round-trip, multi-city, or open-jaw searches (Kiwi supports them; we do not yet)
- ❌ Booking, payment, seat selection, or fare holds
- ❌ User accounts, login, or auth of any kind
- ❌ Persistence — refresh the page and the chat is gone
- ❌ Price alerts, background monitoring, saved searches
- ❌ Subagents / multi-agent orchestration
- ❌ Voice input, file uploads, flight-status tracking

---

## 4. Target User & Primary Stories

**Persona:** A casual traveler who knows roughly where and when they want to go but not the
exact flights, and who doesn't want to fill in a rigid search form.

**Stories**

1. **Happy path** — *"Cheap flight from Prague to Barcelona on 12/06/2026"*
   → agent searches immediately, shows a result table + recommendation.
2. **Missing date** — *"Flights from Berlin to Rome"*
   → agent asks for a departure date (choice or text), then searches.
3. **Ambiguous place** — *"To London"*
   → agent asks which airport/city or resolves automatically if unambiguous.
4. **Refinement** — *"Show me only direct ones under €80"*
   → agent re-searches with `max_sector_stopovers=0`, `price_to=80`.
5. **Preference discovery** — *"Business class, 2 adults, pay in USD"*
   → agent captures cabin class, passenger counts, and currency.
6. **No results** — *"Flights from a tiny island to another on that exact date"*
   → agent explains no results and offers flexible dates / nearby airports.

---

## 5. Functional Requirements

### FR-1 Chat interface
- FR-1.1 Single chat page (Gradio), text input + streaming agent responses.
- FR-1.2 Markdown rendering (tables, links, bold) in assistant messages.
- FR-1.3 Conversation is **in-memory only** and reset on page refresh/reload.
- FR-1.4 No login, no cookies required for functionality.

### FR-2 Natural-language trip parsing
- FR-2.1 Extract: origin, destination, departure date (and optional flexibility),
  passengers (adults/children/infants), cabin class, currency, locale,
  and soft preferences (stops, price cap, duration, airline).
- FR-2.2 Accept IATA codes, city names, and airport names (Kiwi resolves place names).
- FR-2.3 Normalize dates to `dd/mm/yyyy` for the Kiwi tool.

### FR-3 Clarification (`user_ask` tools)
- FR-3.1 The agent **must not call `search-flight`** until required inputs
  (`flyFrom`, `flyTo`, `departureDate`) are known.
- FR-3.2 The agent may optionally clarify high-impact preferences (stops, cabin, currency,
  passenger counts, price cap) but should default sensibly and proceed rather than
  over-questioning.
- FR-3.3 Two question tools are available:
  - `ask_text(question, ...)` → free-form single answer.
  - `ask_choice(question, options, allow_multiple=False)` → one or more options.
- FR-3.4 Questions are rendered as an assistant message in the same chat. The agent run is
  **interrupted** (`interrupt_on`) and resumed with the user's answer via
  `Command(resume={"decisions": [...]})`.
- FR-3.5 Defaults when the user is unsure: 1 adult, economy, USD, no filters.
- FR-3.6 The paused run's state is held by an **in-memory checkpointer** keyed by a
  per-session `thread_id`; it is pruned when the session ends so a reload cannot resume it.

### FR-4 Flight search (Kiwi MCP)
- FR-4.1 Use the Kiwi.com MCP tool **`search-flight`** over **HTTP transport** at
  `https://mcp.kiwi.com`. No API key or auth header is required.
- FR-4.2 Map parsed trip data to the tool's arguments (see §7).
- FR-4.3 One-way workflows must omit `returnDate` and all `return*` / `nights_in_dst_*` args.
- FR-4.4 The agent may issue **multiple searches** (e.g. flexible date bands, nearby
  airports) to answer a single request, but must not brute-force unbounded queries.
- FR-4.5 `feedback-to-devs` is available but only used when the user explicitly reports a
  search-tool problem; it must never be auto-invoked.

### FR-5 Result presentation
- FR-5.1 Group itineraries into **cheapest**, **shortest**, and **other notable** options.
- FR-5.2 Render a markdown table with: route, times/duration (local), cabin, price, link.
- FR-5.3 For each itinerary show route as `ORIGIN → [LAYOVER …] → DEST` from `outbound.route`.
- FR-5.4 Show baggage allowance from `itineraries[].baggage`.
- FR-5.5 End with: best price, shortest option, a recommendation, and a short fun fact.
- FR-5.6 Include the Kiwi `bookingUrl` for every itinerary.
- FR-5.7 Optionally show a destination hero image
  (`https://images.kiwi.com/photos/600x600/{imageId}.jpg`) when an `imageId` is present.

### FR-6 Refinement & follow-ups
- FR-6.1 Subsequent messages refine the previous search using the in-session context.
- FR-6.2 A refinement re-runs `search-flight` with updated args (no need to re-ask known facts).
- FR-6.3 "New search" resets trip context (route/dates) but keeps the conversation.

### FR-7 Errors & edge cases
- FR-7.1 MCP/network/tool errors produce a clear message and a retry offer; no stack traces
  shown to the user.
- FR-7.2 Empty result sets are reported with concrete suggestions (flex dates, nearby
  airports, relax filters).
- FR-7.3 Invalid or past dates are corrected via clarification.
- FR-7.4 If the model cannot call tools reliably, the agent surfaces a friendly failure
  rather than looping.

### FR-8 Configuration, secrets & tracing
- FR-8.1 All config/secrets come from environment variables, loaded from a local `.env`
  file (see `.env.example`).
- FR-8.2 Model config: `OPENROUTER_API_KEY`, `OPENROUTER_MODEL`, `OPENROUTER_BASE_URL`.
- FR-8.3 Default model: `nvidia/nemotron-3-ultra-550b-a55b:free` via
  `https://openrouter.ai/api/v1`.
- FR-8.4 LangSmith tracing is enabled by env vars and auto-instruments the agent
  (LangGraph-based): `LANGCHAIN_TRACING_V2`, `LANGCHAIN_API_KEY`, `LANGCHAIN_PROJECT`,
  `LANGCHAIN_ENDPOINT`.
- FR-8.5 `.env.example` is committed; the real `.env` is git-ignored.
- FR-8.6 Changing models must require **no code changes** — only env vars.

### FR-9 Browser end-to-end acceptance (final stage)
- FR-9.1 Once M0–M6 are complete, run a browser E2E pass with the **Playwright MCP** server
  driving the real Gradio UI (live model + live Kiwi MCP).
- FR-9.2 Cover at least: happy path, clarification/resume, refinement, empty results, and
  refresh-resets.
- FR-9.3 No uncaught console errors and no user-visible stack traces.
- FR-9.4 Confirm refresh prunes the `thread_id` so no stale conversation resumes.
- FR-9.5 Record pass/fail per scenario in `docs/STATUS.md` (M7).

---

## 6. Non-Functional Requirements

| Area | Requirement |
|------|-------------|
| **Model-agnostic** | Any OpenRouter tool-calling model can be swapped in via config. |
| **Latency** | First token / clarification question appears quickly; searches run once required inputs are known. |
| **Simplicity** | Single process; turn-based Gradio handler drives `invoke`/`Command(resume)` — no background task, queue, or external broker. |
| **Observability** | Log each agent turn, tool call, and search args/results count. |
| **Testability** | Agent behavior is testable **without the UI** (interrupt harness + mocked/real Kiwi MCP); final acceptance adds browser E2E via Playwright MCP. |
| **Env** | Managed by `uv`; any modern CPython. |
| **Cost control** | Model is configurable specifically so cheaper models can be evaluated. |

---

## 7. Kiwi `search-flight` Tool — Interface Reference

**MCP endpoint:** `https://mcp.kiwi.com` (HTTP, **no auth key**). Discovery exposed two tools:

| Tool | Purpose | Auto-use? |
|------|---------|-----------|
| `search-flight` | Search flights | ✅ primary tool |
| `feedback-to-devs` | Report bugs about the search tool | ❌ only on explicit user request |

**Returns:** `{ query, currency, passengers, resultsCount, itineraries, searchTimeMs }`

Each `itineraries[]` item:

- `price` (number), `priceFormatted` (e.g. `"123 EUR"`)
- `totalDurationSeconds`
- `bookingUrl` — link to book
- `imageId` — destination city id → `https://images.kiwi.com/photos/600x600/{imageId}.jpg`
- `baggage` — `{ personalItem, cabinBag, checkedBag }` counts, total across travelers
- `outbound` / `inbound` leg with: `route` (e.g. `["PRG","MAD","BCN"]`),
  `departureTime` / `arrivalTime` (local ISO), `durationSeconds`, `stops`, `cabinClass`,
  and `segments[]` (each segment: `from`/`to`, `fromCity`/`toCity`, `fromName`/`toName`,
  `fromCountry`/`toCountry`, `carrier`, `carrierName`, `flightNumber`, `cabinClass`)

**Key arguments**

| Argument | Notes |
|----------|-------|
| `flyFrom`, `flyTo` | Required. IATA code, city, or airport name (auto-resolved). |
| `departureDate` | Required. `dd/mm/yyyy`; earliest date if using a range. |
| `departureDateFlexDays` | Symmetric ±N days (0–10). Leave 0 when using `departureDateTo`. |
| `departureDateTo` | Asymmetric range end (`dd/mm/yyyy`). |
| `adults` / `children` / `infants` | Defaults `1 / 0 / 0`. |
| `cabinClass` | `M` economy, `W` premium economy, `C` business, `F` first. |
| `currency`, `locale` | Defaults `EUR`, `en`. |
| `max_sector_stopovers` | Max connections per sector (e.g. `0` = direct). |
| `price_from` / `price_to` | Total price filter. |
| `max_fly_duration` | Max total duration (hours). |
| `select_airlines` / `exclude_airlines` | Comma-separated IATA codes; mutually exclusive. |
| `dtime_from`/`dtime_to`, `atime_from`/`atime_to` | Local departure/arrival hour windows. |
| `stopover_*` / `*_conection` flags | Layover controls (duration, airports, countries, self-transfer). |
| `adults_hold_bags` / `adults_hand_bags` / child equivalents | Bag counts per traveler. |
| `one_for_city` | Single best result per destination (broad "explore" searches). |
| `fly_days` | Allowed outbound weekdays, `0=Sun..6=Sat`. |
| `sort` | `price` \| `duration` \| `quality` \| `date` \| `popularity`. |

> Return-only args (`returnDate`, `returnDateFlexDays`, `returnDateTo`,
> `nights_in_dst_*`, `ret_*`) are part of the tool but **unused in v1**.

---

## 8. `user_ask` Tools — Interface

Two local LangChain `@tool` declarations (`ask_text`, `ask_choice`) registered with the deep
agent's **`interrupt_on`**. They are **not** an MCP server; MCP is reserved for the remote Kiwi
integration. The tool bodies are never executed — the runtime intercepts the call.

| Tool | Input | Output on resume |
|------|-------|------------------|
| `ask_text` | `question: str`, `context?: str` | `answer: str` |
| `ask_choice` | `question: str`, `options: list[str]`, `allow_multiple?: bool = False` | `answer: str` (multi-select deferred) |

**Behavior**
1. The model emits an `ask_*` tool call.
2. `interrupt_on` stops the run, saves state to the checkpointer, and returns
   `result.interrupts`.
3. Gradio renders the question (buttons for choices, textbox for text).
4. On the next user message, the UI calls `Command(resume={"decisions": [...]})` with a
   `respond` decision; the answer becomes the tool result and the run continues.

**Design constraints**
- One **pause** (possibly containing multiple `action_requests`) at a time per session.
- A paused run survives only while its `thread_id` exists; a dropped thread means the user
  must be re-asked (start a new turn).
- Must be usable **headless in tests** by driving `result.interrupts` + `Command(resume)`
  directly.

---

## 9. Success Metrics (v1)

| Metric | Target |
|--------|--------|
| Successful search rate on complete requests | ≥ 95% |
| Clarification-to-search turnaround | ≤ 2 questions for incomplete requests |
| Missing-required-input searches | 0 (never call `search-flight` without required args) |
| Result rendering completeness | 100% of shown itineraries have route, times, price, link |
| Swappable models | Change model with env only |

---

## 10. Decisions & Assumptions

1. **Search only** — no booking integration in v1.
2. **One-way only** — return args are documented but unused.
3. **Ephemeral** — the checkpointer is in-memory only; the session's `thread_id` is pruned on
   reload so the chat cannot be resumed.
4. **No subagents** — a single deep-agent loop with tools.
5. **HITL via LangGraph interrupts** — `interrupt_on` + `Command(resume)`, with `MemorySaver`.
   (Replaces the earlier in-process `QuestionBroker` queue/future design.)
6. **Kiwi MCP is HTTP at `https://mcp.kiwi.com` with no auth key.**
7. **OpenRouter** via `ChatOpenAI(base_url="https://openrouter.ai/api/v1")`.
8. **Tests target the agent layer only**, enabling model A/B comparison for cost/quality.
9. **Gradio** is the UI framework; details chosen for simplicity.
10. **Local-only deployment**; config via `.env`, tracing via LangSmith env vars.
11. Default currency/locale: **USD / en**.
12. **Playwright MCP** is the final browser E2E tool, run after M6 as milestone M7.

---

## 11. Resolved Questions & Remaining Decisions

**Resolved**
1. Kiwi MCP requires **no key**; transport is HTTP at `https://mcp.kiwi.com`.
2. Default currency/locale: **USD / en**.
3. Hosting: **local only** (no Docker/cloud in v1).
4. `user_ask` is **not MCP** — it is two `@tool` declarations intercepted by `interrupt_on`.
   Only Kiwi (remote) uses MCP.

**Remaining (non-blocking, decide during build)**
1. Should result `imageId` hero images render inline in chat, or is the table enough?
2. How aggressively should the agent widen to flexible dates when a search returns nothing?
3. Should Kiwi rate limits be treated as a hard cap on searches per turn (e.g. ≤ 5)?
