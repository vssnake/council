# Council — Architecture

> **Contract document**, not history. Describes the system as it is. For changelog see [`CLAUDE.md`](CLAUDE.md); for end-user usage see [`README.md`](README.md); for runtime procedures see [`.claude/skills/council/`](.claude/skills/council/); for refactor narratives see [`docs/research/`](docs/research/).
>
> **Audience**: developers / contributors / future-you returning after weeks. **Goal**: coherent mental model of how the pieces fit, where each invariant is enforced, and what can fail.
>
> **Update policy**: only when the architecture changes (new component, new invariant, new failure-mode category). Not on feature additions, refactors, or bug fixes — those go in `CLAUDE.md`'s changelog.

---

## 1. Component map

```
┌────────────────────────────────────────────────────────────────────────┐
│  USER                                                                  │
│    │ types slash command (Claude Code) or invokes custom agent (Copilot CLI)
│    ▼                                                                   │
│  ENTRY POINTS ── Claude Code: /council-<action> <args> (5 slash       │
│                   commands in .claude/commands/, thin shells; the     │
│                   skill also auto-creates /council as umbrella)       │
│                  Copilot CLI: copilot --agent council-<action> -p     │
│                   "<args>" (5 thin per-action agents in .claude/      │
│                   agents/; description-based auto-dispatch)           │
│    │ each wrapper activates SKILL with its action token               │
│    ▼                                                                   │
│  DIRECTOR (LLM running the skill)                                      │
│    │                                                                   │
│    ├── reads (ALWAYS) ────►  .claude/skills/council/SKILL.md           │
│    │                                                                   │
│    ├── reads (on demand) ─►  .claude/skills/council/actions/<action>.md│
│    │                          (+ sidecars: deliberate/panel-design.md) │
│    │                                                                   │
│    ├── resolves ─►   locales/<lang>.yaml      (H/I/R/S/F strings)      │
│    │                  actions/*/templates/*.tpl (output skeletons)     │
│    │                  schemas/*.yaml          (problem/deliverable)    │
│    │                                                                   │
│    ├── persists ─►   branches/<branch>/problems/<id>/{problem,meta}    │
│    │                                                                   │
│    └── spawns sub-agents via SPAWN PRIMITIVE                           │
│         │                                                              │
│         ▼                                                              │
│       COUNCIL-EXPERT (fresh context per spawn — no peer-to-peer)       │
│         │ reads persona + base context + previous-round files          │
│         │ writes exactly ONE output file                               │
│         ▼                                                              │
│       branches/<branch>/problems/<id>/council/<run-id>/...             │
└────────────────────────────────────────────────────────────────────────┘
```

| Component | Type | Role |
|---|---|---|
| `SKILL.md` | Core procedure | Role, data model, cross-cutting disciplines (CAPTURE / LEAD / WRITING / ESCALATION / STYLE & SAFETY), reusable subroutines (`CLOSE_ROUND`, `VERIFY_OUTPUTS`, `EXPERT_SPAWN_HEADER`, `LANGUAGE_DISCIPLINE`), template resolution rules. **Always loaded** into director's context. |
| `actions/<action>.md` | Procedure (5 actions) | Step-by-step for the director. Loaded **on demand** based on `$ARGUMENTS` first token (`iterate` / `import` / `deliberate` / `refine` / `status`). |
| `actions/deliberate/panel-design.md` | Procedure sidecar | STEP 2 of deliberate: 3-pass discipline + naming rules + blocking checkpoint + adjustment loop. Read by the director when it reaches STEP 2. |
| `actions/*/templates/*.tpl` | Output skeletons | Language-neutral markdown with `{{H:key}}` / `{{I:key}}` / `{{R:key}}` placeholders. Resolved against the active locale at materialization time. |
| `locales/<lang>.yaml` | Locale pack | Localized strings in five categories: **H**eadings, **I**nstructional prompts (text inside `<...>` brackets), **R**ules (hard, verbatim), **S**pawn fragments (full bodies of subroutines + round deltas + moderator prompts), **F**ew-shot examples per output role. Contract: `locales/_spec.yaml`. |
| `schemas/problem.schema.yaml` (v0.5) | Schema | Structural definition of `problem.md` (section identifiers + optional flags) + `meta.yaml` shape (with `lang:` field). All human-language strings (titles + purposes) move to the locale pack via each section's `locale_ref:`. |
| `schemas/deliverable.schema.yaml` (v0.2) | Schema | Structural definition of `deliverable.md` per run (section identifiers + optional flags). Titles/purposes via `locale_ref:`. |
| `.claude/commands/council-<action>.md` × 5 | Thin slash command wrappers (Claude Code) | Frontmatter (`description`, `argument-hint`) + 1-line body that activates the skill with the action token. The skill ALSO auto-creates `/council` as an umbrella fallback that accepts the action as first arg. |
| `.github/agents/council-<action>.agent.md` × 5 | Thin custom-agent wrappers (Copilot CLI) | Frontmatter (`name`, `description`, `tools`, `model`, etc.) + 1-line body. Copilot's description-based auto-dispatch routes natural-language prompts to the matching agent. Canonical project-level path per Copilot CLI docs. |
| `.github/agents/council-expert.agent.md` | Sub-agent template (Copilot CLI) | Generic expert/moderator role invoked via `/fleet`. Used as the actual sub-agent process by `council-deliberate` and `council-refine`. |
| `branches/<branch>/problems/<id>/` | Runtime storage | All state lives here — no database, no sessions, no in-memory state. |

