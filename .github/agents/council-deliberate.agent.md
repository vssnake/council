---
name: council-deliberate
description: Council director — launches a deliberation on a closed problem.md. Designs a 4-6 expert panel + 0-2 friction archetypes, runs 4 rounds (proposals / critiques / lead-mediated debate / final positions), and produces a synthesis via an impartial moderator. Invoke when the problem is captured (status open) and the user wants a recommendation.
tools: ["read", "search", "edit", "write", "web", "github/*", "shell"]
agents: ["council-expert"]
model: claude-opus-4.6
user-invocable: true
disable-model-invocation: false
---

Activate the `council` skill by loading `.claude/skills/council/SKILL.md` and execute the **deliberate** action with arguments: $ARGUMENTS. Follow `.claude/skills/council/actions/deliberate.md` to the letter. Sub-agents are spawned via `/fleet` + the `council-expert` custom agent (defined alongside this file).
