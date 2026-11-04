---
name: plan-first
description: Plan a software project before coding. Use when starting a repo, writing a PRD, designing architecture, or turning an idea into milestones and diagrams. Produces PRD, architecture, workflow, stack, implementation plan, and status docs with Mermaid diagrams.
license: MIT
compatibility: Any agent that supports the Agent Skills specification. Markdown only; no scripts or runtime.
metadata:
  version: 0.1.0
---

# Plan First

Turn an idea into a buildable plan before writing code. You produce a small set of
documents, each answering one question, plus diagrams and a milestone plan.

## Workflow

1. **Brief** — ask the user 3–5 questions: problem, target user, non-goals, constraints,
   and one success metric. Do not proceed until you can restate the idea in one paragraph.
2. **PRD** — copy `assets/templates/PRD.template.md` to the user's `docs/PRD.md` and fill
   it in. Keep requirements testable and state the non-goals.
3. **Architecture** — write `docs/ARCHITECTURE.md`: components, data flow, key decisions
   (ADRs), and Mermaid diagrams.
4. **Workflow** — write `docs/WORKFLOW.md` describing runtime behavior: happy path, edge
   cases, and error/recovery flows.
5. **Tech stack** — write `docs/TECH_STACK.md`. For each major choice, give a reason and
   the rejected alternative.
6. **Milestones** — write `docs/IMPLEMENTATION_PLAN.md` with ordered milestones. Each has
   explicit exit criteria and dependencies.
7. **Status** — write `docs/STATUS.md` as a living tracker (legend, task table, changelog).
8. **Verification** — write `docs/VERIFICATION.md` with the test strategy and end-to-end
   scenarios.

## Rules

- One document answers one question. Keep the reading order obvious from the project README.
- Every diagram is Mermaid source, never a screenshot.
- Every requirement must be verifiable; every non-goal must be stated.
- Write the docs into the user's project. Do not invent extra documents beyond the set above.
- This skill is instruction-only: do not run scripts or install anything.
- Resolve bundled paths relative to this skill directory when reading templates or examples.

## Reference

- Templates: `assets/templates/`
- Diagram conventions: `assets/diagrams/README.md`
- Worked example: `examples/flightfinder/README.md`
