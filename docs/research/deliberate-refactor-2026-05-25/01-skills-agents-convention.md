# Report 1/4 — Skills/agents convention

> Background agent investigating best practices for Claude Code skills and Copilot CLI custom agents to inform a refactor of a long procedural file (`actions/deliberate.md`, 467 lines).

## Relevant findings

1. **Anthropic enforces a hard limit of 500 lines for SKILL.md, with escalation to sub-files** (also applies to `actions/*.md`). Quote: *"Keep SKILL.md body under 500 lines for optimal performance. Split content into separate files when approaching this limit."* — [Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices). The Claude Code runtime docs repeats it verbatim: *"Keep SKILL.md under 500 lines. Move detailed reference material to separate files."* — [Extend Claude with skills](https://code.claude.com/docs/en/skills).

2. **There is an explicitly documented anti-pattern: mixing process and context in the same file ("context rot")**. Quote: *"A common issue is cramming everything—the process steps, background context, rules, examples, and edge cases—into one skill.md file. When process steps are surrounded by paragraphs of background information, the model's attention gets fragmented and may treat context as instruction or miss procedural steps entirely."* — [Claude Code Skills Architecture (MindStudio)](https://www.mindstudio.ai/blog/claude-code-skills-architecture-skill-md-reference-files). The derived rule: *"process (the ordered steps Claude should follow) belongs in skill.md, while context (background information, rules, domain knowledge, examples, and reference data) goes elsewhere."*

3. **References should stay at one level of depth — nesting kills**. Quote: *"Keep references one level deep from SKILL.md... Claude may partially read files when they're referenced from other referenced files. When encountering nested references, Claude might use commands like `head -100` to preview content rather than reading entire files, resulting in incomplete information."* — [best-practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices). Implication: `SKILL.md → actions/deliberate.md → templates/...` is legitimate (one level), but adding a fourth level breaks the discipline.

4. **Anthropic's official skills follow that pattern strictly**. `claude-api/SKILL.md` (324 lines) is *"a navigation and decision-flow document that routes users to deeply modular, task-specific files. It embeds only summary tables, decision trees, and quick-reference callouts—not large code blocks or full prompt templates."* — [anthropics/skills/claude-api](https://github.com/anthropics/skills/blob/main/skills/claude-api/SKILL.md). `mcp-builder/SKILL.md` (236 lines) is hybrid: inline workflow + reference files loaded on-demand. `skill-creator` (485 lines) is already at the edge and delegates tests + grader + schemas to `agents/` and `references/`.

5. **The official docs acknowledges the "large workflow" case and recommends extracting it**. Verbatim quote from the `<Tip>` callout: *"If workflows become large or complicated with many steps, consider pushing them into separate files and tell Claude to read the appropriate file based on the task at hand."* — [best-practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices). This validates the core + `actions/` split and opens the door to a second split within `actions/deliberate.md`.

6. **Copilot CLI agents follow an analogous but simpler format** — markdown with frontmatter (`name`, `description`, `tools`, MCP config) in `.github/agents/*.agent.md` or `~/.copilot/agents/`. There is no documented size convention for custom agents; the official documentation only shows short examples. — [Creating custom agents for Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/create-custom-agents-for-cli).

## Patterns applicable to `deliberate.md`

1. **Prompt templates to sidecar files** (Pattern 1 + "execute the script" pattern from the docs). The ad-hoc prompts in STEP 3 (Round A — lines 236-260), STEP 4 (B), STEP 5.c (C — lines 316-329), STEP 6 (D — lines 361-371) and STEP 7.b (moderator — lines 397-456) are ~32% of the file and practically identical structurally. Extract to `actions/deliberate/prompts/{round-a,round-b,round-c-shuttle,round-d,moderator}.md` and reference from `deliberate.md` with a 3-line pointer + `${CLAUDE_SKILL_DIR}` for the sub-agent. Fits the concern: turns 32% of the file into payload the sub-agent reads, not that the director processes; `deliberate.md` is left with the orchestration decisions.

2. **Markdown skeletons to templates/ with table of contents** (Pattern "Structure longer reference files with table of contents", >100 lines). The skeletons for `panel.md` (lines 144-178), `persona.md` (181-207), `debate_mediated.md` (343-355), `lead_notes.md` (382-395) and the moderator header (440-444) are ~28% of the file. Move them to `actions/deliberate/templates/{panel.md,persona.md,debate_mediated.md,lead_notes.md}.tmpl` and reference from the STEP with `> Write following the structure of [templates/panel.md.tmpl](templates/panel.md.tmpl)`. This reduces visual noise without losing content.

3. **"Anti-leak guards" and inline disciplines → reference docs** (Pattern "Conditional details"). The guard in STEP 1.5 (lines 64-86, 23 lines) and the anti-leak rule in STEP 2 (line 142) and the moderator's SCOPE + AUDIT (lines 432-456) are reusable disciplines that can live in `actions/deliberate/disciplines/{anti-leak-hipotesis.md, anti-leak-panel.md, audit-post-sintesis.md}`. The STEP references with one line + "see disciplines/...". This matches the docs observation: *"explain to the model why things are important in lieu of heavy-handed musty MUSTs"* — explain in its file, not inline with the procedure.

4. **Repeated escalation collection → DRY procedure**. The "Collect escalations (round barrier)" block is repeated verbatim or nearly verbatim in Round A (264-281), B (297), C (357) and D (375). The meta note on line 283 *"This deduplication + multiple-attribution procedure also applies..."* confesses the redundancy. Extract to `actions/deliberate/disciplines/escalation-collection.md` with the canonical procedure (deduplication, attribution, `round_x.md` format) and each STEP ends with: *"Collect escalations according to [escalation-collection.md] in `escalations/round_a.md`"*. Reduces ~15-20 lines of duplication.

5. **Workflow checklist at the start of the file** (Pattern "Use workflows for complex tasks", copyable checklist). `deliberate.md` starts with STEP 1 without an overview. Adding a 9-step checklist at the start (`Run progress: STEP 1 validate → 1.5 hypothesis → 1.6 deliverable → 2 panel → 3 A → 4 B → 5 C → 6 D → 7 synthesis → 8 close`) helps the model know where it is and reduces the need to re-read the entire file. Cost: ~10 lines; gain: clear mental navigation as the file grows.

## Prioritized refactor options

| # | Option | Effort | Impact | Notes |
|---|--------|---------|---------|------|
| **1** | **Extract prompt templates to `actions/deliberate/prompts/*.md`** (5 files) and replace with 3-line pointers in STEPs 3/4/5c/6/7b. | **M** (~150 lines modified; reorganization; 5 new files) | **L** | Directly attacks the 32% the user identified as ad-hoc prompts. Makes each prompt diff-able without touching `deliberate.md`. Follows the canonical docs pattern ("execute, not load"). |
| **2** | **DRY "collect escalations"** into a single `disciplines/escalation-collection.md`. | **S** (~20-30 lines modified; 1 new file) | **M** | Quick win. Erases the line 283 "meta note" which is already a symptom of the problem. Low risk, high ROI. |
| **3** | **Extract markdown skeletons to `templates/*.tmpl`** (panel/persona/debate_mediated/lead_notes + moderator header). | **M** (~130 lines modified; 4-5 new files) | **M-L** | Attacks the 28% of embedded templates. Risk: the sub-agent must read 2 files (procedure + template). Mitigable by placing the explicit template path in the sub-agent's prompt. |
| **4** | **Long disciplines (anti-leak, audit, SCOPE rule) to `disciplines/*.md`** + add progress checklist at the start. | **L** (major refactor; 3-4 new files; review of all STEPs) | **L** | Closes the loop: `deliberate.md` ends up ~150-200 lines of pure procedure. But it's where regressions are easiest to introduce — recommended to do it last, after 1+2+3, with regression tests (deliberate an old problem and compare outcome). |

**Recommended order: 2 → 1 → 3 → 4.** Lets you validate the approach with the quick win (#2), attack the large chunk (#1), reduce visual noise (#3), and close with the surgical operation (#4) once you have the extraction mechanics tuned.

## Anti-patterns to avoid

1. **Deep nesting of references.** *"Bad example: Too deep — SKILL.md → advanced.md → details.md"* — [best-practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices). If you extract `actions/deliberate/prompts/round-a.md`, that file must NOT in turn delegate to `prompts/shared/header.md`. Each extraction is direct from `deliberate.md`, a single level.

2. **Too many options / unjustified "voodoo".** *"Don't present multiple approaches unless necessary"* — [best-practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices). When extracting prompts, don't fall into "v1 / v2 / variant for deep-research" versions of the same prompt. One canonical prompt per round; variations are handled by params in the prompt, not by files.

3. **Overloading SKILL.md/action with narrative about the why.** *"Only add context Claude doesn't already have. Challenge each piece of information: 'Does Claude really need this explanation?'"* — [best-practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices). The 5% of prose the user identified: a candidate for aggressive trimming. The "why" of a rule belongs in its `disciplines/<rule>.md` when it helps the model generalize; if it's only justification for humans, out it goes.

## Uncertainty note

There is no officially documented convention for the maximum size of `actions/*.md` (the 500 lines refer to SKILL.md). However, the same arguments (fragmented attention, context rot) apply once the file is loaded into context: treating `deliberate.md` with the same limit is a reasonable extension. There is also no officially documented convention for Copilot CLI custom agents — the docs are limited to frontmatter syntax; sizing is the author's responsibility.

## Sources

- [Skill authoring best practices — platform.claude.com](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- [Extend Claude with skills — code.claude.com](https://code.claude.com/docs/en/skills)
- [anthropics/skills (official repo)](https://github.com/anthropics/skills)
- [skill-creator/SKILL.md](https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md)
- [claude-api/SKILL.md](https://github.com/anthropics/skills/blob/main/skills/claude-api/SKILL.md)
- [mcp-builder/SKILL.md](https://github.com/anthropics/skills/blob/main/skills/mcp-builder/SKILL.md)
- [Claude Code Skills Architecture: Why Your skill.md File Should Only Contain Process Steps — MindStudio](https://www.mindstudio.ai/blog/claude-code-skills-architecture-skill-md-reference-files)
- [What Is Context Rot in Claude Code Skills? — MindStudio](https://www.mindstudio.ai/blog/context-rot-claude-code-skills-bloated-files)
- [Creating and using custom agents for GitHub Copilot CLI — GitHub Docs](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/create-custom-agents-for-cli)
- [Custom agents configuration — GitHub Docs](https://docs.github.com/en/copilot/reference/custom-agents-configuration)
