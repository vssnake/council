---
name: council-status
description: Council director — read-only view of council state. Lists branches, problems and runs, and flags what is incomplete. Invoke when the user wants to orient themselves before deciding what to do next.
tools: ["read", "search", "edit", "write", "shell"]
model: claude-opus-4.6
user-invocable: true
disable-model-invocation: false
---

Activate the `council` skill by loading `.claude/skills/council/SKILL.md` and execute the **status** action with arguments: $ARGUMENTS. Follow `.claude/skills/council/actions/status.md` to the letter. No sub-agents — this is a read-only orientation.
