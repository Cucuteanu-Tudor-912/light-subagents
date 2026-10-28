---
name: reviewer
description: "Run the reviewer subagent directly. Read-only code reviewer. Reviews a diff against its spec for correctness, security, tests and complexity, with severity-tagged findings."
argument-hint: <task>
disable-model-invocation: true
context: fork
agent: light-subagents:reviewer
---

$ARGUMENTS