---

## 2. Lifecycle of a run

### 2.1 Problem state machine

```
                  /iterate or /import                  user closes capture
   [ no problem ] ──────────────────► [ status: draft ] ──────────────► [ status: open ]
                                            ▲ │                                  │
                                            │ │ continues iterate                │
                                            └─┘                                  │
                                                                                 │
                                                                     /deliberate ▼
                                                                       run created
```

`meta.yaml.status` is `draft` (in capture) or `open` (closed, ready for runs). It does NOT track deliberation state — that's derived from `council/<run-id>/run_meta.yaml`.

### 2.2 Run state machine

```
                         /deliberate creates run dir + run_meta.yaml
   [ no run ] ──────────────────────────────────────────► [ in_progress ]
                                                                  │
                                          all rounds + outcome.md │
                                                                  ▼
                                                           [ complete ]
                                                                  │
                                                                  │ /refine creates child
                                                                  ▼
                                                       [ parent of N child runs ]
```

`run_meta.yaml.run_status`: `in_progress` | `complete`. A complete run is **immutable** except for the narrow exception of the parent-appendix navigation pointer added by `refine` STEP 9.a.

### 2.3 Files created — `deliberate`

| STEP | File(s) created | Written by | Notes |
|---|---|---|---|
| 1 | `run_meta.yaml` | director | `kind: deliberation`, `run_status: in_progress` |
| 1.5 | `hypothesis.md` | director | Marked `[user-provided]` / `[auto-generated]` / `[N/A: open case]` |
| 1.6 | `deliverable.md` | director | Skeleton from schema, filled via iterative capture |
| 2 | `panel.md` + `experts/<name>/persona.md` × N | director | Materialized from `.tpl` against active locale. Anti-leak applies. |
| 3 (Round A) | `proposals/expert_<name>.md` × N | **sub-agents** (parallel) | INVIOLABLE: director never writes these |
| 3 close | `escalations/round_a.md` | director (if any) | Via `CLOSE_ROUND('a')` |
| 4 (Round B) | `critiques/expert_<name>.md` × N | **sub-agents** (parallel) | INVIOLABLE |
| 4 close | `escalations/round_b.md` | director (if any) | Via `CLOSE_ROUND('b')` |
| 5.b | `user_directives.md` | director (if user gave any) | Literal user text, no amplification |
| 5.c (Round C) | `debate/<conflict-id>__expert_<name>.md` × M | **sub-agents** (shuttle, sequential) | INVIOLABLE. M = conflicts × 2 (A-side, B-side) × cycles |
| 5.e | `debate_mediated.md` | **director** | Lead-written synthesis. Not INVIOLABLE (lead IS the author) |
| 5.f close | `escalations/round_c.md` | director (if any) | |
| 6 (Round D) | `final_positions/expert_<name>.md` × N | **sub-agents** (parallel) | INVIOLABLE |
| 6 close | `escalations/round_d.md` | director (if any) | |
| 7.a | `lead_notes.md` | **director** | Lead → moderator bridge. Not INVIOLABLE |
| 7.b | `debate_summary.md` + `outcome.md` | **moderator sub-agent** | INVIOLABLE. Outcome shape per `deliverable.md` (NOT a universal template) |
| 7.c | `run_meta.yaml` updated | director | `run_status: complete` |

### 2.4 Files created — `refine`

Tier 1 (clarification): minimal run.

| STEP | File | Written by |
|---|---|---|
| 1 | `run_meta.yaml` | director |
| 2 | `follow_up.md` | director |
| 3 | `run_meta.yaml` updated (`tier: 1`, `trigger`) | director |
| 4 | `outcome.md` OR `NEEDS-TIER-2.md` | **moderator sub-agent** (INVIOLABLE) |

