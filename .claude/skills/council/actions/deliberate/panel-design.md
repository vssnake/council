# Panel design — STEP 2 procedure of `deliberate`

> This file is loaded by `actions/deliberate.md` in its STEP 2 (one-level-deep reference). NOT a sub-skill — it's an additional procedure file under the same skill entry. Return to `deliberate.md` with `panel.md` + `experts/*/persona.md` written and user-confirmed.

## HARD RULES OF THIS STEP

1. **Anti-leak**: `panel.md` and personas DO NOT replicate `problem.md` data (brands, models, figures, locations, stack). Only categorial domain vocabulary.
2. **Naming**: specialists = `<role>-<domain>`; friction archetypes = prefix `arquetipo-` or `<failure-mode>-<dimension>` (NEVER a functional role noun like `consultor-X`).
3. **Cap**: 4-6 specialists + 0-2 friction archetypes, combined max of 7.
4. **Blocking checkpoint**: DO NOT persist `panel.md`/`persona.md` files to disk and DO NOT proceed to STEP 3 without explicit user confirmation on the panel.

## Three-pass discipline (internal, mental — nothing on disk)

The 3 PASS happen entirely in your head / scratch context. Nothing is written to disk until the user confirms the panel in the checkpoint below. This makes the adjustment loop cheap: tweaks before persistence = no files to delete.

1. **PASS 1 — Selection**. Read `problem.md`. List the roles needed in purely **categorial** terms:
   - ✅ "engineering of [field sub-discipline]"
   - ❌ "engineering with [brand-from-input]+[model-from-input]"
   - Mental test per role: "would this role have the same identity if the user had chosen another brand/stack/context?" If not → contaminated by leak.
   - Internal output: list of `[role_id, role_category, categorial angle]`.

2. **PASS 2 — Persona drafting (mentally, no file write)**. WITHOUT re-reading `problem.md`, draft the persona of each role from:
   - The panel's DOMAIN (not the problem's)
   - role_category + angle from PASS 1
   - The implicit inventory of the field's specialty
   - DO NOT (deliberately) access brands/models/figures/reference persons/open hypotheses from `problem.md`.

   Hold the draft in working memory. Materialization to disk happens later, after the checkpoint.

3. **PASS 3 — Validation**. Re-read `problem.md` and verify that each decision to make has at least one specialist covering it **by category** (not by brand). If coverage is missing → return to PASS 1.

## Optional friction archetypes (0-2)

If the domain surfaces non-technical failure modes, add archetypes in dimensions:
- `operacional`: failures in real use, maintenance, edge conditions.
- `adherencia`: humans don't sustain perfect plans long-term.
- `politico`: stakeholders, committees, scope creep, adversarial defense.
- `economico`: TCO vs upfront, hidden costs, lifecycle.

Don't force all 4. If the domain doesn't surface them, omit them.

## Naming in detail

(kebab-case, ASCII, no accents or ñ, max 30 chars, categorial — not `pylontech-engineer`). The name auto-prompts the persona, so each `kind` has its template:

- **Specialists** → `<role>-<bounded-domain>`. Role noun + domain of the field. Examples: `tecnologo-modulos-fotovoltaicos`, `analista-mercado-energia`, `economista-coste-fotovoltaico`. The role is constructive ("proposes, calculates, dimensions, analyzes").
- **Friction archetypes** → prefix `arquetipo-` **or** `<failure-mode>-<dimension>`. NEVER a functional role noun (avoid `consultor-X`, `asesor-X`, `experto-en-Y` for a friction — those names drift the model toward "I propose" instead of "I poke"). Valid examples: `arquetipo-esceptico-operacional`, `adversario-presupuestario`, `disidente-politico`, `escrutador-adherencia`. The name declares the **failure mode** the archetype embodies, not the function it performs.

## Anti-leak rule in detail

`panel.md` and personas DO NOT replicate `problem.md` data. Concrete brands, models, locations, figures, decisions made, stakeholders, stack — all that lives in `problem.md`. The council will read `problem.md` directly; you don't need to copy it. Domain vocabulary (technical language of the field) yes — that's not leak. Mental test per term: "is this here because it belongs to the domain in general, or because I read it in this user's `problem.md`?". If the second: out.

## Checkpoint with the user (BEFORE persisting)

After PASS 1+2+3 (nothing on disk yet), present the user a compact summary in main chat — **in the active locale** (`meta.yaml.lang`):

- Names of chosen experts + `role_category` + one-line angle each.
- Friction archetypes if any, with the dimension they cover.
- Omitted friction dimensions and why (briefly).
- Per-variable coverage of `deliverable.md` (briefly) — to verify each output section has at least one owner.

Then ask the user — in the active locale — whether the panel is good and whether they want to add / remove / swap experts, change a `role_category` or adjust an angle. The exact wording is up to the director; the semantic is: present the composition + invite changes or confirmation.

## Adjustment loop (pre-persistence — cheap)

If the user asks for changes, adjust the in-memory draft and re-present the summary. **Nothing is on disk yet** — no files to delete, no rewrites needed beyond your working draft:

- **Remove an expert** → drop it from the in-memory panel.
- **Add an expert** → repeat PASS 1+2 ONLY for that role (keeping the rest of the panel in memory).
- **Change role_category or angle** of an expert → re-draft its persona from PASS 2 with the new angle (maintaining anti-leak discipline: do not re-read `problem.md` beyond what's needed).
- **Replace friction archetype** → same procedure as for an expert.

After each adjustment, present the updated summary and ask again. Repeat until the user gives an **explicit confirmation token** in the active locale (a clear affirmative — not a hedge, not a "creo que sí / I think so"). DO NOT proceed to "Write the files" without explicit confirmation.

## Write the files (only after explicit confirmation)

Only once the user has confirmed the panel, persist to disk.

**Write `council/<run-id>/panel.md`** materializing `actions/deliberate/templates/panel.md.tpl` against the active locale (see SKILL.md TEMPLATE RESOLUTION). In the `user_composition_decisions` block of the panel notes (heading resolved from the locale pack), reflect whatever happened during the checkpoint loop: either that the panel was accepted on first proposal, or the specific adjustments the user requested.

**Write `council/<run-id>/experts/<name>/persona.md`** (one per expert) materializing the template per `kind` against the active locale:
- `kind: specialist` → `actions/deliberate/templates/persona-specialist.md.tpl` (constructive voice: assertions, recommendations, criteria).
- `kind: friction` → `actions/deliberate/templates/persona-friction.md.tpl` (adversarial voice: uncomfortable questions, lenses of doubt; mandatory `{{R:friction_function}}` guard-line resolved from the locale).

If something needs to change after this point (e.g. the user reads a persona and dislikes it), the cost is rewriting that file specifically — the cheap edits already happened in the pre-persistence loop.

## Return to `deliberate.md`

Once `panel.md` + all `experts/<name>/persona.md` are on disk reflecting the user-confirmed composition, return to `deliberate.md` and continue with STEP 3.
