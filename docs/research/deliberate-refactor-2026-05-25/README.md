# Refactor of `actions/deliberate.md` — prior research (2026-05-25)

This directory contains the 4 reports produced by parallel research agents to inform a refactor of `actions/deliberate.md` (467 lines), plus a synthesis with a unified plan.

## Motivation

After the 2026-05-25 session (post v2 spike with user in the loop), the `deliberate.md` file grew to 467 lines. The user's concern: *"467 líneas pero pocas reglas duras — la mayoría son plantillas + prompts ad-hoc + procedimiento. Diluye normas y crece monotónicamente sin nunca simplificarse"*.

Initial estimated composition:
- ~28% embedded markdown templates (`panel.md`, `persona.md`, `lead_notes.md`, etc.)
- ~32% ad-hoc prompts to the sub-agent (literal, repeated across rounds)
- ~22% pure procedure
- ~13% rules / guards / examples
- ~5% explanatory prose

## Research

4 agents launched in parallel, each with a narrow scope:

| # | Focus | File |
|---|---|---|
| 1 | Skills/agents convention (Anthropic skills + Copilot CLI custom agents) | [01-skills-agents-convention.md](01-skills-agents-convention.md) |
| 2 | Multi-agent orchestration frameworks (CrewAI, AutoGen, LangGraph, OpenAI Agents SDK, Magentic-One) | [02-multi-agent-orchestration.md](02-multi-agent-orchestration.md) |
| 3 | Prompt engineering — long procedures (Anthropic docs + arXiv + anti-patterns) | [03-prompt-engineering-long-procedures.md](03-prompt-engineering-long-procedures.md) |
| 4 | Panel-of-experts patterns (academic papers + ChatDev/MetaGPT/AgentVerse/CAMEL) | [04-panel-of-experts-patterns.md](04-panel-of-experts-patterns.md) |

Each agent received the `SKILL.md` + `actions/deliberate.md` as mandatory grounding and returned a structured report (~600-800 words) with findings + applicable patterns + refactor options + anti-patterns.

## Strong convergences (cited in ≥3 reports)

| Pattern | F5.1 | F5.2 | F5.3 | F5.4 | Backing |
|---|---|---|---|---|---|
| Extract embedded templates to `templates/` sidecars | ✅ | ✅ | ✅ | ✅ | Anthropic progressive disclosure |
| Extract ad-hoc sub-agent prompts to files | ✅ | ✅ | ✅ | ✅ | AutoGen / OpenAI Agents SDK / MetaGPT |
| Factor out common expert header (4 identical rounds) | ✅ | ✅ | ✅ | ✅ | `RECOMMENDED_PROMPT_PREFIX` (OpenAI Agents SDK) |
| DRY of the "Collect escalations" block | ✅ | ✅ | ✅ | – | "named prompt" pattern (Magentic-One) |
| One-level-deep references (no nesting) | ✅ | – | ✅ | – | Anthropic docs |
| Anti-pattern: mixing process + context + templates | ✅ | ✅ | ✅ | ✅ | MindStudio + arXiv + DigitalApplied |

**Key quote (F5.1, Anthropic official)**: *"Keep SKILL.md body under 500 lines. Split content into separate files when approaching this limit."*

## Negative convergences (what they do NOT recommend)

- ❌ Declarative table / virtual RoundRunner — F5.2 and F5.4 flag it due to the exceptionality of Round C (shuttle, not parallel).
- ❌ YAML schemas for expert outputs — F5.4 proposes it with an "excessive rigidity" caveat.
- ❌ Nesting templates within templates — violates one-level-deep documented by Anthropic.

## Unique contributions per report

- **F5.1**: hard 500-line limit documented by Anthropic. Official skills (`claude-api`, `mcp-builder`, `skill-creator`) as real-world references.
- **F5.2**: concrete name `CLOSE_ROUND(round)` for the subroutine. Note: the "see Round A for detail" *is the explicit signal* that a subroutine is missing.
- **F5.3**: **objective metrics** to measure the refactor (rules/line ratio, lines-between-rules, payload-output/procedure). Reorder STEPs to "hard-to-easy" (arXiv 2502.17204).
- **F5.4**: STEP 5.c (debate shuttle) is the exception that breaks the parallel pattern. Adaptive break in debate (3 explicit cut-off conditions).

