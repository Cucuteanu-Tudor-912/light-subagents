---
name: doc-writer
description: "Run the doc-writer subagent directly. Updates README, docs and comments for a finished change. Docs only."
argument-hint: <task>
disable-model-invocation: true
context: fork
agent: light-subagents:doc-writer
---

$ARGUMENTS
