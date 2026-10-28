---
name: refactorer
description: "Run the refactorer subagent directly. Behavior-preserving refactoring in small verified steps, with tests passing before and after."
argument-hint: <task>
disable-model-invocation: true
context: fork
agent: light-subagents:refactorer
---

$ARGUMENTS
