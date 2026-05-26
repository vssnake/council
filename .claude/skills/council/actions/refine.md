# ACTION · `refine`  (v2.3 — refine a closed deliberation without redoing it)

> Procedure for the `refine` action. Loaded by the core (`SKILL.md`) in its STEP 0.
> Applied TOGETHER with the core: LEAD/DIRECTOR RULES, USER ESCALATION, STRUCTURED MARKERS,
> CAPTURE DISCIPLINE, WRITING DISCIPLINE, INLINE VALIDATION, INCOMPLETE RUNS, and
> WHAT NOT TO DO live in `SKILL.md` and remain in effect here.

Invocation: `/council-refine <branch> <problem-id> [<parent-run-id>] [<child-slug>]` (Claude Code) or `copilot --agent council-refine -p "<branch> <problem-id> [<parent-run-id>] [<child-slug>]"` (Copilot CLI). The skill's umbrella `/council refine ...` also works.

**What it is.** `refine` iterates on an already-closed run (a deliberation or a previous refinement) producing a scoped **child run**, without redoing the entire deliberation. Cost is proportional to the change: the follow-up is classified into a **tier** and the tool routes.

| Tier | What it is | Cost |
|---|---|---|
| **1 · Clarification** | The answer is already in the parent run's material, just poorly exposed | ~1 spawn (moderator) |
| **2 · Refinement** | Real sub-question or new data; the parent's panel is the correct one | inherited panel + compressed rounds |
| **3 · Out of scope** | Needs another panel, or the `problem.md` no longer describes the situation | NOT `refine` — redirect |

> **Subroutines used**: this action invokes `CLOSE_ROUND(round)`, `VERIFY_OUTPUTS(expected_paths)`, `EXPERT_SPAWN_HEADER(name, role_category, lang)` and `LANGUAGE_DISCIPLINE(lang, role)`. Their definitions live in `SKILL.md` → "REUSABLE SUBROUTINES" section (always loaded). **`VERIFY_OUTPUTS` carries an INVIOLABLE rule** — read it before composing any round.

