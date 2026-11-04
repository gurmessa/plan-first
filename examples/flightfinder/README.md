# Worked example — FlightFinder Agent

A complete, real plan built with this method, so you can see what "done" looks like.

The project: a chat-based agent that finds one-way flights via the Kiwi.com MCP server,
asks clarifying questions with human-in-the-loop interrupts, and links the user to
Kiwi.com. Search-only; no booking.

Source: <https://github.com/gurmessa/flightfinder-agent>

## What each document demonstrates

| Document in the source project | Demonstrates |
|--------------------------------|--------------|
| `docs/PRD.md` | Testable requirements, explicit non-goals, resolved open questions |
| `docs/ARCHITECTURE.md` | Component + sequence diagrams, ADRs, thread lifecycle |
| `docs/AGENT_WORKFLOW.md` | Runtime behavior: clarify-before-act rules, error/recovery |
| `docs/TECH_STACK.md` | Reasons and rejected alternatives for each dependency |
| `docs/IMPLEMENTATION_PLAN.md` | M0–M7 milestones with exit criteria and a risk table |
| `docs/STATUS.md` | A living tracker used across multiple agent handoffs |
| `docs/E2E.md` | Real browser end-to-end scenarios and recorded results |

The documents will be copied and sanitised into this folder as the example is written.
Until then, read them at the source link above.

## What this example is not

- Not a template to copy verbatim — the flight/agent specifics are the point, the
  structure is what transfers.
- Not a claim that the plan was perfect. See the project's change log and open
  decisions for the parts that were revised mid-build.
