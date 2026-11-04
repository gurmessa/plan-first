# Diagram conventions

Diagrams are **sources**, not screenshots.

- Author diagrams in [Mermaid](https://mermaid.js.org/) and embed them in Markdown
  with fenced ` ```mermaid ` blocks so they render on GitHub.
- Keep a copy of each diagram as a standalone `.mmd` file in this directory when it is
  reused in more than one document.
- Give every diagram a caption that says what it teaches, not just what it shows.
- Prefer left-to-right flowcharts for pipelines and `sequenceDiagram` for request flows.
- Do not use binary-only images for architecture; they cannot be reviewed in a diff.

## Diagram set

| File | Teaches |
|------|---------|
| `planning-pipeline.mmd` | The order of planning stages |
| `doc-dependencies.mmd` | Which document must exist before which |
| `component.mmd` | How the system is split into parts |
| `sequence-primary.mmd` | One request through the system, including a question step |
| `state-session.mmd` | What state exists, who owns it, and when it resets |
| `milestones.mmd` | Dependencies between milestones |

## Rendering

GitHub renders Mermaid in Markdown natively. To export images for slides or docs, use the
Mermaid CLI:

```bash
npm install -g @mermaid-js/mermaid-cli
mmdc -i planning-pipeline.mmd -o planning-pipeline.svg
```