> **Active locale**: `<lang>` is sourced from `meta.yaml.lang` of the active problem (a refinement inherits its parent run's locale unless `run_meta.yaml.lang` overrides — rare). All sub-agent prompts and materialized artifacts are produced in that language. User-facing chat messages in this file are written in English for readability; the director addresses the user in `<lang>` at runtime.

STEP 1 · Validate inputs and resolve the child run

- Parse `$ARGUMENTS`. Extract `<branch>`, `<problem-id>`, and optional `<parent-run-id>` and `<child-slug>`.
- Read `branches/<branch>/problems/<id>/meta.yaml`. If missing → clear error and stop. If `status != open` → abort: "El problema no está capturado/cerrado. Usa `/council-problem-iterate <branch> <id>` primero."
- **Incomplete refinements detection** (see `INCOMPLETE RUNS` in the core): scan `council/*/` for `kind: refinement` runs without `outcome.md` (and runs without `run_meta.yaml`). If any, surface to the user and let them choose to resume / discard / start another. If they resume one, adopt its `<child-run-id>` (its `parent_run` is in `run_meta.yaml`), jump to the inferred resumption point, and skip the rest of this STEP 1.
- Resolve `<parent-run-id>` (the run to refine):
  - If NOT passed: list the runs of `branches/<branch>/problems/<id>/council/` (with their `kind` and slug/`trigger` from each `run_meta.yaml`) and ask the user which to refine.
  - If passed: validate that `council/<parent-run-id>/` exists. Indulgence: if the user typed only a slug and it's unambiguous among the runs, resolve it; if ambiguous, list and ask.
  - If there is no run in `council/` → abort: "No hay deliberaciones que refinar. Usa `/council <branch> <id> <slug>` para deliberar primero."
- Validate that the parent run is **complete**: `council/<parent-run-id>/outcome.md` exists. If not → abort: "Ese run está incompleto (sin `outcome.md`) — no hay nada que refinar."
- Read `problem.md` completely and `council/<parent-run-id>/outcome.md` completely.
- Resolve `<child-slug>`: if passed, validate (kebab-case ASCII, max 30 chars, no accents or ñ); if not, ask the user (in Spanish) — something short to distinguish this refinement.
- Compose `<child-run-id> = <YYYY-MM-DD UTC>-<child-slug>`. If `council/<child-run-id>/` already exists **and is a complete run** (has `outcome.md`) → abort and ask for another slug. An incomplete refinement with that id will already have been handled in the detection above.
- Create `branches/<branch>/problems/<id>/council/<child-run-id>/`. Write `run_meta.yaml`:
  ```yaml
  run_id: <child-run-id>
  kind: refinement
  parent_run: <parent-run-id>
  created_at: <ISO 8601>
  run_status: in_progress
  ```
  (`tier` and `trigger` are filled in STEP 3.)
- DO NOT create subdirs yet (`experts/`, `proposals/`...): they depend on the tier.

STEP 2 · Follow-up capture (`follow_up.md`)

> **HARD RULES OF THIS STEP**:
> 1. **CAPTURE DISCIPLINE**: do not add figures/brands/inferences the user did not say. Only distill and paraphrase.
> 2. **Precedence P4**: if the user's new info contradicts `problem.md`, mark it as `[actualiza el problem.md ...]`. In any conflict between `follow_up.md` and `problem.md`, **`follow_up.md` wins** — this is transmitted to experts in their prompts. `refine` NEVER rewrites `problem.md`.
> 3. **Mandatory anchor**: the follow-up must declare which section of the parent's `outcome.md` it refers to (or "general"). It drives the appendix in STEP 9.

Free mini-iterate (no formal schema). Initial question to the user (in Spanish):
> *"¿Qué quieres refinar de la deliberación `<parent-run-id>`? Cuéntame qué te falta, qué no te quedó claro, o qué dato nuevo aporta."*

Free capture, distill, validate with the user. Apply CAPTURE DISCIPLINE rigorously (do not add unsaid figures/brands, do not decode, do not interpret). Persist `council/<child-run-id>/follow_up.md` following the template `actions/refine/templates/follow_up.md.tpl`.

- The **anchor**: ask the user (or infer and confirm) which part of the parent's `outcome.md` it refers to. Drives the STEP 9 appendix.

STEP 3 · Classify the tier

Apply the **three-test cascade**, in order, on the captured follow-up:

1. **Test 1 → Tier 1?** Can the moderator answer the follow-up by re-reading the parent run's files (outcome, debate_summary, final_positions, etc.), **without new expert reasoning and without new data**? If **yes** → Tier 1.
2. **Test 2 → Tier 2?** If Test 1 fails: is the parent run's panel competent for this **and** does `problem.md` still describe the situation well? If **yes** → Tier 2.
3. **Test 3 → Tier 3.** If Test 2 fails → outside `refine`.

- **Propose the tier to the user** in plain language, with the reason ("esto lo resuelvo releyendo lo que ya hay" vs. "esto necesita que el panel piense de nuevo"). When in doubt, **round up** but explicitly offer the cheap option with its trade-off. The user confirms.
- If **Tier 3**: abort `refine`. Delete `council/<child-run-id>/` (only has `run_meta.yaml` + `follow_up.md`; never became a real run). Tell the user: if they need another panel → `/council-deliberate <branch> <id> <slug>`; if `problem.md` no longer describes the situation → `/council-iterate <branch> <id>` first.
- If **Tier 1 or 2**: update `run_meta.yaml` adding `tier:` and `trigger:` (the follow-up in one line). Then: Tier 1 → STEP 4; Tier 2 → STEP 5.

STEP 4 · Tier 1 — Clarification (moderator-only)

Only if Tier 1. No deliverable capture, no panel, no rounds. The child run stays minimal: `run_meta.yaml`, `follow_up.md`, `outcome.md`.

Spawn ONE moderator sub-agent via the spawn primitive (in Claude Code: `Task` with subagent_type=general-purpose; in Copilot CLI: invoke the `council-expert` agent with the impartial-moderator persona).

**Prompt composition**:
1. Body = `S:moderator_prompt_refine_tier1` from `locales/<lang>.yaml`, with `<branch>`, `<id>`, `<parent-run-id>`, `<child-run-id>` substituted.
2. `LANGUAGE_DISCIPLINE(lang, 'critique')` appended at the END (Tier 1 is re-exposition + framing; closest few-shot match is `critique`).

The moderator body covers: file list to read (problem.md + parent outcome + parent debate_summary if present + parent final_positions if present + child follow_up), output requirement (short bounded clarification at `outcome.md` opening with a framing line "*This clarifies <X> of `<parent-run-id>`.*"), and the **fallback rule**: if faithful response would require new reasoning not in parent files, DO NOT invent it — write `NEEDS-TIER-2.md` instead.

When done, **verify outputs (INVIOLABLE)**: run `VERIFY_OUTPUTS` with expected paths = the EITHER-OR `[council/<child-run-id>/outcome.md OR council/<child-run-id>/NEEDS-TIER-2.md]`. The moderator wrote exactly one of the two; if NEITHER exists, the spawn failed — re-spawn per the subroutine. Under no circumstance write either file yourself from the moderator's output stream.

Then:
- If `outcome.md` exists → update `run_meta.yaml` `run_status: complete` and go to STEP 9.
- If `NEEDS-TIER-2.md` exists → tell the user: "Tier 1 is not enough — this needs fresh analysis from the panel (Tier 2). Promote to Tier 2?". If the user accepts: delete `NEEDS-TIER-2.md`, update `run_meta.yaml` `tier: 2`, and go to STEP 5. If they decline: close the refinement explaining it remains unresolved.

STEP 5 · Tier 2 — Iterate the deliverable

- Read `schemas/deliverable.schema.yaml`.
- For each **required** section, resolve `locale_ref` against `locales/<lang>.yaml` → returns `{title, purpose}`. Generate skeleton `council/<child-run-id>/deliverable.md` with one `## <title>` per required section, each with `[GAP: <purpose>]`. (See SKILL.md → SCHEMA REFERENCE.)
- Mini capture loop (same mechanics as deliverable capture in `deliberate`): ask 1-3 neutral questions, persist each response, report inline validation, loop until zero `[GAP]` in required sections. `SKIP`/`N/A` only in optional.
- any-else? sweep before closing.
- A refinement's deliverable is usually different from the parent's (the parent gave a table; the refinement may give a verdict). Capture it clean — do not inherit the parent's shape by default.

STEP 6 · Tier 2 — Inherit the panel, analyze relevance, and prune

> **HARD RULES OF THIS STEP**:
> 1. **Inherited panel is prunable**: the parent's roster is the STARTING POINT, not fixed. Experts without substantial contribution to the follow-up are pruned.
> 2. **Minimum 2-3 experts** after pruning — if only 1 viewpoint remains, reconsider Tier 1.
> 3. **Maximum 1 new expert** (if the follow-up opens an uncovered angle). If several would be needed → it was Tier 3, return to STEP 3.
> 4. **Blocking checkpoint**: DO NOT proceed to Round A′ without explicit user confirmation on the refinement's panel.

The parent run's panel is the **starting point**, not a fixed datum. A refinement is narrower than its parent deliberation: it's normal that some expert from the original panel contributes nothing to this concrete follow-up.

1. Read the parent run's `panel.md` (the inherited roster: specialists + friction archetypes).
2. **Analyze the relevance** of each inherited expert against the CONCRETE follow-up of `follow_up.md` — not against the parent's broad question. For each one: *"does this expert have something substantial to say about THIS sub-question?"*.
3. **Propose the refinement's panel**: the subset of inherited experts relevant to the follow-up. Keep at least 2-3 so that the cross-critique of B′ makes sense (if truly only one viewpoint matters, reconsider whether it was Tier 1). If the follow-up opens a MINOR angle the inherited panel does not cover, you may additionally propose **one (1) new expert** — its creation is in the "Create the new expert" block at the end of this STEP.
4. **Checkpoint (always)**: present to the user the proposed panel for the refinement — which experts stay and, with concrete reason, which are pruned (in Spanish): *"para esta pregunta, `<X>` e `<Y>` no aportan — los quito"*. The user confirms or adjusts. Loop until explicit confirmation; DO NOT proceed to Round A′ without it.
5. After confirmation:
   - Copy the `persona.md` of confirmed inherited experts to `council/<child-run-id>/experts/`.
   - If the user approved a new expert, create it per the block below and write its `persona.md` to `council/<child-run-id>/experts/<name>/persona.md`.
   - Write `council/<child-run-id>/panel.md`: based on the parent's, with the refinement's expert list and, in "Notas del panel", what was inherited from `<parent-run-id>`, what was pruned and why (and the new expert, if any, with its reason).
   - Create the Tier 2 subdirs: `proposals/`, `critiques/`, `debate/`, `final_positions/`, `escalations/`.

**Create the new expert** (only if the user approved one in the checkpoint). `refine` does not reload `deliberate`'s panel design procedure; this is the minimal, self-contained procedure for ONE expert:
1. **Categorial role.** Define it as a sub-discipline of the field, NOT as the brand/stack/concrete context of the problem. Test: *"would the role be the same if the user had chosen another brand?"* If not → trim to the category.
2. **Persona from the domain.** Write its persona from the domain inventory (heuristics, red flags, anti-patterns of the field), not from `problem.md` data. Apply the core's anti-leak rule: no brands, no models, no figures, no `problem.md` decisions in the persona.
3. **Format.** Follow the SAME `persona.md` structure as the inherited personas just copied to `experts/` — same frontmatter (`nombre`, `kind`, `role_category`) and the same sections. Do not invent a different format.
4. **Naming.** kebab-case ASCII, no accents or ñ, max 30 chars, categorial (no `brand-engineer`).

A major redesign (the inherited panel barely serves, or several new experts would be needed) means it was Tier 3 → return to STEP 3.

STEP 7 · Tier 2 — Compressed rounds (adaptive)

Rounds A′ and B′ always run; C′ and D′ are conditional. Each round = fresh spawn of sub-agents via the environment's spawn primitive (see SPAWN PRIMITIVE in the core), in parallel. Escalations are a **round barrier** (see USER ESCALATION in the core).

**Parent context experts of the child read** (this is what avoids re-investigating):
- Always: `council/<parent-run-id>/outcome.md` — the only file every complete run is guaranteed to have.
- If they exist: `council/<parent-run-id>/final_positions/*.md`, `escalations/*.md`, `user_directives.md`. A parent that is a Tier 1 refinement has no `final_positions/`; a run without debate or escalations may have no `escalations/` or `user_directives.md`.
- Conditional: `council/<parent-run-id>/debate_summary.md` if the follow-up touches a point already debated (only exists if the parent had a debate round).
- They do NOT read the parent's raw `proposals/`, `critiques/`, `debate/` — intermediate material, partly corrected later; `final_positions/` and `outcome.md` already carry the refined version.
- If the parent run is itself a refinement (`kind: refinement`), walk the `parent_run` chain: experts also read the root deliberation's `outcome.md` and that of each intermediate refinement.

**Round A′ — Proposals** (always). For EACH expert, spawn fresh (via the spawn primitive) in parallel.

**Sub-agent prompt composition**:
1. `EXPERT_SPAWN_HEADER(name, role_category, lang)` (the base paths already include `follow_up.md` as anchor when the run is a refinement, and the P4 precedence rule).
2. `{{S:refine_parent_context_label}}:` + the inherited paths list:
   - `council/<parent-run-id>/outcome.md` (always)
   - From `council/<parent-run-id>/`: any of `final_positions/*.md`, `escalations/*.md`, `user_directives.md`, `debate_summary.md` that exist
   - If the parent run is itself a refinement (`kind: refinement`), walk the `parent_run` chain and also include the `outcome.md` of the root and of each intermediate link.
3. Delta resolved from `S:task_round_a` with `<output-path>` = `council/<child-run-id>/proposals/expert_<name>.md`. Note: append a single line declaring the focus is the follow-up only ("focused EXCLUSIVELY on the follow-up" — produce in `<lang>`).
4. Resolved `S:task_round_a_refine_starting_point` (the "STARTING POINT" rule for refinements).
5. `LANGUAGE_DISCIPLINE(lang, 'proposal')` appended at the END.

**Verify outputs (INVIOLABLE)**: run `VERIFY_OUTPUTS` with expected paths = `[council/<child-run-id>/proposals/expert_<name>.md for each <name> in the refinement's panel]`. Honor the hard rule — never create a missing expert file yourself.

Run `CLOSE_ROUND('a')`.

**Round B′ — Critiques** (always). Fresh spawn per expert.

**Prompt composition**:
1. `EXPERT_SPAWN_HEADER(name, role_category, lang)`.
2. `{{S:read_also_label}}:` + the paths list:
   - `council/<child-run-id>/proposals/*.md`
   - `council/<child-run-id>/escalations/round_a.md` (if it exists)
   - Inherited parent context — same list as Round A′ step 2.
3. Delta resolved from `S:task_round_b` with `<output-path>` = `council/<child-run-id>/critiques/expert_<name>.md`.
4. `LANGUAGE_DISCIPLINE(lang, 'critique')` appended at the END.

**Verify outputs (INVIOLABLE)**: run `VERIFY_OUTPUTS` with expected paths = `[council/<child-run-id>/critiques/expert_<name>.md for each <name> in the refinement's panel]`.

Run `CLOSE_ROUND('b')`.

**Conflict detection** (lead decision). Read all critiques and identify the substantial conflicts that divide the panel. Structure each as `{short id, expert-A, expert-B, exact point of disagreement}` — that shape feeds the C′ shuttle and the file names `debate/<conflict-id>__expert_<name>.md`.
- If **zero substantial conflicts**: skip C′ and D′. Tell the user (in Spanish): *"Las críticas convergen, sin conflictos — voy directo a la síntesis."* Go to STEP 8.
- If **there are conflicts**: consult the user before mediating — list the conflicts factually and ask if they have feedback/preferences. Persist any literal directive in `council/<child-run-id>/user_directives.md`. Run C′.

**Round C′ — Debate** (conditional). Shuttle diplomacy per conflict.

**Shuttle sub-agent prompt composition** (per expert, sequential):
1. `EXPERT_SPAWN_HEADER(name, role_category, lang)`.
2. `{{S:read_also_label}}:` + paths list:
   - `council/<child-run-id>/critiques/*.md`
   - `council/<child-run-id>/escalations/round_a.md`, `round_b.md` (if they exist)
   - `council/<child-run-id>/user_directives.md` (if it exists)
   - Inherited parent context.
3. Delta resolved from `S:task_round_c_shuttle` with these placeholders: `<conflict-id>`, `<conflict-description>`, `<other-expert>`, `<position-B>`, `<reasons-B>`, `<position-A>`, `<output-path>` = `council/<child-run-id>/debate/<conflict-id>__expert_<name>.md`.
4. `LANGUAGE_DISCIPLINE(lang, 'debate')` appended at the END.

Repeat 1-2 cycles per conflict.

**Verify outputs (INVIOLABLE)**: run `VERIFY_OUTPUTS` with expected paths = every `council/<child-run-id>/debate/<conflict-id>__expert_<name>.md` you spawned in the shuttle (one per turn). Honor the hard rule.

Synthesize in `council/<child-run-id>/debate_mediated.md` materializing `actions/deliberate/templates/debate_mediated.md.tpl` (same skeleton as in `deliberate`, resolved against the active locale). (Lead writes this — INVIOLABLE rule does NOT apply; the lead IS the author of `debate_mediated.md`.)

Run `CLOSE_ROUND('c')`.

**Round D′ — Final positions** (conditional — only if C′ ran). Fresh spawn per expert.

**Prompt composition**:
1. `EXPERT_SPAWN_HEADER(name, role_category, lang)`.
2. `{{S:read_also_label}}:` + paths list:
   - `council/<child-run-id>/critiques/*.md`
   - `council/<child-run-id>/debate_mediated.md`
   - `council/<child-run-id>/escalations/*.md`
   - `council/<child-run-id>/user_directives.md` (if it exists)
3. Delta resolved from `S:task_round_d` with `<output-path>` = `council/<child-run-id>/final_positions/expert_<name>.md`.
4. `LANGUAGE_DISCIPLINE(lang, 'final_position')` appended at the END.

**Verify outputs (INVIOLABLE)**: run `VERIFY_OUTPUTS` with expected paths = `[council/<child-run-id>/final_positions/expert_<name>.md for each <name> in the refinement's panel]`. Honor the hard rule — the spawn-completion notification you receive contains the sub-agent's full text, which is enough to fabricate a shape-correct file. Resist that — wait / re-spawn / stop are the only legitimate actions.

Run `CLOSE_ROUND('d')`.

STEP 8 · Tier 2 — Synthesis

> **HARD RULES OF THE REFINEMENT OUTCOME** (the moderator receives them in its prompt; the lead verifies on close):
> 1. **SCOPED**: the outcome responds ONLY to the follow-up. The parent's outcome is NOT dumped.
> 2. **AUTONOMOUS**: the reader must understand the response without re-reading the parent outcome (mandatory framing/lineage line at the start).
> 3. **DO NOT DUPLICATE** what didn't change: if the parent already covered something and this refinement does not modify it, it's not rewritten.
> 4. **`debate_summary.md`** only if Round C′ ran (there was debate). If no C′, DO NOT create it.
> 5. **IMPARTIALITY**: disagreements as symmetric positions, do not force consensus.

**8.a** · Write `council/<child-run-id>/lead_notes.md` (lead → moderator bridge) materializing `actions/deliberate/templates/lead_notes.md.tpl` (reused from `deliberate` — same skeleton, resolved against the active locale).

**8.b** · Spawn moderator sub-agent (via the spawn primitive — in Claude Code: `Task` with subagent_type=general-purpose; in Copilot CLI: invoke `council-expert` with the impartial-moderator persona).

**Prompt composition**:
1. Body = `S:moderator_prompt_refine_tier2` from `locales/<lang>.yaml`, with `<branch>`, `<id>`, `<parent-run-id>`, `<child-run-id>` substituted.
2. `LANGUAGE_DISCIPLINE(lang, 'final_position')` appended at the END.

The moderator body covers: file list to read (child run files + parent `outcome.md`), the two outputs (`outcome.md` SCOPED + SELF-CONTAINED + lineage line + conditional "Verdict on user's leaning" section if `follow_up.md` has one; `debate_summary.md` only if Round C′ ran), and IMPARTIALITY rules.

**8.c · Verify outputs (INVIOLABLE)**: run `VERIFY_OUTPUTS` with expected paths = `[council/<child-run-id>/outcome.md]` (plus `council/<child-run-id>/debate_summary.md` if Round C′ ran). Under no circumstance write the moderator's files yourself from its output stream. Update `run_meta.yaml`: `run_status: complete`.

STEP 9 · Appendix in parent and close (both tiers)

**9.a · Appendix in the parent's `outcome.md`.** The lead appends — **append-only, at the end of the file, without altering a single line of existing analysis** — a section materialized from `actions/refine/templates/parent-appendix.md.tpl` (resolved against the parent run's locale — see SKILL.md `TEMPLATE RESOLUTION`). If the parent already has a section with the same heading (i.e. `## <H:subsequent_refinements>` resolved in the parent's locale, from a previous refinement), add only the new bullet entry (not the header).

This is the only edit to a moderator file allowed for the lead (see WRITING DISCIPLINE, narrow exception).

**9.b · Close.** Report to the user in chat (in Spanish):
- "Refinamiento `<child-run-id>` (Tier <N>) completado. Resultado: `council/<child-run-id>/outcome.md`. Run padre `<parent-run-id>`: apéndice de navegación añadido."
- If there was `debate_summary.md` with open disagreements, list them for the user to decide.