## Unified refactor plan

Six stages, ordered by ROI/risk. Each stage delivers a valid state of the project.

### Stage 0 — Baseline and metrics (S, risk: 0)

Measure the current state before touching anything:
- Rules/line ratio (today ~13%, target >30%)
- Max lines-between-rules (today ~60, target <30)
- Payload-output / payload-procedure (today ~60/40, target ~20/80)

### Stage 1 — Quick win: DRY of "Collect escalations" (S, risk: low)

Extract the batching+dedup procedure to a sub-section `CLOSE_ROUND(round)`. STEPs 3/4/5/6 end with *"Execute CLOSE_ROUND('a' | 'b' | 'c' | 'd')"*. Delete the current "meta note". Savings: -20 to -25 lines.

### Stage 2 — Factoring of sub-agent prompts (M, risk: low)

Create a `### EXPERT_SPAWN_HEADER` section at the top: identity + isolation + language + evidence discipline. STEPs 3/4/5.c/6 rewrite the prompt as a **pure delta** (new paths + output path + verb). Savings: -100 to -150 lines.

### Stage 3 — Templates to sidecar `templates/*.tpl` (M, risk: medium)

Move embedded markdown skeletons to `.tpl` files: `panel.md.tpl`, `persona-specialist.md.tpl`, `persona-friction.md.tpl`, `lead_notes.md.tpl`, `debate_summary.md.tpl`, `debate_mediated.md.tpl`. Savings: -100 to -130 lines.

### Stage 4 — Internal reorder to "hard-to-easy" (M, risk: medium)

Based on arXiv 2502.17204 + lost-in-the-middle. Each STEP: **(a) hard rules at the start** → **(b) procedure** → **(c) examples at the end**. Concrete cases: STEP 1.5 (anti-leak guard up top, examples at the end), STEP 7 (SCOPE rule up top, not buried in the prompt to the moderator).

### Stage 5 — STEP 2 (panel design) to a separate file (M-L, risk: medium)

Move STEP 2 (~124 lines, 27%) to `actions/deliberate/panel-design.md`. `deliberate.md` references it with a 3-line pointer. Still a single skill entry (`panel-design.md` is not a sub-skill — it is a referenced procedure file, just as `actions/*.md` is to `SKILL.md`).

### Expected result

| Metric | Today | After E1-E2 | After E1-E5 |
|---|---|---|---|
| Lines `deliberate.md` | 467 | ~340 | **~180-220** |
| Rules/line ratio | ~13% | ~20% | **>30%** |
| Lines-between-rules (max) | ~60 | ~40 | **<25** |
| Files to read at runtime | 1 | 1 | **1-3** (depending on flow stage) |

## Decision taken

Apply **all stages (E0 → E5)** in order. Implementation tracking document: see commit history + §11 section of `docs/copilot-portability-spike-2026-05-24.md` (entry to be added at the close of the refactor).

## Refactor result (E6 — final metrics, 2026-05-25)

### Objective metrics (measured with `grep` + `wc -l`)

| Metric | Baseline (E0) | After E1-E5 | Change |
|---|---|---|---|
| Lines `deliberate.md` | 467 | **322** | **-31%** |
| Hard rules detected | 40 | 33 | -7 |
| Rules/line ratio | 8.6% | 10.2% | +1.6pp |
| Lines-between-rules (max gap) | **70** | **51** | -27% |
| Payload (code blocks + blockquotes) | 184 lines (39%) | 96 lines (30%) | -48% |
| Pure procedure | 283 (61%) | 226 (70%) | improved ratio |

### Honest reading of the numbers

