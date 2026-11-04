# Compatibility

`plan-first` follows the [Agent Skills specification](https://agentskills.io/specification).
It contains only Markdown and Mermaid — no scripts, no runtime, no package manager.

## Structural validation

| Check | Result |
|-------|--------|
| `SKILL.md` at repo root | ✅ |
| Frontmatter uses spec fields only (`name`, `description`, `license`, `compatibility`, `metadata`) | ✅ |
| `name` matches the directory (`plan-first`) | ✅ |
| `name` is lowercase-hyphenated, ≤64 chars | ✅ |
| `description` ≤1024 chars and states what + when | ✅ |
| No scripts or executable assets | ✅ |

## Runtime reports

Any agent that discovers `SKILL.md` files and supports the specification should load this
skill. Install locations differ per agent; see the [README](../README.md#where-agents-look).

| Agent | Version | Result | Notes |
|-------|---------|--------|-------|
| — | — | *no reports yet* | Be the first |

## Test it in your agent

1. Clone into a location your agent scans (or symlink the clone in):
   ```bash
   git clone https://github.com/gurmessa/plan-first.git ~/.agents/skills/plan-first
   ```
2. Restart the agent, or reload skills if supported.
3. Force-load where supported, e.g. `/skill:plan-first`.
4. Ask: *"Plan a new project: a CLI that renames photos by EXIF date."*
5. Confirm it creates `docs/PRD.md` and walks through the rest of the set.

## Reporting

Open a PR adding a row to the runtime table, or open an issue with:

- agent name and version
- install path used
- what worked and what did not

## Known constraints

- Skills that agent-specific implementations place in non-standard directories need a symlink.
- Some agents require the project to be trusted before project-level skills load.
- Mermaid rendering depends on the viewer; GitHub renders it natively.
