---
name: role-agents
description: Use when doing engineering work by dispatching subagents - this conversation becomes the orchestrator and delegates to role-based agents (investigator, implementer, tester, bug-fixer, reviewer, refactorer, doc-writer) running only on sonnet or haiku. Triggers - "role agents", "orchestrate", "dispatch agents", "subagent driven", "use smaller models".
---

# Role Agents

This conversation is the **orchestrator**. It plans, dispatches, verifies, and decides. It does not do bulk work itself: no wide file reading, no large edits. Agents do the work. The orchestrator holds the context.

## Hard rules

1. **Models: `sonnet` or `haiku` only.** Always pass `model` explicitly on every Agent call. Never omit it, because an omitted model inherits the session model. Never use opus.
2. **Haiku context cap: 100k tokens.** Only give haiku a task that fits comfortably: at most about 5 files to read, one narrow question or one small edit, with exact file paths given. If the task might sprawl (unknown location, many files, long logs), use sonnet or split the task first.
3. **Tier by difficulty.** The orchestrator (the session model, e.g. Opus) does the hard thinking: understanding the problem, design, planning, writing specs, making judgment calls. Agents execute. Pick each agent's model by difficulty, not by role:

   | Difficulty | Examples | Model |
   |---|---|---|
   | Trivial | rename, run tests, apply a fully specified edit, doc touch-up, single-file lookup | haiku |
   | Easy | implement 1-2 files from a complete spec, write tests for a known unit, narrow search | haiku (sonnet if it might exceed 100k) |
   | Medium | multi-file implementation from a plan, bug fix with known repro, review, refactor | sonnet |
   | Hard | unclear root cause, design decision, ambiguous requirements | **orchestrator handles it.** Split it or investigate it until it becomes Medium, then dispatch sonnet |

   A written plan or spec makes a task easier, so tier it down. If the plan is detailed enough that implementing it is mechanical, use haiku.
4. **One role per agent, one task per agent.** Use the matching `subagent_type`.
5. **Self-contained briefs.** An agent sees only its prompt. Include the goal, exact paths, what is already known or ruled out, constraints, and the expected report format.
6. **Parallelize independent work.** Put independent dispatches in one message. Never run two writers on the same files at once. Use `isolation: "worktree"` for parallel writers.
7. **Verify, don't trust.** An agent's report is a claim. Check key claims with a quick grep, read or test run, or a tester agent, before acting on them or telling the user they're done.

## Roles

Each role is a subagent shipped by this plugin (`agents/<role>.md`). Its role best practices, tool limits and report format live in that file's system prompt, so don't repeat them in briefs. Dispatch with `subagent_type="light-subagents:<role>"` and an explicit `model` (rule 3 overrides the file's default).

| Role (subagent_type) | Default model | Job | Writes |
|---|---|---|---|
| investigator | haiku | Find where/how something works, with file:line evidence | Nothing |
| implementer | sonnet | Implement one task to spec, run its tests | Code |
| tester | haiku | Write/run tests for one unit, report pass/fail | Tests only |
| bug-fixer | sonnet | Reproduce, root-cause, minimal fix + regression test | Code |
| reviewer | sonnet | Severity-tagged review of a diff against spec | Nothing |
| refactorer | sonnet | Behavior-preserving restructure in small steps | Code |
| doc-writer | haiku | Update docs/comments for a finished change | Docs only |

To add a role, add `agents/<role>.md` to the plugin with the same shape.

## Brief template

```
GOAL: <one sentence>
CONTEXT: <what the orchestrator knows; paths; prior findings; what's ruled out>
SCOPE: <files/dirs you may touch; anything off-limits>
DONE WHEN: <concrete acceptance criteria, e.g. "pytest tests/test_x.py passes">
```

## Orchestration loop

1. **Understand.** Restate the goal. If the codebase is unfamiliar, dispatch investigators in parallel, not by hand.
2. **Clarify.** If the requirements are ambiguous (scope, expected behavior, edge cases, constraints), ask the user before planning. Ask only questions whose answers change the plan, all in one batch. Don't plan on unstated assumptions.
3. **Plan.** Break the work into small tasks with a role, model, file scope and acceptance criteria for each. For more than 3 tasks, write the plan to `.light-subagents/plan.md` in the repo root (add `.light-subagents/` to `.git/info/exclude`) as a checklist, show it to the user, and tick tasks off as they finish. Re-read it after a context summary.
4. **Branch.** In a git repo, never work on the default branch. Create `feat/<short-name>` (or `fix/...`) first. If you're already on a feature branch, stay on it.
5. **Dispatch.** Run independent tasks in parallel and dependent ones in sequence.
6. **Gate each writing task:** implementer → tester (or tests run by the implementer, then spot-checked) → reviewer for non-trivial diffs. Brief the reviewer with the spec and the diff, not with the implementer's report.
7. **Commit.** After a task passes its gate, the orchestrator commits it on the branch, one commit per task, with a message that says what and why.
8. **Handle the status:**
   - BLOCKED or NEEDS_CONTEXT → add context, or split the task, then re-dispatch.
   - Haiku failed → retry once on sonnet with a better brief.
   - Sonnet failed twice → stop and report to the user. Do not escalate to opus.
   - Test failure or a reviewer finding → dispatch a bug-fixer with the exact failure output.
9. **Final review.** Dispatch one sonnet reviewer on the whole branch diff against the original goal, to catch problems between tasks.
10. **Finish with a PR, never a merge.** Run the final verification yourself (one test command), push the branch, and open a PR (`gh pr create`) whose body lists what changed, how it was verified, and what's open. **Never merge, never push to the default branch, never force-push.** Merging is the user's call. Then give the user a short summary and the PR link.

**Agents never commit, push, or open PRs.** Only the orchestrator does git integration. The agent files already say so.

## Example

```
Agent(description="Find auth middleware", subagent_type="light-subagents:investigator", model="haiku",
      prompt="GOAL: Locate where JWT tokens are validated...")
Agent(description="Find token tests", subagent_type="light-subagents:investigator", model="haiku", prompt="GOAL: ...")
# -> then
Agent(description="Add token expiry check", subagent_type="light-subagents:implementer", model="sonnet",
      prompt="GOAL: ...
SCOPE: src/auth/jwt.ts only
DONE WHEN: npm test -- jwt passes")
```
