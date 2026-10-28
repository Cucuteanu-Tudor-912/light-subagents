# light-subagents

Lightweight subagent-driven development for [Claude Code](https://claude.com/claude-code).

Your main session (e.g. Opus) is the **orchestrator**: it plans, writes specs, and makes judgment calls. The work goes to role-based subagents that run only on the cheaper **Sonnet** and **Haiku** models, chosen per task by difficulty.

There's no framework, no hooks and no session-start injection. It's one orchestration skill, seven agent files and seven slash-command wrappers.

## What's inside

| Role | Default model | Job | Writes |
|---|---|---|---|
| `investigator` | haiku | Find where/how something works, with file:line evidence | Nothing (read-only tools) |
| `implementer` | sonnet | Implement one task to spec, run its tests | Code |
| `tester` | haiku | Write/run tests for one unit, report pass/fail | Tests only |
| `bug-fixer` | sonnet | Reproduce, root-cause, minimal fix + regression test | Code |
| `reviewer` | sonnet | Severity-tagged review (BLOCKER/MAJOR/NIT) of a diff | Nothing (read-only tools) |
| `refactorer` | sonnet | Behavior-preserving restructure in small verified steps | Code |
| `doc-writer` | haiku | Update docs/comments for a finished change (Diátaxis) | Docs only |

Each agent's system prompt carries best practices for its role. The sources are Google's code review guide, Fowler's *Refactoring*, systematic debugging, unit-testing guidance and Diátaxis.

```
agents/<role>.md          native Claude Code subagent definitions
skills/role-agents/       the orchestrator skill  (/role-agents)
skills/<role>/            slash-command wrappers  (/investigator, /reviewer, ...)
```

### Model tiering

| Difficulty | Examples | Model |
|---|---|---|
| Trivial | rename, run tests, apply a fully specified edit | haiku |
| Easy | implement 1-2 files from a complete spec, narrow search | haiku |
| Medium | multi-file implementation from a plan, bug fix with repro, review | sonnet |
| Hard | unclear root cause, design, ambiguous requirements | orchestrator, then split down |

Agents never run on Opus. Haiku gets only narrow tasks (about 5 files at most, with exact paths given) to stay well under 100k context.

## Install

Copy `agents/` and `skills/` into your user-level `~/.claude` directory (or a project's `.claude/` directory to scope it to one repo).

**macOS / Linux / Git Bash**

```bash
git clone https://github.com/Cucuteanu-Tudor-912/light-subagents.git
mkdir -p ~/.claude/agents ~/.claude/skills
cp light-subagents/agents/*.md ~/.claude/agents/
cp -r light-subagents/skills/* ~/.claude/skills/
```

**Windows PowerShell**

```powershell
git clone https://github.com/Cucuteanu-Tudor-912/light-subagents.git
New-Item -ItemType Directory -Force "$HOME\.claude\agents", "$HOME\.claude\skills" | Out-Null
Copy-Item light-subagents\agents\*.md "$HOME\.claude\agents\"
Copy-Item -Recurse -Force light-subagents\skills\* "$HOME\.claude\skills\"
```

Then restart Claude Code. Check with `/agents` (the seven roles should be listed).

To update, `git pull` and copy again. To uninstall, delete the copied files.

## Usage

**Orchestrator mode.** Hand a whole task to the orchestrator:

```
/role-agents add rate limiting to the /login endpoint
```

The session investigates, plans, dispatches agents in parallel where possible, gates each change through test and review, and verifies before reporting.

**Single agent.** Call one role directly:

```
/investigator where are JWT tokens validated?
/reviewer review the staged diff against docs/spec.md
/bug-fixer tests/test_auth.py::test_expiry fails with KeyError
```

You can also @-mention an agent (`@agent-tester ...`) or just ask ("use the reviewer agent on this diff").

## Customizing

- **Change a role's default model:** edit `model:` in `agents/<role>.md`.
- **Add a role:** create `agents/<role>.md` (frontmatter: `name`, `description`, optional `tools`, `model`), and optionally copy a wrapper from `skills/<role>/`.
- **Tiering rules:** edit `skills/role-agents/SKILL.md`.

## License

MIT
