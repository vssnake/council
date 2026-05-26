# ACTION · `deliberate`  (v2.3 — fresh spawn primitive per round, no peer-to-peer, synthesis by moderator)

> Procedure for the `deliberate` action. Loaded by the core (`SKILL.md`) in its STEP 0.
> Applied TOGETHER with the core: SPAWN PRIMITIVE, LEAD/DIRECTOR RULES, USER ESCALATION, STRUCTURED MARKERS,
> CAPTURE DISCIPLINE, WRITING DISCIPLINE, INLINE VALIDATION, INCOMPLETE RUNS, and
> WHAT NOT TO DO live in `SKILL.md` and remain in effect here.

Invocation: `/council-deliberate <branch> <problem-id> [<slug>]` (Claude Code) or `copilot --agent council-deliberate -p "<branch> <problem-id> [<slug>]"` (Copilot CLI). The skill's umbrella `/council deliberate ...` also works.

**Coordination model**: no teammate persists across rounds. Each round spawns N fresh sub-agents via the **environment's spawn primitive** (see SPAWN PRIMITIVE in the core: `Task` tool in Claude Code, `council-expert` custom agent via `/fleet` in Copilot CLI). Experts NEVER talk to each other — the lead is the only channel. User escalations are a **round barrier**: experts annotate them in their file and the lead batches them to the user at each round's close.

> **Subroutines used**: this action invokes `CLOSE_ROUND(round)`, `EXPERT_SPAWN_HEADER(name, role_category, lang)` and `LANGUAGE_DISCIPLINE(lang, role)`. Their definitions live in `SKILL.md` → "REUSABLE SUBROUTINES" section (always loaded). Each reference like *"Run `CLOSE_ROUND('a')`"* is equivalent to inlining the procedure described there.

> **Active locale**: throughout this procedure, `<lang>` is sourced from `meta.yaml.lang` of the active problem (see SKILL.md DATA MODEL + TEMPLATE RESOLUTION). All sub-agent prompts and all materialized artifacts are produced in that language. User-facing chat messages in this file are written in English for readability; the director addresses the user in `<lang>` at runtime.

STEP 1 · Validate inputs and resolve run-id

- Parse `$ARGUMENTS`. Extract `<branch>`, `<problem-id>`, and optional `<slug>`.
- Read `branches/<branch>/problems/<id>/meta.yaml`. If `status != open`: abort, telling the user (in Spanish): "El problema debe estar `status: open` antes de deliberar. Usa `/council-problem-iterate <branch> <id>` para completarlo."
- Read `problem.md` completely.
- **Incomplete runs detection** (see `INCOMPLETE RUNS` in the core): scan `branches/<branch>/problems/<id>/council/*/` for deliberations without `outcome.md`. If any, surface to the user and let them choose to resume / discard / start another. If they resume one, adopt its `<run-id>`, jump to the inferred resumption point, and skip the rest of this STEP 1.
- Resolve `<slug>` (**blocking — do not proceed to STEP 1.5 without slug**):
  - If passed via CLI: use it (validate kebab-case ASCII, max 30 chars, no accents or ñ).
  - If NOT: ask the user (in Spanish) — *"¿Slug para esta deliberación? (ej: `dimensionado`, `comparativa-bd`, `revisar-hipotesis`). Algo corto que recuerdes para distinguirla de futuras deliberaciones sobre el mismo problema."* **WAIT FOR USER INPUT.** DO NOT infer a slug from `problem.md`, do not reuse the problem-id, do not advance with a placeholder. The slug is human input; without it the `<run-id>` cannot be constructed.
- Compose `<run-id> = <YYYY-MM-DD UTC>-<slug>` (e.g.: `2026-05-12-dimensionado`).
- If `branches/<branch>/problems/<id>/council/<run-id>/` already exists **and is a complete run** (has `outcome.md`): abort and suggest another slug. An incomplete run with that id will already have been handled in the detection above.
- Create `branches/<branch>/problems/<id>/council/<run-id>/` with subdirs: `experts/`, `proposals/`, `critiques/`, `debate/`, `final_positions/`, `escalations/`.
- Write `council/<run-id>/run_meta.yaml` with `kind: deliberation` and `run_status: in_progress` (see DATA MODEL in the core).

