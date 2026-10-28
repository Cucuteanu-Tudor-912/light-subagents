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
.claude-plugin/           plugin + marketplace manifests
agents/<role>.md          subagent definitions
skills/role-agents/       the orchestrator skill
skills/<role>/            slash-command wrappers, one per role
```

### Workflow

1. **Clarify.** Ambiguous requirements get questions before any planning.
2. **Plan.** Tasks are written to `.light-subagents/plan.md` (git-excluded) as a checklist.
3. **Branch.** All work happens on a feature branch, never on the default branch.
4. **Implement, test, review.** Each task goes through all three, then gets one commit.
5. **Final review.** A reviewer checks the whole branch diff.
6. **PR.** The orchestrator pushes and opens a PR. **It never merges.** Merging stays with you.

Agents never touch git. Only the orchestrator commits, pushes and opens PRs.

### Model tiering

| Difficulty | Examples | Model |
|---|---|---|
| Trivial | rename, run tests, apply a fully specified edit | haiku |
| Easy | implement 1-2 files from a complete spec, narrow search | haiku |
| Medium | multi-file implementation from a plan, bug fix with repro, review | sonnet |
| Hard | unclear root cause, design, ambiguous requirements | orchestrator, then split down |

Agents never run on Opus. Haiku gets only narrow tasks (about 5 files at most, with exact paths given) to stay well under 100k context.

## Install

In Claude Code:

```
/plugin marketplace add Cucuteanu-Tudor-912/light-subagents
/plugin install light-subagents@light-subagents
```

Restart Claude Code, then run `/agents`. The seven roles should be listed as `light-subagents:<role>`.

Update with `/plugin marketplace update light-subagents`. Uninstall with `/plugin uninstall light-subagents`.

## Usage

**Orchestrator mode.** Hand a whole task to the orchestrator:

```
/light-subagents:role-agents add rate limiting to the /login endpoint
```

The session investigates, plans, dispatches agents in parallel where possible, gates each change through test and review, and verifies before reporting. It also triggers on phrases like "orchestrate this with role agents".

**Single agent.** Call one role directly:

```
/light-subagents:investigator where are JWT tokens validated?
/light-subagents:reviewer review the staged diff against docs/spec.md
/light-subagents:bug-fixer tests/test_auth.py::test_expiry fails with KeyError
```

Typing `/investigator` and picking from autocomplete works too. You can also @-mention an agent or just ask ("use the reviewer agent on this diff").

## Customizing

Fork the repo and add your fork as the marketplace instead.

- **Change a role's default model:** edit `model:` in `agents/<role>.md`.
- **Add a role:** create `agents/<role>.md` (frontmatter: `name`, `description`, optional `tools`, `model`), and copy a wrapper from `skills/<role>/` with `agent: light-subagents:<role>`.
- **Tiering rules:** edit `skills/role-agents/SKILL.md`.
- **Test local changes:** `claude --plugin-dir ./light-subagents`.

## License

MIT