Tier 2 (refinement): inherited panel + compressed rounds.

| STEP | File(s) | Written by |
|---|---|---|
| 5 | `deliverable.md` | director |
| 6 | `panel.md` + `experts/<name>/persona.md` × N (subset of parent + ≤1 new) | director |
| 7 (A′/B′/C′/D′) | Same as deliberate Rounds A/B/C/D but in child run dir | sub-agents (INVIOLABLE) |
| 8 | `lead_notes.md` (director) + `debate_summary.md` (conditional) + `outcome.md` (moderator, INVIOLABLE) | mixed |
| 9.a | Navigation appendix appended to **parent** `outcome.md` | director (only allowed edit to a moderator file) |

---

## 3. Invariants — and where each is enforced

The system depends on a small number of hard rules. Listed in approximate order of severity (top = most critical).

| # | Invariant | Where enforced |
|---|---|---|
| 1 ⚠ | **INVIOLABLE — no fabrication.** Director NEVER writes a sub-agent's missing output. On missing file: wait → re-spawn → stop. | `SKILL.md` → REUSABLE SUBROUTINES → `VERIFY_OUTPUTS` (full procedure + rationale); WHAT NOT TO DO #1; WRITING DISCIPLINE (lead does NOT create them); per-round verify calls in `deliberate.md` and `refine.md` |
| 2 | **No peer-to-peer.** Experts never communicate with each other; the director is the only channel. | `SKILL.md` → SPAWN PRIMITIVE invariants; `EXPERT_SPAWN_HEADER` body in `locales/<lang>.yaml`; `council-expert.agent.md` absolute invariants |
| 3 | **No expert ↔ user communication.** Experts never address the user directly; escalations go via the round barrier. | `EXPERT_SPAWN_HEADER`; `council-expert.agent.md`; `SKILL.md` → USER ESCALATION |
| 4 | **Round barrier for escalations.** Experts annotate `## {{H:questions_for_user}}` in their file; the lead batches at round close. | `SKILL.md` → USER ESCALATION + `CLOSE_ROUND` subroutine |
| 5 | **Scope of `outcome.md`.** Outcome respects EXACTLY the sections of `deliverable.md`. Out-of-scope content from experts is discarded. | Moderator prompts (`S:moderator_prompt_deliberate`, `S:moderator_prompt_refine_tier2`) + POST-SYNTHESIS AUDIT inside the moderator body |
| 6 | **Anti-leak.** `panel.md` and personas DO NOT replicate `problem.md` data (brands, models, figures, locations, decisions, stakeholders, stack). Domain vocabulary only. | `panel-design.md` HARD RULES; STEP 2 of `deliberate.md`; WHAT NOT TO DO #3; mental test in 3-pass discipline |
| 7 | **Capture discipline.** Director never adds figures/brands/inferences the user didn't say. Structured scribe, not consultant. | `SKILL.md` → CAPTURE DISCIPLINE; `iterate.md`/`import.md` STEPs |
| 8 | **Director does not synthesize `outcome.md` itself.** Final synthesis is delegated to an impartial moderator sub-agent. | `SKILL.md` → LEAD/DIRECTOR RULES "DO NOT"; STEP 7.b of `deliberate.md` / STEP 8.b of `refine.md` |
| 9 | **Filesystem-as-state.** No `state.md`, no `log.md`, no `sessions/`. All state lives in DATA MODEL files. | `SKILL.md` → WHAT NOT TO DO → Architecture |
| 10 | **Fresh spawn per round.** No persistent teammates between rounds — each round's sub-agents have isolated context. | `SKILL.md` → SPAWN PRIMITIVE invariants |
| 11 | **User has final word.** If the user rejects something, the director respects it. | Project rules (`CLAUDE.md`); LEAD/DIRECTOR RULES "DO NOT push for consensus" |
| 12 | **Language stickiness.** `meta.yaml.lang` is set on first iterate/import and sticky for the problem's lifetime; refinements inherit the parent run's locale. | `SKILL.md` → STYLE & SAFETY + DATA MODEL; `iterate.md`/`import.md` |
| 13 | **Output language consistency.** Sub-agents produce output in the active locale; canonical vocabulary pinned via few-shot. | `LANGUAGE_DISCIPLINE` subroutine appended at the END of every spawn prompt (lost-in-the-middle mitigation) |
| 14 | **Schema validation.** `problem.md` follows `problem.schema.yaml`; `deliverable.md` follows `deliverable.schema.yaml`. | Per-action READ schema steps in `iterate.md`/`import.md`/`deliberate.md` STEP 1.6/`refine.md` STEP 5 |
| 15 | **One-level-deep references.** `SKILL.md` → `actions/X.md` → optional sidecar/template — never deeper. | Anthropic Agent Skills best practice; enforced by project convention |