STEP 1.5 · Iterate hypothesis (skippable)

> **HARD RULE — anti-leak**: the distillation NEVER adds figures, metrics, analytical reformulations, or inferred consequences that the user did not LITERALLY say in their hypothesis. Full detail in the "Anti-leak guard" block below, with good/bad examples from spike v2.

Ask the user (in Spanish) in a single line:
> *"¿Quieres aportar tú la hipótesis tentativa (lo que ya tienes en mente como solución), o prefieres que la deduzca yo del problem.md y la valides? También puedes declarar caso abierto si quieres divergencia total."*

Accept three responses:

**(a) "Yo la aporto" / "aportar"** (user provides):
- Mini-iterate. Initial question (in Spanish): *"Cuéntame qué solución tentativa vienes barajando para esto."*
- Free capture, distill to 2-5 points, validate with the user, persist to `council/<run-id>/hypothesis.md`:
  ```markdown
  # Hipótesis tentativa — <branch>/<problem-id> · <run-id>

  [user-provided]

  ## Hipótesis
  <2-5 distilled points>
  ```
- Apply CAPTURE DISCIPLINE (do not add brands/figures not said, do not decode).

**(b) "Auto-generar" / "deduce tú"** (auto-generate):
- Read `problem.md` (already done in STEP 1). Distill what you see as implicit hypothesis: decisions marked as taken + hypotheses marked as "to evaluate" in `decisiones_previas`. If there's nothing like that, propose moving to (c).
- Present to user (in Spanish): *"La hipótesis que veo en tu problem.md: <2-5 puntos>. ¿Confirmas, editas, o reemplazas?"*
- If confirms: persist with marker `[auto-generated · validated by user]`.
- If edits: apply changes and persist with `[auto-generated · edited by user]`.
- If replaces completely: branch to (a).

**(c) "Caso abierto" / "sin hipótesis"** (open case):
- Persist:
  ```markdown
  # Hipótesis tentativa — <branch>/<problem-id> · <run-id>

  [N/A: caso abierto]

  El user explícitamente declinó aportar hipótesis. El consejo debe divergir
  desde el problem.md sin anchor.
  ```

**Anti-leak guard (critical — applies to branches (a) and (b))**

The hypothesis distillation must be **a more structured paraphrase of the user's literal text**, NOT an enrichment. Before presenting the distillation to the user for validation, audit each point against this list:

- ❌ **DO NOT** add numbers, figures, units, or ranges not in the text the user said *in hypothesis*. Even if the figure is in `problem.md`, it does NOT enter `hypothesis.md` if the user did not say it here.
- ❌ **DO NOT** introduce analytical reformulations or derived metrics (e.g., user says "low price" → you DO NOT write "€/Wp as relevant metric"; user says "more watts" → you DO NOT write "maximize nominal power per unit").
- ❌ **DO NOT** add inferred consequences (e.g., "→ minimize # of panels", "→ prioritize X format"). Those are director's hypotheses, not the user's.
- ❌ **DO NOT** mix dimensions the user separated. If the user said one thing in hypothesis, the distillation has at most one entry — do not inflate to 3 points to "look complete".
- ✅ **DO** paraphrase, normalize colloquial terms to neutral vocabulary, remove filler words, and group repetitions of the same point.

**Bad example** (leak observed in spike v2):
- User says (literal in hypothesis): *"preferencia por los paneles con más watts"*.
- Faulty distillation:
  1. *Priorizar paneles de mayor potencia por unidad (W/panel)* ← OK, paraphrase.
  2. *Minimizar número de paneles necesarios para alcanzar 7 kW* ← ❌ figure leak (`7 kW`) from `problem.md` + inferred consequence.
  3. *El precio por vatio es el indicador de eficiencia económica relevante* ← ❌ analytical reformulation not said by the user.

**Good example**:
- User says: *"preferencia por los paneles con más watts"*.
- Correct distillation:
  1. *Priorizar paneles con mayor potencia por unidad.*

