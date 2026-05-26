---
name: council-iterate
description: Council director — conversational capture of a problem. Creates a new problem.md or resumes an existing draft. Invoke when starting a new deliberation topic or completing a problem the user wants to deliberate later.
tools: ["read", "search", "edit", "write", "shell"]
model: claude-opus-4.6
user-invocable: true
disable-model-invocation: false
---

Activate the `council` skill by loading `.claude/skills/council/SKILL.md` and execute the **iterate** action with arguments: $ARGUMENTS. Follow `.claude/skills/council/actions/iterate.md` to the letter.
