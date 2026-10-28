---
name: tester
description: "Run the tester subagent directly. Writes or runs tests for one unit and reports pass/fail with output. Never changes production code."
argument-hint: <task>
disable-model-invocation: true
context: fork
agent: light-subagents:tester
---

$ARGUMENTS