If after the audit the distillation stays at 1 point, it stays at 1 point. A short faithful distillation beats a long contaminated one.

After persisting `hypothesis.md`, advance to STEP 1.6.

STEP 1.6 · Iterate deliverable (mandatory)

NOT skippable. Without it, `outcome.md` will come out shaped randomly.

- Read `schemas/deliverable.schema.yaml`.
- For each **required** section, resolve `locale_ref` against `locales/<lang>.yaml` → returns `{title, purpose}`. Generate skeleton `council/<run-id>/deliverable.md` with one `## <title>` per required section, each with `[GAP: <purpose>]`. Optional sections are NOT created in skeleton. (See SKILL.md → SCHEMA REFERENCE.)
- Mini capture loop (shorter than `iterate` of `problem.md`):
  1. Read `deliverable.md` from disk.
  2. Identify open `[GAP]` in required sections.
  3. Ask 1-3 neutral questions (without explaining why).
  4. Receive response. `SKIP`/`N/A` valid **only** in optional sections; in required, the user must provide content or rephrase.
  5. Persist response. Apply CAPTURE DISCIPLINE.
  6. Report inline validation (count gaps).
  7. Loop until zero `[GAP]` in required sections.
- Before closing, run the any-else? sweep (ask in Spanish): *"¿Hay algo del entregable que no te haya preguntado y pueda importar (ej. condiciones de formato, restricciones de longitud, tono, qué excluir explícitamente)?"*
- When closed, persist and advance to STEP 2.

STEP 2 · Design the panel — **delegated to `actions/deliberate/panel-design.md`**

> **HARD RULES OF THIS STEP** (summary — detail and instrumentation in `panel-design.md`):
> 1. **Anti-leak**: `panel.md` and personas DO NOT replicate `problem.md` data.
> 2. **Naming**: specialists = `<role>-<domain>`; friction = `arquetipo-` or `<failure-mode>-<dimension>`.
> 3. **Cap**: 4-6 specialists + 0-2 friction archetypes, combined ≤ 7.
> 4. **Blocking checkpoint**: DO NOT proceed to STEP 3 without explicit user confirmation.

Read `actions/deliberate/panel-design.md` and execute its procedure (3 PASS + naming + checkpoint loop). Return here with `panel.md` + `experts/*/persona.md` written and user-confirmed, and continue with STEP 3.

STEP 3 · Round A — Proposals (spawn primitive, fresh spawn per expert, in parallel)

For EACH expert in the panel, spawn a sub-agent via the environment's spawn primitive (see SPAWN PRIMITIVE in the core). **Launch them all in parallel** — in Claude Code: multiple `Task` calls in a single lead message; in Copilot CLI: a `/fleet` that dispatches N invocations of the `council-expert` agent in parallel.

**Sub-agent prompt composition** (the director assembles these parts in order):
1. `EXPERT_SPAWN_HEADER(name, role_category, lang)` — resolved against the active locale.
2. Delta resolved from the locale pack `S:task_round_a`, with `<output-path>` substituted to `branches/<branch>/problems/<id>/council/<run-id>/proposals/expert_<name>.md`.
3. `LANGUAGE_DISCIPLINE(lang, 'proposal')` — appended at the END (resolution includes `F:example_proposal` few-shot).

Launch all experts in parallel. Wait for all to finish.

**Verify outputs (INVIOLABLE)**: run `VERIFY_OUTPUTS` with the expected paths = `[proposals/expert_<name>.md for each <name> in panel.md]`. Honor its hard rule — under no circumstance create a missing expert file yourself.

Run `CLOSE_ROUND('a')`.

STEP 4 · Round B — Cross critiques (spawn primitive, fresh spawn per expert, in parallel)

**Sub-agent prompt composition**:
1. `EXPERT_SPAWN_HEADER(name, role_category, lang)`.
2. `{{S:read_also_label}}:` + the paths list:
   - `council/<run-id>/proposals/*.md` (all proposals from the other experts)
   - `council/<run-id>/escalations/round_a.md` (user Q&A from Round A, if it exists)
