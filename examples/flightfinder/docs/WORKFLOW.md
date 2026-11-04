# Agent Workflow — FlightFinder Agent

**Companion docs:** [`PRD.md`](./PRD.md) · [`ARCHITECTURE.md`](./ARCHITECTURE.md) · [`IMPLEMENTATION_PLAN.md`](./IMPLEMENTATION_PLAN.md)

This document describes **how the deep agent behaves**: the decision loop, when it asks
questions, how it calls Kiwi, how it formats results, and how it handles refinement.

---

## 1. High-Level Loop

```mermaid
flowchart TD
    Start([User message]) --> Parse[Parse trip intent<br/>route, date, passengers,<br/>cabin, currency, filters]
    Parse --> Req{All required inputs<br/>flyFrom, flyTo, departureDate?}
    Req -- No --> Clarify[Ask via user_ask]
    Clarify --> Merge[Merge answer into trip context]
    Merge --> Req
    Req -- Yes --> Pref{High-impact preference<br/>missing and unclear?}
    Pref -- yes, and it changes results --> Clarify
    Pref -- no --> Search[Call search-flight<br/>with mapped args]
    Search --> Results{resultsCount > 0?}
    Results -- No --> NoRes[Explain + suggest<br/>flex dates / nearby airports / relax filters]
    NoRes --> Offer{User wants<br/>another try?}
    Offer -- yes --> Clarify
    Offer -- no --> End([End turn])
    Results -- Yes --> Format[Group cheapest / shortest / other,<br/>build markdown table + recommendation]
    Format --> Recommend[Best price, shortest option,<br/>fun fact, booking links]
    Recommend --> End
    End -->|follow-up message| Parse
```

> **Ask = interrupt.** Every `Clarify` step is an `interrupt_on` pause, not a blocking call.
> The run ends at the interrupt and resumes on the next user message (see
> [`ARCHITECTURE.md`](./ARCHITECTURE.md) §6).

---

## 2. Clarify-Before-Search Rules

Required inputs **must** be present before any search:

| Input | Source | If missing |
|-------|--------|-----------|
| `flyFrom` | text | ask (`ask_text` if open, `ask_choice` if a couple of likely options) |
| `flyTo` | text | ask |
| `departureDate` | text | ask; if the user is flexible, ask for a date **or** a range |

Optional-but-asked-when-decided matters:

```mermaid
flowchart TD
    Q{What is the user actually<br/>optimizing for?} -->|"cheapest"| P1[No filters needed, sort=price]
    Q -->|"fastest"| P2[max_fly_duration or sort=duration]
    Q -->|"direct only"| P3[max_sector_stopovers=0]
    Q -->|"specific airline"| P4[select_airlines]
    Q -->|"luxury"| P5[cabinClass=C or F]
    Q -->|"budget ceiling"| P6[price_to]
    Q -->|"day trip / time window"| P7[dtime_from/to, atime_from/to]
    P1 & P2 & P3 & P4 & P5 & P6 & P7 --> Ask{Clearly stated?}
    Ask -- yes --> Apply[Apply to search args]
    Ask -- no --> Clarify[Ask with ask_choice<br/>then apply]
```

**Do not over-question.** Defaults (1 adult, economy, USD, no filters) are assumed unless the
user says otherwise. Ask at most what materially changes the result.

### Clarification event flow (interrupt/resume)

```mermaid
sequenceDiagram
    participant Agent
    participant Graph as LangGraph runtime
    participant CP as MemorySaver
    participant UI as Gradio UI
    participant User

    Agent->>Graph: ask_choice("Max stops?", ["Any","Direct only","1 stop"])
    Graph-->>CP: save state at interrupt
    Graph-->>UI: result.interrupts [action_requests]
    UI-->>User: render options
    Note over Graph,CP: run ended, state checkpointed
    User->>UI: "Direct only"
    UI->>Graph: Command(resume={decisions:[respond]})
    Graph->>CP: load state
    Graph-->>Agent: tool result = "Direct only"
    Agent->>Agent: max_sector_stopovers = 0
```

