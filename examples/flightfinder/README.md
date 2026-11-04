# Worked example — FlightFinder Agent

A complete, real plan built with this method, so you can see what "done" looks like.

The project: a chat-based agent that finds one-way flights via the Kiwi.com MCP server,
asks clarifying questions with human-in-the-loop interrupts, and links the user to
Kiwi.com. Search-only; no booking.

Source: <https://github.com/gurmessa/flightfinder-agent>

## The document set

| Document | Demonstrates |
|----------|--------------|
| [`docs/PRD.md`](docs/PRD.md) | Testable requirements, explicit non-goals, resolved open questions |
| [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) | Component + sequence diagrams, ADRs, thread lifecycle |
| [`docs/WORKFLOW.md`](docs/WORKFLOW.md) | Runtime behavior: clarify-before-act rules, error/recovery |
| [`docs/TECH_STACK.md`](docs/TECH_STACK.md) | A reason and a rejected alternative for each dependency |
| [`docs/IMPLEMENTATION_PLAN.md`](docs/IMPLEMENTATION_PLAN.md) | M0–M7 milestones with exit criteria and a risk table |
| [`docs/STATUS.md`](docs/STATUS.md) | A living tracker used across multiple agent handoffs |
| [`docs/VERIFICATION.md`](docs/VERIFICATION.md) | Real browser end-to-end scenarios and recorded results |

## What to notice

- **The PRD states non-goals.** No booking, no auth, no persistence — so nobody re-asks.
- **The architecture lists rejected alternatives**, not just choices.
- **The workflow document owns behavior.** Rules like "clarify before acting" live there,
  and the tests point back at it.
- **The plan is milestone-based.** Each milestone is independently shippable and has
  explicit exit criteria.
- **The status file is the handoff artifact.** A new person or agent can resume from it
  without asking questions.

## What this example is not

- Not a template to copy verbatim — the flight/agent specifics are the point, the
  structure is what transfers.
- Not a claim that the plan was perfect. The source project's change log and open
  decisions record what was revised mid-build.

> **Note:** these are sanitised copies of the source project's docs, renamed to match this
> repo's template names (`AGENT_WORKFLOW.md` → `WORKFLOW.md`, `E2E.md` → `VERIFICATION.md`).
> File paths mentioned inside the docs refer to the source repository.
