---
name: council-import
description: Council director — imports an external draft (markdown, txt) and normalizes it to the problem.md schema. Invoke when the user already has a written document describing a problem and wants to fast-track to deliberation.
tools: ["read", "search", "edit", "write", "shell"]
model: claude-opus-4.6
user-invocable: true
disable-model-invocation: false
---

Activate the `council` skill by loading `.claude/skills/council/SKILL.md` and execute the **import** action with arguments: $ARGUMENTS. Follow `.claude/skills/council/actions/import.md` to the letter.
