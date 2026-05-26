# ACTION · `iterate`

> Procedure for the `iterate` action. Loaded by the core (`SKILL.md`) in its STEP 0.
> Applied TOGETHER with the core: everything in `SKILL.md` (ROLE, STRUCTURED MARKERS, CAPTURE
> DISCIPLINE, WRITING DISCIPLINE, INLINE VALIDATION, WHAT NOT TO DO) remains in effect here.

Invocation: `/council-iterate <branch> [<problem-id>] [--lang=<code>]` (Claude Code) or `copilot --agent council-iterate -p "<branch> [<problem-id>] [--lang=<code>]"` (Copilot CLI). The skill's umbrella `/council iterate <branch> ...` also works as a fallback.

`iterate` is pure conversational capture: DO NOT spawn sub-agents (via the environment's spawn primitive — `Task` in Claude Code, `/fleet` in Copilot CLI) in this action — you conduct all capture directly in chat.

> **Active locale**: `iterate` is the place where a problem's `lang` is first set (or, for legacy problems without it, where the director may add it). All chat with the user happens in that `lang`. User-facing example phrases in this file are written in English for readability; at runtime the director speaks in `lang`.

STEP 1 · Resolve target

- Parse `$ARGUMENTS`. Extract `<branch>` (required), optional `<problem-id>`, and optional `--lang=<code>` flag (e.g. `--lang=es`, `--lang=en` — supported locales are the YAML files under `.claude/skills/council/locales/` minus `_spec`).
- If `<branch>` is missing: clear error and stop.
- If `<problem-id>` is passed AND `branches/<branch>/problems/<problem-id>/meta.yaml` exists:
  - If `status: open`: ask the user (in Spanish) to confirm reopening (returns to `draft` during iterate, back to `open` on close). Reopening is legitimate even if the problem already has runs in `council/` — those runs are untouched.
  - If `status: draft`: resume from current state → STEP 2.
- If `<problem-id>` is passed but does NOT exist: this is not an error. List the branch's existing problems and ask (in Spanish) — *"No hay ningún problema `<problem-id>` en `<branch>`. ¿Creo uno nuevo con ese nombre, o querías otro?"*. If confirmed → create with `<id> = <problem-id>` (this is the anti-typo guard). If they wanted another → adjust and resolve again.
- If `<problem-id>` is NOT passed: ask the user (in Spanish) — *"¿Nombre corto para este problema? (kebab-case, ej: `paneles-huerto`, `riego-invernadero`). Algo que reconozcas y puedas teclear."*. That's the `<id>`. If the user declines to name it, use the fallback `YYYY-MM-DD-HHmm` (UTC).

**Create the new problem** (when one of the paths above decides it):
- Validate the `<id>`: kebab-case ASCII, no accents or ñ, max 30 chars. If `branches/<branch>/problems/<id>/` already exists, say so and ask for another name.
- **Resolve `lang`**:
  - If `--lang=<code>` was passed: use it. Verify `locales/<code>.yaml` exists; if not, error and list available locales.
  - Otherwise: infer from the language of the user's first input (the message that triggered the wrapper). If the user wrote in Spanish → `es`; English → `en`. If ambiguous (single short word) ask once: *"What language should this council operate in? `es` / `en`."*
- Create directory `branches/<branch>/problems/<id>/`.
- Read schema `schemas/problem.schema.yaml`.
- For each **required** entry of `sections`, resolve its `locale_ref` against `locales/<lang>.yaml` → returns `{title, purpose}`. Generate `problem.md` skeleton with one `## <title>` per required entry, each initialised with `[GAP: <purpose>]`. **Optional** sections are NOT created in skeleton — they materialize if the user provides content. (See SKILL.md → SCHEMA REFERENCE for the resolution algorithm.)
- Write `meta.yaml` with `id: <id>`, `status: draft`, `lang: <resolved code>`, ISO timestamps (`created_at` / `updated_at`), `schema_version`.

**Resume of an existing problem** (when `meta.yaml` exists):
- Read `meta.yaml.lang`. If the field is absent (pre-v2.6 problem), default to `es` and write that field back to `meta.yaml` on the next persistence write. **Do NOT bump `schema_version`** when backfilling `lang` — `schema_version` records which schema generated the `problem.md` content, and that content (with hardcoded Spanish titles) is not being regenerated. DO NOT silently re-infer from user input — once a problem has a `lang`, it's sticky.
- If `--lang=<code>` was passed AND it differs from the persisted `lang`: this is an explicit language change. Confirm with the user in the active language (*"You requested `<new>`; the problem is currently in `<old>`. Confirm switch?"*); on confirmation update `meta.yaml.lang` and proceed. Note that **the heading text inside any existing `problem.md` is NOT rewritten** — locale switching applies to future captures and new runs, not retroactive content.

STEP 2 · Capture loop

Repeat until user close signal OR zero `[GAP]` remaining:

1. **Read** `problem.md` from disk.
2. **Identify open `[GAP]`**: list, prioritize 1-3. Heuristic: early sections of the schema before late ones; gaps that block downstream interpretation (Goal, Decision) before peripheral ones.
3. **Ask in chat**: 1-3 plain-language questions, **without explaining why you ask**. Reference the section being filled (in Spanish, e.g. "Para el contexto: ..."). DO NOT show raw markers to the user.
4. **Receive response**. The user can:
   - Answer all, some, or none.
   - **Skip** ("paso", "no sé", "no aplica") → convert the corresponding `[GAP]` to `[SKIP: ...]` or `[N/A]`.
   - **Accept an assumption** the director proposed earlier → record as `[ASSUME: ...]`.
   - **Redirect** ("primero háblame de X") → re-prioritize.
   - **Close** ("listo", "done", "cerrar", "ya está") → STEP 3.
5. **Write** the response to `problem.md`:
   - Place content in the correct schema section.
   - Replace the resolved `[GAP]` with the user's content (no marker).
   - If the response surfaces new gaps not in the original schema, ADD `[GAP: ...]` in the most relevant section. Schema is baseline; conversations expand it.
   - Apply CAPTURE DISCIPLINE rigorously.
   - Update `meta.yaml: updated_at`.
6. **Validate inline** (see INLINE VALIDATION). Report a compact line.
7. Loop. **DO NOT suggest closing** unless:
   - **(a) Zero `[GAP]` remaining** (→ STEP 3), OR
   - **(b) User typed an explicit close signal** — concrete strings: "cerrar", "listo", "done", "ya está", "termina", "cierra".

   When `[GAP]`s remain and no close is requested, ask the next thing. If the only remaining `[GAP]`s are items the user said they would provide LATER, you MAY offer to convert them to `[SKIP]` (in Spanish: "¿lo dejo como SKIP o lo dejas abierto?") — that is NOT a close proposal.

STEP 3 · Close

Enter only if **(a)** zero `[GAP]` OR **(b)** user typed an explicit signal.

- Final inline validation: count by status.
- If zero `[GAP]` (close path inferred by zero-gap): **before** announcing close-ready, ask ONCE (in Spanish): *"¿hay algo que no te haya preguntado y pueda importar?"* If the user adds material, capture and repeat. Only when they answer "no, nada más" (or equivalent) can you announce (in Spanish) "Cero gaps abiertos — listo para cerrar." and ask for confirmation.
- If user-initiated close with `[GAP]` remaining: list the open ones and ask (in Spanish) "¿Confirmas cierre con estos gaps abiertos? Los puedes dejar abiertos o convertir a SKIP." (The any-else? sweep is NOT triggered here — the user asked to close.)
- On confirmation:
  - Update `meta.yaml`: `status: open`, `updated_at`.
  - Inform (in Spanish): "Problema `<id>` cerrado. Para deliberar: `/council <branch> <id>`."
- If the user cancels close: return to the capture loop.

STEP 4 · Interruption / resume

- If the conversation is interrupted without explicit close: the file stays `status: draft`. Recoverable.
- On resume with `/council-iterate <branch> <id>`: read `problem.md` + `meta.yaml`, pick up from current state.