3. Delta resolved from `S:task_round_b` with `<output-path>` = `council/<run-id>/critiques/expert_<name>.md`.
4. `LANGUAGE_DISCIPLINE(lang, 'critique')` appended at the END.

Launch all in parallel, wait for all to finish.

**Verify outputs (INVIOLABLE)**: run `VERIFY_OUTPUTS` with expected paths = `[critiques/expert_<name>.md for each <name> in panel.md]`.

Run `CLOSE_ROUND('b')`.

STEP 5 · Round C — Lead-mediated debate (shuttle diplomacy, no peer-to-peer)

The lead (you) operates as mediator. There is NO peer-to-peer debate between experts.

**5.a · Identify conflicts**. Read all critiques. Identify the 3-6 main conflicts that divide the panel. For each conflict:
- Conflict ID (short slug, e.g. `bd-influx-vs-timescale`).
- Which expert holds position A?
- Which expert holds position B?
- What is the exact point of disagreement?

**5.b · Consult the user (before mediating)**. Ask the user in main chat (in Spanish):
> *"Antes de mediar el debate, ¿tienes feedback, restricciones adicionales o preferencias sobre estos conflictos? [lista factual]. Si no, prosigo con la mediación."*

Persist any user directive in `council/<run-id>/user_directives.md` (user's literal text, without amplifying or adding options). If no directives, do not create the file.

**5.c · Shuttle diplomacy per conflict**. For each conflict, in order:

1. Spawn sub-agent targeted at expert A. **Prompt composition**:
   1. `EXPERT_SPAWN_HEADER(name_A, role_category_A, lang)`.
   2. `{{S:read_also_label}}:` + the paths list:
      - `council/<run-id>/critiques/*.md`
      - `council/<run-id>/escalations/round_a.md`, `round_b.md` (if they exist)
      - `council/<run-id>/user_directives.md` (if it exists)
   3. Delta resolved from `S:task_round_c_shuttle` with these placeholders substituted: `<conflict-id>` (short slug), `<conflict-description>` (factual), `<other-expert>` (the opposing expert's name), `<position-B>` (their position), `<reasons-B>` (their reasoning from their critique), `<position-A>` (A's position as the lead understood it), `<output-path>` = `council/<run-id>/debate/<conflict-id>__expert_<name>.md`.
   4. `LANGUAGE_DISCIPLINE(lang, 'debate')` appended at the END.

2. Wait for the sub-agent to finish. Read the response.

3. Spawn sub-agent targeted at expert B with the same composition, injecting A's updated position into the `<position-B>`/`<reasons-B>` placeholders (now B is being asked to respond to A).

4. Repeat N cycles per conflict (typically 1-2). **Stopping conditions** (any one suffices):
   - An expert clearly concedes.
   - Both hold firm without new arguments (open disagreement — annotate as such).
   - They reach an explicit compromise.

**5.d · Verify debate outputs (INVIOLABLE)**: run `VERIFY_OUTPUTS` with expected paths = the `debate/<conflict-id>__expert_<name>.md` files for every shuttle turn you spawned (one per expert per conflict, both A and B sides). Honor the hard rule.

**5.e · Synthesize the mediated debate**. Write `council/<run-id>/debate_mediated.md` materializing `actions/deliberate/templates/debate_mediated.md.tpl` against the active locale (see SKILL.md TEMPLATE RESOLUTION). (Lead writes this — INVIOLABLE rule does NOT apply here; the lead IS the author of `debate_mediated.md`.)

**5.f · Collect escalations**. If experts added a `## {{H:questions_for_user}}` section in their `debate/` files: run `CLOSE_ROUND('c')`.

STEP 6 · Round D — Final positions (spawn primitive, fresh spawn per expert, in parallel)

**Sub-agent prompt composition**:
1. `EXPERT_SPAWN_HEADER(name, role_category, lang)`.
2. `{{S:read_also_label}}:` + the paths list:
   - `council/<run-id>/critiques/*.md`
   - `council/<run-id>/debate_mediated.md`
   - `council/<run-id>/escalations/*.md` (all previous rounds)
   - `council/<run-id>/user_directives.md` (if it exists)
3. Delta resolved from `S:task_round_d` with `<output-path>` = `council/<run-id>/final_positions/expert_<name>.md`.
4. `LANGUAGE_DISCIPLINE(lang, 'final_position')` appended at the END.

Launch all in parallel, wait for all to finish.

**Verify outputs (INVIOLABLE)**: run `VERIFY_OUTPUTS` with expected paths = `[final_positions/expert_<name>.md for each <name> in panel.md]`. Honor the hard rule — even when you can see the sub-agent's output text in the completion notification, **do not create the file yourself** if it is missing on disk. The race-condition between "completed" and "file present" is real (~tens of seconds gap observed in Copilot CLI `/fleet`) — let VERIFY_OUTPUTS handle the retry / re-spawn / stop logic.

Run `CLOSE_ROUND('d')`.

STEP 7 · Synthesis (impartial moderator via spawn primitive, with `lead_notes.md` bridge)

> **HARD RULES OF THE OUTCOME** (the moderator receives them in its prompt; the lead verifies on close):
> 1. **SCOPE**: `outcome.md` respects EXACTLY the sections declared in `deliverable.md`. Out-of-scope content touched by some expert is DISCARDED — even if useful.
> 2. **IMPARTIALITY**: disagreements reflected as symmetric positions. DO NOT force consensus. Cite by category, not by brand.
> 3. **POST-SYNTHESIS AUDIT** (mandatory): 1:1 section→deliverable mapping + evidence verification per concrete figure.
>
> The moderator prompt (section 7.b) instruments these rules.

**7.a · Write `council/<run-id>/lead_notes.md`** (lead → moderator bridge) materializing `actions/deliberate/templates/lead_notes.md.tpl` against the active locale.

**7.b · Spawn moderator sub-agent** (via the spawn primitive — in Claude Code: `Task` with subagent_type=general-purpose; in Copilot CLI: invoke the `council-expert` agent with the impartial-moderator persona).

**Prompt composition**:
1. Body = `S:moderator_prompt_deliberate` from `locales/<lang>.yaml`, with `<branch>`, `<id>`, `<run-id>` substituted.
2. `LANGUAGE_DISCIPLINE(lang, 'final_position')` appended at the END (the moderator produces consolidated narrative — closest few-shot match is `final_position`).

The moderator body covers: file list to read, the two output files (`debate_summary.md` materialized from the locale-resolved template + `outcome.md` shaped per `deliverable.md`), the SCOPE hard rule (outcome respects `deliverable.md` sections only, out-of-scope content discarded), the conditional "Verdict on user's hypothesis" section (mandatory if `hypothesis.md` marked `[user-provided]` or `[auto-generated]`; omitted on `[N/A: open case]`), IMPARTIALITY rules (symmetric positions, cite by category), and the mandatory POST-SYNTHESIS AUDIT (section→deliverable mapping + evidence discipline per concrete figure).

Wait for the moderator to finish. **Verify outputs (INVIOLABLE)**: run `VERIFY_OUTPUTS` with expected paths = `[council/<run-id>/debate_summary.md, council/<run-id>/outcome.md]`. If missing → wait / re-spawn / stop, per the subroutine. Under no circumstance write the moderator's files yourself from the spawn's output stream or from prior-round material.

**7.c · Update `run_meta.yaml`**: `run_status: complete`. (The problem's `meta.yaml` is NOT touched — `status` remains `open`.)

STEP 8 · Close

- Report to the user in chat (in Spanish):
  - "Council `<run-id>` completado. Resumen del debate: `council/<run-id>/debate_summary.md`. Recomendación: `council/<run-id>/outcome.md`."
  - If there are open points for the user (from the "Desacuerdos abiertos" section of `debate_summary.md`), list them in chat for the user to decide how to proceed.
  - If `outcome.md` seems to have left any point not cleanly closed, remind the user they can refine it with `/council-refine <branch> <problem-id> <run-id>` without redoing the whole deliberation.
