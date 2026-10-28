---
name: bug-fixer
description: "Run the bug-fixer subagent directly. Reproduces a bug, finds its root cause, fixes it with the smallest diff, and adds a regression test. Use for failing tests, reviewer findings, and reported bugs."
argument-hint: <task>
disable-model-invocation: true
context: fork
agent: light-subagents:bug-fixer
---

$ARGUMENTS
