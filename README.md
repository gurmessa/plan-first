# plan-first

An agent skill that turns an idea into a buildable plan **before** any code is written.

It walks the agent through producing a small, opinionated document set — PRD, architecture,
workflow, tech stack, implementation plan, status tracker, and verification — with Mermaid
diagrams, so both humans and coding agents know what to build and when a milestone is done.

Works with any agent that supports the [Agent Skills specification](https://agentskills.io/specification).
Markdown only. No scripts, no runtime, no package manager.

## Install (git clone)

Because the repo **is** the skill (`SKILL.md` lives at the root), you install it by cloning
it into a skills directory.

```bash
# clone once to a portable location
git clone https://github.com/gurmessa/plan-first.git ~/.agents/skills/plan-first
```

If your agent does not scan `~/.agents/skills/`, symlink the clone into the directory it does:

```bash
ln -s ~/.agents/skills/plan-first ~/.claude/skills/plan-first    # example
ln -s ~/.agents/skills/plan-first ~/.pi/agent/skills/plan-first  # example
```

Or clone directly into a single agent's directory:

```bash
git clone https://github.com/gurmessa/plan-first.git ~/.claude/skills/plan-first
```

`git clone <url> <dir>` needs `<dir>` to not exist yet — which is exactly the case for a
fresh skill install.

## Where agents look

| Scope | Portable convention | Common agent paths |
|-------|---------------------|--------------------|
| User | `~/.agents/skills/` | `~/.claude/skills/`, `~/.pi/agent/skills/` |
| Project | `<repo>/.agents/skills/` | `<repo>/.claude/skills/` |

If your agent reads `~/.agents/skills/` or `.agents/skills/`, no symlink is needed. If it
uses its own directory, point a symlink at the single clone. Check your agent's docs for the
exact path, and note that project-level skills often require the project to be trusted first.

## Use it in a project

**Local only** — the global clone is already available; nothing to add.

**Symlink into the project (recommended):**
```bash
mkdir -p .agents/skills
ln -s ~/.agents/skills/plan-first .agents/skills/plan-first
```

**Commit it for the team** — use a submodule so the version is pinned:
```bash
git submodule add https://github.com/gurmessa/plan-first.git .agents/skills/plan-first
```

## Verify it loaded

- Restart the agent, or reload skills if your agent supports it.
- Force the skill where supported: `/skill:plan-first`.
- Then ask: *"Plan a new project: <one-line idea>."*

## Update / pin / uninstall

```bash
# update
git -C ~/.agents/skills/plan-first pull --ff-only

# pin a release
git clone --branch v0.1.0 --depth 1 \
  https://github.com/gurmessa/plan-first.git ~/.agents/skills/plan-first

# uninstall
rm -rf ~/.agents/skills/plan-first
# also remove any symlinks you created
```

## What the skill produces

| Stage | Document | Question it answers |
|-------|----------|---------------------|
| 1 | Brief | What are we making, for whom, why now? |
| 2 | `docs/PRD.md` | What exactly does v1 do, and not do? |
| 3 | `docs/ARCHITECTURE.md` | How is it structured, and why? |
| 4 | `docs/WORKFLOW.md` | How does it behave at runtime? |
| 5 | `docs/TECH_STACK.md` | What are we building with, and why? |
| 6 | `docs/IMPLEMENTATION_PLAN.md` | In what order, and when is each milestone done? |
| 7 | `docs/STATUS.md` | Where are we right now? |
| 8 | `docs/VERIFICATION.md` | How do we prove it works? |

## Repo layout

```
SKILL.md                  # the skill entry point
assets/templates/         # blank document templates
assets/diagrams/          # Mermaid conventions and sources
examples/flightfinder/    # a complete worked example
```

## Compatibility

Tested with agents that implement the Agent Skills specification. See
`examples/flightfinder/` for the result of applying the method to a real project.

If you test it on another agent, a PR documenting the result is welcome.

## License

MIT — see [LICENSE](LICENSE).
