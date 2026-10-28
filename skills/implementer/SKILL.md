---
name: implementer
description: "Run the implementer subagent directly. Implements one well-specified task (from a plan or spec) and runs its tests. Use for writing feature code once the design is decided."
argument-hint: <task>
disable-model-invocation: true
context: fork
agent: light-subagents:implementer
---

$ARGUMENTS
