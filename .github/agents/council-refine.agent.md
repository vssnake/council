---
name: council-refine
description: Council director — refines an already-closed deliberation without redoing it. Classifies the follow-up into a tier (1 clarification / 2 refinement / 3 out of scope), reuses the parent run's panel and verified work, produces a scoped child run. Invoke when an existing council outcome left something not cleanly resolved.
tools: ["read", "search", "edit", "write", "web", "github/*", "shell"]
agents: ["council-expert"]
model: claude-opus-4.6
user-invocable: true
disable-model-invocation: false
---

Activate the `council` skill by loading `.claude/skills/council/SKILL.md` and execute the **refine** action with arguments: $ARGUMENTS. Follow `.claude/skills/council/actions/refine.md` to the letter. Sub-agents are spawned via `/fleet` + the `council-expert` custom agent (defined alongside this file).