> The `ask_*` tool body never executes. `interrupt_on` intercepts the call; the answer arrives
> as the tool result on resume.

---

## 3. Intent Parsing Contract

The agent extracts this internal trip context and keeps it for the session:

```json
{
  "flyFrom": "Prague",
  "flyTo": "Barcelona",
  "departureDate": "12/06/2026",
  "departureDateFlexDays": 0,
  "departureDateTo": null,
  "adults": 1,
  "children": 0,
  "infants": 0,
  "cabinClass": null,
  "currency": "USD",
  "locale": "en",
  "max_sector_stopovers": null,
  "price_to": null,
  "max_fly_duration": null,
  "select_airlines": null,
  "exclude_airlines": null,
  "sort": "price"
}
```

Normalization rules:
- Dates → `dd/mm/yyyy` (Kiwi's format). "next Friday" etc. are resolved to a concrete date.
- Place names kept as-is; Kiwi resolves IATA/city/airport automatically.
- Cabin words → `M`/`W`/`C`/`F` (economy / premium / business / first).
- Currency defaults to `USD`; locale to `en`.
- One-way: **never** set `returnDate`, `returnDateFlexDays`, `returnDateTo`, `nights_in_dst_*`,
  or `ret_*`.

---

## 4. Search Workflow

```mermaid
flowchart LR
    Ctx[Trip context] --> Map[Map to search-flight args]
    Map --> Call[Call search-flight]
    Call --> Raw["{ resultsCount, itineraries,<br/>searchTimeMs, currency }"]
    Raw --> Empty{Any itineraries?}
    Empty -- no --> Retry[Suggest flex dates /<br/>nearby airports / relax filters]
    Empty -- yes --> Group[Group results]
    Group --> Render[Render table + recommendation]
```

### Searching with flexibility
When the exact date yields nothing (or the user says "around then"), the agent may search a
**bounded** range:
- `departureDateFlexDays` (±N, 0–10) for symmetric flexibility, **or**
- `departureDateTo` for an asymmetric range.

The agent must avoid unbounded/brute-force searching (see IMPLEMENTATION_PLAN for a cap).

---

## 5. Result Formatting Workflow

Follow Kiwi's own display guidance.

```mermaid
flowchart TD
    I[itineraries] --> Cheapest[Cheapest: lowest price]
    I --> Shortest[Shortest: lowest totalDurationSeconds]
    I --> Other[Other notable options]
    Cheapest & Shortest & Other --> Table[Markdown table]
    Table --> Cols["Route · Times & duration · Cabin · Price · Link"]
    Cols --> Baggage[Show baggage counts]
    Baggage --> Rec[Best price + shortest option + recommendation]
    Rec --> Fun[Short destination fun fact]
    Fun --> Img{imageId present?}
    Img -- yes --> Hero[Optional hero image]
    Img -- no --> Done([Done])
```

Example table row semantics:

| Column | Source |
|--------|--------|
| Route | `outbound.route.join(" → ")` (e.g. `PRG → MAD → BCN`) |
| Times & duration | `outbound.departureTime` → `arrivalTime` (local) + `durationSeconds` |
| Cabin | `outbound.cabinClass` |
| Price | `priceFormatted` |
| Link | `[Book](bookingUrl)` |
| Baggage | `baggage.personalItem / cabinBag / checkedBag` |

---

## 6. Refinement Workflow

```mermaid
stateDiagram-v2
    [*] --> HasContext: completed a search
    HasContext --> HasContext: "show only direct"  (add max_sector_stopovers=0)
    HasContext --> HasContext: "under $80"          (add price_to=80)
    HasContext --> HasContext: "cheaper dates"      (widen flex days)
    HasContext --> HasContext: "2 adults"          (update passenger counts)
    HasContext --> Reset: "new search: X to Y"
    Reset --> [*]
    HasContext --> [*]: "thanks / no more"
```

- Follow-ups **reuse** the stored trip context; only changed fields are updated.
- A refinement re-calls `search-flight` with merged args.
- "New search" clears route/date but keeps the conversation history.

---

## 7. Error & Recovery Workflow

```mermaid
flowchart TD
    Tool[Tool call] --> Ok{Success?}
    Ok -- yes --> Continue[Continue]
    Ok -- no, transient --> Retry[Retry once, then explain + offer retry]
    Ok -- no, empty results --> Suggest[Explain + concrete alternatives]
    Ok -- no, invalid args --> Fix[Re-clarify and retry]
    Retry & Suggest & Fix --> Continue
    Continue --> Done([User-informed outcome])
```

Rules:
- Never expose stack traces; log them and show a friendly message.
- Never loop indefinitely: bounded retries, then a clear failure.
- If an interrupt is abandoned (session refresh, pruned thread), drop the pending
  `action_requests`, start a fresh turn, and report what is still needed.
- If `interrupt_on` is missing for an `ask_*` tool, the tool body raises; surface a clear
  configuration error rather than a fake answer.

---

## 8. System Prompt Outline

The deep agent's system prompt encodes the following (single agent, no subagents):

1. **Role** — a helpful, concise one-way flight finder.
2. **Hard rule** — never call `search-flight` without `flyFrom`, `flyTo`, `departureDate`.
3. **Clarification policy** — ask only material questions; use defaults otherwise.
4. **Tool policy** — `search-flight` for searches; `feedback-to-devs` only on explicit request;
   `ask_text`/`ask_choice` for questions.
5. **One-way policy** — omit all return-related args.
6. **Formatting policy** — markdown table grouped cheapest/shortest/other; include baggage,
   price, and booking link; end with recommendation + fun fact.
7. **Safety policy** — never claim to have booked or held a fare; always link out.
8. **Error policy** — friendly failures, bounded retries, offer alternatives.

---

## 9. Worked Example

```mermaid
sequenceDiagram
    actor U as User
    participant UI as Gradio UI
    participant A as Agent
    participant CP as MemorySaver
    participant K as Kiwi

    U->>UI: cheap flight to Lisbon from London
    UI->>A: invoke({messages}, thread_id)
    A->>A: ask_text("What date would you like to depart?")
    A-->>CP: checkpoint at interrupt
    A-->>UI: result.interrupts
    UI-->>U: question shown
    U->>UI: around mid-July, flexible
    UI->>A: invoke(Command(resume={decisions}), thread_id)
    A->>CP: load state
    A->>A: departureDate=15/07/2026, flex=3
    A->>K: search-flight(flyFrom=London, flyTo=Lisbon,<br/>departureDate=15/07/2026, departureDateFlexDays=3,<br/>currency=USD, sort=price)
    K-->>A: itineraries[]
    A-->>UI: cheapest / shortest table + recommendation + fun fact
    UI-->>U: results
    U->>UI: only direct ones
    UI->>A: invoke({messages}, thread_id)
    A->>CP: load prior trip context
    A->>K: search-flight(..., max_sector_stopovers=0)
    K-->>A: itineraries[]
    A-->>UI: refined table
    UI-->>U: refined results
```

---

## 10. Agent Invariants (checked in tests)

- 🔒 No `search-flight` call without the three required args.
- 🔒 Defaults are `adults=1`, `cabinClass=null (economy)`, `currency=USD`, `locale=en`.
- 🔒 No return-related arguments are ever sent (one-way only).
- 🔒 Every presented itinerary includes route, local times/duration, price, and a `bookingUrl`.
- 🔒 `feedback-to-devs` is never auto-invoked.
- 🔒 `ask_text` and `ask_choice` are registered in `interrupt_on`; a clarification turn
  produces an interrupt **before** any `search-flight` call.
- 🔒 Resuming with a `respond` decision continues the run to `search-flight` (no restart from
  scratch, no duplicated side effects).
- 🔒 The agent never claims to have booked or paid.