---

## 4. Failure modes — and their mitigations

Catalog of observed and theoretical failures. Each entry: mechanism + mitigation + where the mitigation lives.

### 4.1 Spawn-primitive races

**Mechanism**: The `/fleet` (Copilot CLI) and `Task` (Claude Code) spawn primitives emit a `completed` event when the sub-agent's text stream ends, **before** the sub-agent's `Write` tool effect is durable on the filesystem. Observed gap: up to ~40 s between sibling mtimes in a single parallel round.

**Reference**: [Copilot CLI issue #1324](https://github.com/github/copilot-cli/issues/1324) — `task_complete` token terminates execution before response capture.

**Mitigation**: `VERIFY_OUTPUTS` subroutine with 3 retries × 3 s backoff before declaring spawn failure. Built-in tolerance for ~10 s of race window.

### 4.2 Director fabricates missing sub-agent output

**Mechanism**: Both spawn primitives surface the sub-agent's full text output to the parent's context. When verification fails (race or genuine spawn failure), the director has enough material to fabricate the file: the persona, the spawn prompt's few-shot example, prior rounds' outputs from the same expert. Observed in real run `filtrado-aguas-grises/2026-05-25-diseno-sistema` Round D.

**Reference**: [Copilot CLI issue #2265](https://github.com/github/copilot-cli/issues/2265) (`assistant.message` delivers child's content to parent context); [Claude Code subagent docs](https://code.claude.com/docs/en/agent-sdk/subagents) (parent receives child's final message verbatim as tool result).

**Mitigation**: INVIOLABLE rule in `VERIFY_OUTPUTS`. **This mitigation is disciplinary, not architectural** — the material to fabricate is still in the director's context window. The rule depends on the director honoring the prohibition.

**Stronger mitigation available (not yet implemented)**: spawn a separate verifier sub-agent with empty context (no prior rounds) whose sole job is `ls` + checksum. The verifier reports a factual manifest the director acts on without ever seeing the children's content. See `docs/research/` for design notes.

### 4.3 Run interrupted mid-flow

**Mechanism**: Network failure, user kill, context window exhaustion, environment crash. Run dir exists but lacks `outcome.md`.

**Mitigation**: `INCOMPLETE RUNS` section in `SKILL.md` defines a detection-and-resumption table mapping "last artifact present" → "resume at STEP". Both `deliberate.md` STEP 1 and `refine.md` STEP 1 execute the detection before creating a new run. User chooses: resume / discard / start another.

### 4.4 Language confusion in sub-agent output

**Mechanism**: Documented LLM failure mode where output mixes languages, line-by-line or word-by-word, especially in long contexts with mixed-language prompts. Worse at high temperature.

**Reference**: [Marchisio et al. 2024, Language Confusion Benchmark](https://arxiv.org/html/2406.20052v1).

**Mitigation**: `LANGUAGE_DISCIPLINE(lang, role)` subroutine appended at the **end** of every spawn prompt (lost-in-the-middle positioning). Contains output-language directive + canonical vocabulary pinning + one in-language few-shot of the expected output structure for the target round.

### 4.5 Anti-leak violation

**Mechanism**: The director, while writing personas in STEP 2, reads `problem.md` and copies the user's specific brands/models/figures into the persona — contaminating the abstract expert role with this-user-this-project data.

**Mitigation**: 3-pass discipline in `panel-design.md` (PASS 1 selection, PASS 2 persona without re-reading problem.md, PASS 3 validation); HARD RULES at the top of STEP 2; mental test ("would this role be the same if the user had chosen another brand?").

### 4.6 Out-of-scope content in `outcome.md`

**Mechanism**: An expert in `proposals/`, `critiques/`, or `final_positions/` includes content outside `deliverable.md`'s declared sections (adjacent components, costs of unagreed subsystems, payback, implementation plan). The moderator includes it in `outcome.md`.

**Mitigation**: SCOPE rule in moderator prompt (locale pack: `S:moderator_prompt_deliberate` / `_refine_tier2`); POST-SYNTHESIS AUDIT executed by the moderator before closing (mandatory section→deliverable mapping).

### 4.7 Concurrent file write (last-writer-wins)

**Mechanism**: Two sub-agents in the same parallel round write to the same path. The spawn primitive does not arbitrate — the second write silently overwrites the first.

**Reference**: documented behavior of `/fleet` and `Task`.

**Mitigation**: Per-expert file naming convention. Each spawn prompt specifies a unique `<output-path>` derived from the expert's name (`expert_<name>.md`). The director composes paths from `panel.md`'s roster, which has unique names by construction (kebab-case identifiers).

### 4.8 Lost user escalation

**Mechanism**: An expert needs user input mid-round, but the round barrier rule forbids interrupting. If the lead fails to collect `## {{H:questions_for_user}}` sections at round close, the question is lost.

**Mitigation**: `CLOSE_ROUND(round)` subroutine, called explicitly at the end of every round. Reads all `## {{H:questions_for_user}}` sections, dedupes + attributes, batches to user, persists in `escalations/round_<round>.md`, injects in next round's spawn prompts.

---

## 5. Provider abstraction

The same SKILL runs in two environments via a normalized spawn primitive.

| Aspect | Claude Code | Copilot CLI |
|---|---|---|
| SKILL discovery | `.claude/skills/council/SKILL.md` (Skill tool) | `.claude/skills/council/SKILL.md` (direct path read) |
| User entry points | 5 `/council-<action>` slash commands in `.claude/commands/` (autocompleted) + umbrella `/council <action>` auto-created from skill | 5 `--agent council-<action>` custom agents in `.github/agents/` (description-based auto-dispatch) |
| Sub-agent spawn primitive | `Task` tool (`subagent_type="general-purpose"`) | `/fleet` for parallel + `council-expert` custom agent for the role |
| Parallel rounds (A, B, D) | Multiple `Task` calls in one assistant message | One `/fleet` with N invocations of `council-expert` |
| Sequential shuttle (C) | One `Task` per turn | One `--agent council-expert` invocation per turn |
| Web evidence | `WebSearch` + `WebFetch` tools | `web` + `web_fetch` + `github/*` MCP tools |

**Abstraction layer**: the `SPAWN PRIMITIVE` section in `SKILL.md` normalizes both providers into a single verb ("spawn a sub-agent with this prompt, write to this path"). The `actions/*.md` files reference the abstract verb, not the concrete primitive — so adding a third provider would be a `SKILL.md` SPAWN PRIMITIVE table update + a new wrapper directory, without touching the actions.

**Trade-off**: Claude Code's `Task` is built-in and simpler (no separate config file per agent), but offers less telemetry — the parent sees only the sub-agent's final message. Copilot CLI's `/fleet` requires a custom agent definition (`.github/agents/council-expert.agent.md`) and surfaces tool calls in the parent context (extra debugging info), but the same property creates the contamination vector documented in §4.2 (failure mode "Director fabricates missing sub-agent output").

**Provider-symmetric invariants** (hold in both):
- Fresh isolated context per spawn — sub-agents do not see the director's context or each other's.
- Shared filesystem without locks — coordinate writes by unique paths, not by mutex.
- No persistent teammates between rounds.
- Intra-round parallelism for A/B/D; sequential for C.

---

## 6. Where to look when…

| Question | File |
|---|---|
| How does the director know what action to run? | `SKILL.md` → ACTION ROUTING + STEP 0 |
| What's the data model on disk? | `SKILL.md` → DATA MODEL |
| How is a template materialized? | `SKILL.md` → TEMPLATE RESOLUTION + `locales/_spec.yaml` |
| What does each expert receive in its spawn prompt? | `SKILL.md` → `EXPERT_SPAWN_HEADER` + the round-specific delta in `S:task_round_*` |
| Why does the language-discipline block go at the END of the prompt? | `SKILL.md` → `LANGUAGE_DISCIPLINE` (lost-in-the-middle reasoning) |
| What happens if a sub-agent's file doesn't appear? | `SKILL.md` → `VERIFY_OUTPUTS` (with INVIOLABLE rule) |
| How are user escalations collected? | `SKILL.md` → USER ESCALATION + `CLOSE_ROUND` |
| How is the moderator scoped? | Locale pack: `S:moderator_prompt_deliberate` / `S:moderator_prompt_refine_tier1` / `S:moderator_prompt_refine_tier2` |
| What if a run is interrupted? | `SKILL.md` → INCOMPLETE RUNS (resumption table) |
| Why this architecture and not another? | `docs/research/deliberate-refactor-2026-05-25/` |
| When did each feature land? | `CLAUDE.md` (version log) |
