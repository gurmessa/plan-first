# Contributing

Thanks for helping. This project is documentation; contributions are mostly Markdown.

## Ground rules

- Keep `SKILL.md` lean. It is the router; detail belongs in `references/` or `assets/`.
- The skill stays **instruction-only**: no scripts and no runtime dependencies.
- Frontmatter uses only the [Agent Skills specification](https://agentskills.io/specification)
  fields. Do not add agent-specific keys.
- Every template ends with an exit checklist.
- Diagrams are Mermaid sources, not screenshots.

## Ways to contribute

- **Report compatibility.** Add your agent to the table in `docs/compatibility.md`.
- **Improve a template.** Keep it generic; the worked example holds the specifics.
- **Fix a diagram.** Edit the `.mmd` source and the embedded copy together.
- **Add an example.** Use a real, finished plan, and say what it demonstrates.

## Before opening a PR

- [ ] `SKILL.md` frontmatter is valid (`name` matches the directory)
- [ ] No secrets, tokens, or personal data
- [ ] Links resolve
- [ ] `CHANGELOG.md` updated for user-visible changes

## License

By contributing, you agree your work is licensed under the MIT license in [LICENSE](LICENSE).