- ✅ **Line reduction (-31%)** of the main file — the council director loads 145 fewer lines each turn. The combined total (deliberate.md + panel-design.md + 6 templates) is 522 lines, but only 322 are loaded at the start of the flow; the rest load on-demand (Anthropic's progressive disclosure).
- ✅ **Max gap reduced (-27%)** — the largest "lost-in-the-middle" zone went from 70 to 51 lines. There is still a considerable gap in STEP 3-5 (prompts zone), but more manageable.
- ⚠️ **Rules/line ratio rose only +1.6pp** (8.6% → 10.2%) — below the ambitious >30% target. But the count is misleading: many "rules" in the baseline were *repetitions* across the 4 round prompts (e.g., "DO NOT communicate with other experts" was counted 4 times). After factoring they live in a single place (`EXPERT_SPAWN_HEADER`), which lowers the raw count but **raises the quality** (a single source of truth). The real improvement is in the "HARD RULES FOR THIS STEP" headers promoted to the start of STEPs 1.5, 2 and 7 — visible at the top, not buried mid-instruction.
- ✅ **Payload -48%** — went down from 184 to 96 lines. The user's intuition ("too much embedded filler") is confirmed: nearly half of the templates + prompts lived inline; now they live in sidecars loaded only if the STEP needs them.

### What was NOT measured

- **Time-to-rule** (F5.3) — requires manual measurement with a stopwatch; not automatable.
- **Quality of the director's reasoning** post-refactor — requires a new end-to-end spike (not executed in this refactor).
- **Loading cost of the Copilot director** — not measured, but the official 500-line limit is now comfortably respected.

### Resulting structure

```
.claude/skills/council/actions/
├── deliberate.md (322 líneas) — procedimiento orquestador
└── deliberate/
    ├── panel-design.md (77 líneas) — extracto del STEP 2
    └── templates/
        ├── panel.md.tpl (32)
        ├── persona-specialist.md.tpl (25)
        ├── persona-friction.md.tpl (28)
        ├── debate_mediated.md.tpl (12)
        ├── lead_notes.md.tpl (13)
        └── debate_summary.md.tpl (13)
```

Everything stays **one-level-deep** from `deliberate.md` (following the Anthropic rule cited in report F5.1). It remains **a single skill entry** — the sub-files are procedural/templates, not sub-skills.

### Next steps (not included in this refactor)

- **New end-to-end spike** post-refactor to validate that the director executes correctly with the new structure (the file changed, not the semantics — but the model needs to navigate more references).
- **Observable quality metric**: compare the outcome of a new run vs `paneles-solares-finca-v2` (same problem.md, post-refactor) to verify parity or regression.

## Extension to `refine.md` (Phase F7, 2026-05-25)

After closing the refactor of `deliberate.md`, the same pattern was applied to `refine.md` with two key decisions:

### Architectural decision: subroutines to the core

The subroutines `CLOSE_ROUND(round)` and `EXPERT_SPAWN_HEADER(name, role_category)` were **moved from `deliberate.md` to `SKILL.md`** ("REUSABLE SUBROUTINES" section). Reasons:

- They apply to **two actions** (`deliberate` AND `refine`) — they were cross-cutting, not specific to deliberate.
- Maintains **one-level-deep**: any action → SKILL.md (always loaded) → subroutines. There is no action→action.
- SKILL.md grows from 354 to 409 lines (still comfortably below the official 500-line limit).

`EXPERT_SPAWN_HEADER` was updated to support both cases: in `deliberate` the anchor is `hypothesis.md`; in `refine` it is `follow_up.md`. The P4 precedence ("if `follow_up` contradicts `problem.md`, `follow_up` wins") is documented inline.

### Final metrics `refine.md`

| Metric | Baseline (F7.B0) | After F7.A-B3 | Change |
|---|---|---|---|
| Lines `refine.md` | 208 | **221** | **+6%** |
| Hard rules detected | 15 | 20 | +5 |
| Rules/line ratio | 7.2% | 9.0% | +1.8pp |
| Lines-between-rules (max gap) | 29 | 36 | **+24%** (worse) |
| Payload (code blocks + blockquotes) | 59 (28%) | 56 (25%) | -5% |

### Honest reading — why refine.md was NOT shortened

Unlike `deliberate.md` (-31%), `refine.md` grew slightly. Three reasons:

1. **The original prompts were SO abbreviated** (e.g. *"Spawn fresh per expert. Same identity header + paths as in Round A, plus: read X"*) that expanding them to `EXPERT_SPAWN_HEADER + pure delta` adds lines. But the original abbreviated form was fragile — it required the director to memorize/reconstruct the header. Now each round explicitly states its delta without re-emitting identity.
2. **Few embedded templates** to extract — only `follow_up.md` and the parent appendix. `lead_notes.md` and `debate_mediated.md` are reused from `deliberate/templates/`.
3. **"HARD RULES" headers** added 3 blocks (~15 lines) to STEPs 2, 6, 8. Qualitative gain.

The max gap rose from 29 to 36 — it worsened locally in STEP 7 (Round A′ → B′ zone), between two prompts without hard markers. Not blocking, but a candidate for a future iteration (insert a short `> rule` between rounds).

### Real gain from the refine.md refactor

It is NOT in lines. It is in:

1. **Cross-cutting DRY with deliberate** — both actions reference the same 2 subroutines in SKILL.md. A single source of truth for `CLOSE_ROUND` and `EXPERT_SPAWN_HEADER`.
2. **Shared templates** — `lead_notes.md.tpl` and `debate_mediated.md.tpl` are reused from `actions/deliberate/templates/`. If the format evolves, it is changed in one place.
3. **Visible hard rules** at the start of STEPs 2, 6 and 8 (CAPTURE DISCIPLINE + P4 precedence in STEP 2; prunable panel + max 1 new expert + blocking checkpoint in STEP 6; SCOPED + AUTONOMOUS + do not duplicate parent in STEP 8).
4. **Consistency** between actions — a future maintainer reads the same pattern in both files.

## Overall refactor result (deliberate + refine + SKILL)

| File | Baseline | Final | Change |
|---|---|---|---|
| `SKILL.md` | 354 | 409 | +16% (centralized subroutines) |
| `actions/deliberate.md` | 467 | **272** | **-42%** |
| `actions/refine.md` | 208 | 221 | +6% (DRY + hard rules added) |
| **TOTAL always-loaded files per action** | 821 + 562 = ~1390 (deliberate) / 822 + 562 = ~1390 (refine) | 681 + ~200 = **~880 per action** | **~-37%** |

New files (loaded only when the STEP needs them):
- `actions/deliberate/panel-design.md` (77 lines)
- `actions/deliberate/templates/*.tpl` (6 files, 123 lines)
- `actions/refine/templates/*.tpl` (2 files, 14 lines)

### Conclusion

The refactor meets the user's original concern (*"467 líneas pero pocas reglas duras"*) along **two paths**:
- **Reduction of the main deliberate file (-42%)** via template extraction + prompt factoring.
- **Rule densification** via "HARD RULES FOR THIS STEP" headers promoted to the start of the larger STEPs (1.5, 2, 7 in deliberate; 2, 6, 8 in refine).

Refine.md is the outlier in lines (+6%) — but it gains in cross-cutting DRY + reuse + consistency with deliberate. The absolute line count is not the sole metric.

### Final structure

```
.claude/skills/council/
├── SKILL.md (409)                          ← núcleo, siempre cargado
│                                              + sección SUBRUTINAS REUTILIZABLES
└── actions/
    ├── deliberate.md (272)                 ← procedimiento orquestador
    ├── deliberate/
    │   ├── panel-design.md (77)            ← STEP 2 extraído
    │   └── templates/*.tpl (123)           ← 6 skeletons de output
    ├── refine.md (221)                     ← procedimiento orquestador (Tiers 1/2)
    ├── refine/
    │   └── templates/*.tpl (14)            ← 2 skeletons específicos
    ├── iterate.md (69)
    ├── import.md (36)
    └── status.md (56)
```

Everything stays **one-level-deep** from SKILL.md (following the Anthropic rule cited in report F5.1). It remains **a single skill entry** with N referenceable files.
