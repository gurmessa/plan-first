# Diagram conventions

Diagrams are **sources**, not screenshots.

- Author diagrams in [Mermaid](https://mermaid.js.org/) and embed them in Markdown
  with fenced ` ```mermaid ` blocks so they render on GitHub.
- Keep a copy of each diagram as a standalone `.mmd` file in this directory when it is
  reused in more than one document.
- Give every diagram a caption that says what it teaches, not just what it shows.
- Prefer left-to-right flowcharts for pipelines and `sequenceDiagram` for request flows.
- Do not use binary-only images for architecture; they cannot be reviewed in a diff.

Suggested diagram set for a planned project:

| File | Teaches |
|------|---------|
| `planning-pipeline.mmd` | The order of planning stages |
| `component.mmd` | How the system is split into parts |
| `sequence-primary.mmd` | One request through the system |
| `state-session.mmd` | What state exists and when it resets |
| `milestones.mmd` | Dependencies between milestones |
