# Report 3/4 — Prompt engineering for long procedures

> Background agent researching documented best practices (Anthropic docs + papers + community) for dense LLM procedures (>300 lines) and metrics for "rule density".

## Relevant findings

1. **[Anthropic official]** Skills use **3-level progressive disclosure**: metadata always loaded, SKILL.md loaded on-demand, linked files loaded only if Claude asks for them. *"Keep SKILL.md body under 500 lines for optimal performance. If your content exceeds this, split it into separate files."* — [Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices). The `deliberate.md` at 467 lines is right at the limit.

2. **[Anthropic official]** Context is a **public good**; every token competes with conversation history. *"The context window is a public good. […] every token competes with conversation history and other context."* The operational heuristic: *"Does Claude really need this explanation? Can I assume Claude knows this?"* — [same link above].

3. **[Anthropic official]** XML > Markdown for Claude in mixed prompts (instructions + context + examples). Anthropic recommends: *"Wrap each type of content in its own tag […] to reduce misinterpretation"* and reports that *"structured XML prompts produce 20-40% more consistent outputs"* — [Use XML tags to structure your prompts](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/use-xml-tags). **However**: if a section is a single sentence, it doesn't deserve a tag.

4. **[Academia / Lost-in-the-middle]** Instructions mid-instruction receive less attention (U-shape). *"Performance is often highest when relevant information occurs at the beginning or end of the input context."* — [Lost in the Middle (arXiv:2307.03172)](https://arxiv.org/abs/2307.03172). Applicable to rules, not just retrieval: critical rules buried in the middle of a long STEP get ignored more.

5. **[Academia]** In multi-constraint instruction following, **"hard-to-easy" ordering wins**: presenting the hardest constraints first improves consistency across models and sizes. — [Order Matters (arXiv:2502.17204)](https://arxiv.org/abs/2502.17204).

6. **[Academia / industry]** **Prompt chaining > single long prompt** ~20% on complex tasks because each step isolates attention. *"When the model receives a single prompt containing multiple instructions, it has to juggle all of them simultaneously and errors compound."* — [Stepwise vs chain (arXiv:2406.00507)](https://arxiv.org/abs/2406.00507) + [PromptHub guide](https://www.prompthub.us/blog/prompt-chaining-guide). The council model of spawn primitive per round **already applies this**; the problem remains inside the director's file.

7. **[Community / DEV]** **"Instruction stacking"** documented as anti-pattern: after ~8-10 distinct rules, attention per rule drops measurably and the model *"triages which rules to follow"*. — [Prompt Engineering Anti-Patterns](https://www.digitalapplied.com/blog/prompt-engineering-anti-patterns-10-mistakes-2026).

## Patterns applicable to `deliberate.md`

1. **Extract embedded templates to `templates/`** (impact: -28% lines). The 4 large markdown templates (`panel.md` skeleton, `persona.md` skeleton, `outcome.md` header, `debate_summary.md`) are *output* content, not instructions to the director. Replace with a single line: *"Write `panel.md` following `actions/deliberate/templates/panel.md.tpl`"*. Applies to STEP 2 and STEP 7.b. Direct progressive disclosure pattern from Anthropic.

2. **Extract ad-hoc sub-agent prompts to `actions/deliberate/prompts/`** (impact: -32% lines). The inline prompts for Rounds A/B/C/D are **the output the director composes**, not the procedure it executes. Move to `prompts/round_a.md.tpl`, `round_b.md.tpl`, etc. with placeholders `<branch>`, `<run-id>`, `<name>`. STEPs 3-6 reduce to 3-4 lines each: *"Compose prompt from `prompts/round_a.md.tpl`, launch N sub-agents in parallel, verify outputs."* This **isolates the director's attention on the procedure**, not on the literal content of the prompt to the expert.

3. **Promote the `5.c` block and the "ANTI-LEAK guard" to a visible section** (impact: critical rule density). The hard rules of STEP 1.5 anti-leak (lines 64-86) and the "ANTI-LEAK RULE" of STEP 2 (line 142) are buried. Move the hard rules to a block at the start of each STEP (or to a sidecar `actions/deliberate/rules.md` always referenced) and leave **only the procedure** in the linear flow. Mitigates lost-in-the-middle.

4. **Reorder to "hard-to-easy" within each STEP** (impact: compliance). STEP opening: *hard constraints (DO NOT DO)* → *procedure* → *optional examples*. Currently `deliberate.md` has mid-page examples (lines 74-86) between the rule and the rest of the procedure. Applies the recommendation from arXiv:2502.17204.

5. **Centralize an expert prompt header** (DRY). Rounds A, B, C, D repeat the block *"You are `<name>` ... DO NOT communicate with other experts ... Language: Spanish"*. Factor into a fragment `prompts/_expert_header.md.tpl` referenced from each `round_*.md.tpl`. Reduces ~40 lines and eliminates drift between rounds.

## Metrics / heuristics for "rule density"

- **Rules/line ratio**: count lines containing a hard rule marker (`NO`, `DEBE`, `NUNCA`, `OBLIGATORIO`, `REGLA`, `❌`, `✅`, `MUST`) over total lines. Currently in `deliberate.md`: ~13% (~60 / 467). Post-refactor target: >30% of what remains.
- **Time-to-rule**: with a stopwatch, open the file and count seconds until finding 3 specific hard rules (e.g. anti-leak, no peer-to-peer, no synthesizing). If >30s the density is low.
- **Lines-between-rules**: max gap between two consecutive hard rules. Currently there are stretches of ~60 lines (all of STEP 7 from line 396 to 451) without an isolable hard rule — that's "lost in the middle" territory.
- **Output-payload size / procedure-payload size**: currently ~60/40. Target after extracting templates+prompts: ~20/80 (mostly procedure).

## Prioritized refactor options

| # | Option | Effort | Impact |
|---|--------|--------|--------|
| 1 | **Extract output templates to `templates/*.tpl`** | Low (mechanical) | High (-130 lines, separates output from instruction) |
| 2 | **Extract sub-agent prompts to `prompts/*.tpl` + common header** | Medium | Very high (-150 lines, DRY, director's attention focused) |
| 3 | **Refactor internal ordering of each STEP to `[hard rules] → [procedure] → [examples]`** | Medium (rewrite) | High (mitigates lost-in-the-middle + hard-to-easy bias) |
| 4 | **Sidecar `rules.md` for cross-cutting critical rules** of deliberate, referenced from each STEP | Low | Medium (controlled redundancy — Anthropic accepts *"redundancy on the contract"* when it reduces errors) |

Recommendation: combine 1+2 as mechanical baseline, then 3 as a quality pass. Use 4 only if after 1-3 the rules remain diluted.

## Confirmed anti-patterns

1. **Instruction stacking** — *"Past about eight to ten distinct instructions, attention to any one of them measurably drops […] the model starts triaging which rules to follow rather than following them all."* — [DigitalApplied](https://www.digitalapplied.com/blog/prompt-engineering-anti-patterns-10-mistakes-2026). `deliberate.md` has ~20 hard rules scattered — clearly in triage territory.

2. **Mixing instructions with context/data** — *"When behavioral rules are mixed with task-specific context in one string, the model treats everything with equal weight and starts dropping instructions."* — same link. The file mixes *(a)* instructions to the director, *(b)* literal text of the prompt to the expert, and *(c)* output skeleton. Separating the three is the cure.

3. **Deeply nested references** — *"Claude may partially read files when they're referenced from other referenced files […] resulting in incomplete information. Keep references one level deep from SKILL.md."* — [Anthropic Skill best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices). Important: if you extract to `templates/` and `prompts/`, keep the reference **one level only** from `deliberate.md`, do not chain.

## Honesty caveat

- The "Order Matters" paper (hard-to-easy) is measured in classic multi-constraint instruction following (several constraints in a simple task), not specifically in procedural skills of 400+ lines. The transfer is plausible but not proven for this case.
- The "8-10 instructions" number is a community heuristic, not a peer-reviewed paper metric.
- Anthropic does not publish official "rule density" metrics — the proposal to measure it is derived, not official.
- The "<500 lines for SKILL.md" target refers to the root SKILL.md; Anthropic does not set an explicit limit for action files under `references/` or equivalent. `deliberate.md` plays a hybrid role (it is a reference of SKILL.md, but also a mandatory procedure) — the target still applies by analogy but not by rule.

## Sources

- [Anthropic — Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- [Anthropic — Use XML tags to structure your prompts](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/use-xml-tags)
- [Anthropic — Prompting best practices (Opus 4.7)](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)
- [Anthropic — Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [arXiv 2307.03172 — Lost in the Middle](https://arxiv.org/abs/2307.03172)
- [arXiv 2502.17204 — Order Matters (multi-constraint position bias)](https://arxiv.org/abs/2502.17204)
- [arXiv 2406.00507 — Prompt Chaining vs Stepwise refinement](https://arxiv.org/abs/2406.00507)
- [DigitalApplied — Prompt Engineering Anti-Patterns 2026](https://www.digitalapplied.com/blog/prompt-engineering-anti-patterns-10-mistakes-2026)
- [Codastra — Prompt Hygiene for Engineers](https://medium.com/@2nick2patel2/prompt-hygiene-for-engineers-edc4cabdbc28)
- [PromptHub — Prompt Chaining Guide](https://www.prompthub.us/blog/prompt-chaining-guide)
