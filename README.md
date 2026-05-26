# Council

> Expert-panel LLM system for deliberating complex technical decisions. Conversational capture of a problem + dynamic panel of 4-6 experts (plus 0-2 friction archetypes) that debate across 4 rounds and produce a recommendation synthesized by an impartial moderator. No agent-teams, no hooks, no scripts: the entire system is **a single skill** runnable in Claude Code or GitHub Copilot CLI.

> **Note on language**: this project's documentation and procedures are in English (maintainability). **Runtime is multilingual via a locale pack** (`.claude/skills/council/locales/`): two locales ship today — `es` (Spanish) and `en` (English). Locale is set per problem at capture time (inferred from your first input, or via `--lang=`), persisted in `meta.yaml`, and sticky for the problem's lifetime. The example runs under `branches/` are in Spanish (the locale used when they were created).

---

## The problem it solves

When you have to decide something complex — choosing a database for a pipeline, sizing an installation, settling on an architecture, validating a technical purchase — you turn to an LLM. It usually gives you **a reasonable opinion** but with two chronic problems:

1. **It's a single voice.** The model suggests `X` with confidence, but you don't know if a specialist from the adjacent field would discard it for a reason the first model didn't consider.
2. **There's no deliberative tension.** If you ask "give me the pros and cons", you get a balanced but flat list — without the real friction of two experts defending opposing positions and a third poking both.

Council instruments that friction. Instead of a big prompt with "act as N experts", it **spawns N isolated sub-agents** (one per expert, with a persona generated inline), has them **propose, critique, debate, and take positions** across 4 rounds, and delegates the final synthesis to an **impartial moderator** distinct from the panel.

The result is an `outcome.md` with the recommendation + a `debate_summary.md` documenting where there was consensus, where there was conflict, and how it was resolved.

## How it works in one page

```
┌─────────────────────────────────────────────────────────────────────┐
│                          CAPTURE PHASE                              │
│                                                                     │
│  /council-iterate <branch>                                          │
│  └─ You capture the problem conversationally → problem.md            │
│     (or /council-import if coming from an external draft)            │
└─────────────────────────────────────────────────────────────────────┘
                                  ▼
┌─────────────────────────────────────────────────────────────────────┐
│                       DELIBERATION PHASE                            │
│                                                                     │
│  /council-deliberate <branch> <problem-id> <slug>                   │
│  │                                                                  │
│  ├─ 1. Iterate hypothesis (optional, skippable)                     │
│  ├─ 2. Iterate deliverable (mandatory — what shape the outcome has) │
│  ├─ 3. Design panel (4-6 specialists + 0-2 friction archetypes)     │
│  │                                                                  │
│  ├─ Round A — Initial proposals          ┐                          │
│  ├─ Round B — Cross critiques            │  Fresh spawn per         │
│  ├─ Round C — Debate (lead-mediated)     │  expert, in parallel     │
│  ├─ Round D — Final positions            ┘                          │
│  │                                                                  │
│  └─ Synthesis by impartial moderator → debate_summary.md + outcome.md
└─────────────────────────────────────────────────────────────────────┘
                                  ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    REFINEMENT PHASE (optional)                      │
│                                                                     │
│  /council-refine <branch> <problem-id> <parent-run-id> <slug>       │
│  └─ Iterates on the closed run without redoing it (3 tiers by cost) │
└─────────────────────────────────────────────────────────────────────┘
```

**Invariants that always hold**:

- Experts **NEVER talk directly to the user or to each other**. The director (the LLM running the skill) is the only channel.
- Each round **spawns fresh sub-agents** — no persistent agent-teams, no memory between rounds beyond the files.
- The director **NEVER fabricates a sub-agent's output**. If a spawn fails to produce its file (e.g. due to the documented race between `/fleet`'s completion notification and the child's filesystem write — see [Copilot CLI issue #1324](https://github.com/github/copilot-cli/issues/1324)), the only legitimate actions are **wait**, **re-spawn**, or **stop the run**. The director cannot synthesize a file from the sub-agent's output stream, its persona, or prior rounds — enforced by the `VERIFY_OUTPUTS` subroutine in `SKILL.md` as the system's strongest discipline rule.
- User escalations are a **round barrier**: experts annotate questions in their file, the lead batches them at round close.
- The final synthesis is produced by an **impartial moderator distinct** from the lead — so there's no shared authorship between who orchestrates and who concludes.
- **All state lives in files** under `branches/<branch>/problems/<id>/`. No database, no ephemeral sessions — a closed run is indefinitely auditable.

## When to use council — and when not

| Use council if... | Do NOT use council if... |
|---|---|
| The decision crosses several disciplines (technical + economic + operational) | It's a question with a direct answer ("what does the `-x` flag of `tar` do?") |
| You benefit from tension between viewpoints | You just need a quick suggestion |
| You want traceability: ability to revisit the debate 6 months from now | The context is ephemeral (one-off debugging) |
| The cost of being wrong justifies 10-20 minutes of deliberation | You need the answer in seconds |
| You want to know **what was challenged** and why, not just the final verdict | You don't care about the nuances, just the output |

## Comparison with alternatives (1 minute)

| System | Approach | When council wins |
|---|---|---|
| **CrewAI / AutoGen / LangGraph** | Code-first multi-agent (Python, YAML) | You prefer procedure-as-markdown executed by an LLM. You don't want to maintain a Python project to define agents. |
| **Multi-Agent Debate (Du et al. 2023)** | N copies of the same model deliberate | You want personas with differentiated identity (specialist vs friction) instead of N identical voices. |
| **MetaGPT / ChatDev** | Pipeline with fixed roles (PM, Architect, Engineer...) | Your problem is NOT software construction — the panel must be dynamic based on the problem. |
| **Single LLM consulting** ("act as N experts") | One enormous prompt | You want real isolation between voices — a single model "playing" 5 personas converges on itself (degeneration-of-thought). |
| **Official Anthropic Skills** | Executable procedure markdown | Council IS a skill — that's the base. The difference: council adds the expert orchestration pattern. |

Council borrows from all of them and distinguishes itself by the combo: **dynamic panel + shuttle diplomacy in debate + impartial moderator separate from the lead + filesystem-as-state + round barrier for user escalations**. It's not a framework — it's a concrete pattern persisted as a skill.

## Quick start — end-to-end example

### Requirements

- **Claude Code** (CLI or desktop) — uses the `Task` tool as spawn primitive.
- **Or GitHub Copilot CLI v1.0.51+** — uses `/fleet` + custom agents.

The skill is **provider-agnostic**: the procedure is the same, the spawn tool is decided by the environment.

Clone the repo and enter:

```bash
git clone <repo>
cd cc-prompts/council
```

### Real example: "which solar panels to buy"

This example is **already executed** in the repo — you can see the final outcome at `branches/paneles-solares/problems/2026-05-22-0644/council/2026-05-22-comparativa-paneles/outcome.md`. Here we reproduce the commands that generated it. *The runtime outputs of this example are in Spanish, because that was the locale active when it was captured. The same commands with `--lang=en` on iterate would have produced the equivalent run in English.*

**Step 1 — Capture the problem** (conversational, ~5-10 messages with the director):

```
/council-iterate paneles-solares
```

The director asks for a short name (slug). After that it enters *structured scribe* mode: it asks you neutral questions to fill `problem.md` following the schema in `schemas/problem.schema.yaml`. It captures what you say **without interpreting** — numbers, brands, constraints remain as you said them. When there are zero `[GAP]`s in required sections, the problem moves to `status: open` and is ready to deliberate.

Resulting file: `branches/paneles-solares/problems/2026-05-22-0644/problem.md` (~60 lines with sections for goal, context, constraints, prior decisions, etc.).

**Step 2 — Launch the deliberation**:

```
/council-deliberate paneles-solares 2026-05-22-0644 comparativa-paneles
```

The director executes 8 STEPs:

1. **Validates inputs and creates the run** `council/2026-05-22-comparativa-paneles/`.
2. **Iterates hypothesis** (skippable). In this example the user said: *"preference for panels with more watts"*. The director distills to 1 point without enriching.
3. **Iterates deliverable**. Asks what shape you want: the user said *"a panel comparison table"*.
4. **Designs the panel** (5 experts in this case): `tecnologo-modulos-fotovoltaicos`, `analista-mercado-fotovoltaico`, `economista-coste-fotovoltaico`, `ingeniero-dimensionado-fv` + 1 friction archetype `arquetipo-esceptico-operacional`. The user confirms or adjusts.
5. **Round A — Proposals** (5 sub-agents in parallel, each writes `proposals/expert_<name>.md`). If any expert asks for info, the director batches at close.
6. **Round B — Cross critiques** (5 sub-agents read the others' proposals and critique them).
7. **Round C — Debate** (lead-mediated, shuttle diplomacy per identified conflict).
8. **Round D — Final positions** + synthesis by an impartial moderator sub-agent.

Real measured cost: ~15-20 minutes, ~18 premium requests in Copilot CLI.

**Step 3 — Read the outcome**:

```
branches/paneles-solares/problems/2026-05-22-0644/council/2026-05-22-comparativa-paneles/
├── outcome.md              # final recommendation (149 lines: table of 11 models in 3 tiers + verdict on the hypothesis)
├── debate_summary.md       # what was debated and how it was resolved
├── proposals/              # 5 files, one per expert
├── critiques/              # 5 files
├── debate/                 # shuttle files per conflict
├── final_positions/        # 5 files
└── escalations/            # user Q&A, batched per round
```

**Step 4 — (optional) refine without redoing everything**:

```
/council-refine paneles-solares 2026-05-22-0644 2026-05-22-comparativa-paneles mono-vs-bifacial
```

The director classifies the follow-up into a tier (1 clarification / 2 real refinement / 3 out of scope) and routes. A Tier 2 inherits the parent run's panel, prunes it according to relevance to the follow-up, and runs compressed rounds. The parent's outcome receives a navigation appendix to the child refinement.

You can see the real refinement at `branches/paneles-solares/problems/2026-05-22-0644/council/2026-05-22-mono-vs-bifacial/outcome.md`.

### Other available commands

```
/council-status                            # read-only view: which problems and runs exist, what's incomplete
/council-import paneles-solares --from-file=draft.md   # imports an external draft instead of capturing conversationally
```

## Project structure

```
council/
├── README.md                                ← this file
├── CLAUDE.md                                ← instructions for Claude Code (not for humans)
├── .github/copilot-instructions.md          ← analogous for Copilot CLI
├── .claude/
│   ├── skills/council/
│   │   ├── SKILL.md                         ← skill core (always loaded): role, data model,
│   │   │                                       cross-cutting disciplines, reusable subroutines
│   │   ├── locales/                         ← v2.6 — locale pack (multilingual runtime)
│   │   │   ├── _spec.yaml                   ← contract: required keys per category (H/I/R/S/F)
│   │   │   ├── es.yaml                      ← Spanish locale
│   │   │   └── en.yaml                      ← English locale
│   │   └── actions/                         ← per-action procedures (loaded on demand)
│   │       ├── iterate.md                   ← conversational capture
│   │       ├── import.md                    ← import of external drafts
│   │       ├── deliberate.md                ← panel orchestration (4 rounds + synthesis)
│   │       ├── deliberate/                  ← deliberate sidecar
│   │       │   ├── panel-design.md          ← STEP 2 extracted (panel design)
│   │       │   └── templates/*.tpl          ← language-neutral skeletons (resolved against locale pack)
│   │       ├── refine.md                    ← refinement of closed runs (3 tiers)
│   │       ├── refine/
│   │       │   └── templates/*.tpl          ← specific skeletons (follow_up.md, parent-appendix.md)
│   │       └── status.md                    ← read-only view
│   ├── commands/                            ← slash commands (Claude Code) — 5 thin per-action shells
│   │   └── council-{iterate,import,deliberate,refine,status}.md
│   └── agents/                              ← custom agents (Copilot CLI + Claude Code subagents)
│       ├── council-{iterate,import,deliberate,refine,status}.agent.md ← 5 thin per-action shells
│       └── council-expert.agent.md          ← sub-agent (spawn primitive target)
├── schemas/
│   ├── problem.schema.yaml                  ← v0.5 — problem.md sections (titles/purposes resolved via locale pack) + meta.yaml.lang
│   └── deliverable.schema.yaml              ← v0.2 — deliverable shape (titles/purposes resolved via locale pack), per run
├── docs/
│   ├── copilot-portability-spike-2026-05-24.md
│   └── research/deliberate-refactor-2026-05-25/    ← research + refactor plan
└── branches/<branch>/problems/<id>/        ← local storage of problems and runs (not checked in by default)
    ├── problem.md
    ├── meta.yaml
    └── council/<YYYY-MM-DD>-<slug>/         ← one subfolder per deliberation or refinement
```

The most important architecture notes:

- **A single skill** (`council`) — no sub-skills.
- **Core + actions/**: `SKILL.md` always loaded (role + disciplines + reusable subroutines), specific procedures loaded only when the director executes an action.
- **One-level sidecars**: `actions/deliberate/templates/*.tpl` and `actions/refine/templates/*.tpl` load only when the STEP needs them. Following the Anthropic "one-level-deep" rule — we don't nest references.
- **Storage in `branches/`**: each *branch* groups related problems. Each problem can have N runs (sibling deliberations comparable to each other, or refinements as children).

## Additional documentation

To understand the system in depth:

| I want to understand... | See... |
|---|---|
| How the system is architected (components, lifecycle, invariants, failure modes, provider abstraction) | [`ARCHITECTURE.md`](ARCHITECTURE.md) |
| How to instruct the agent (rules and disciplines) | `CLAUDE.md` (Claude Code) or `.github/copilot-instructions.md` (Copilot CLI) |
| The core and procedures | `.claude/skills/council/SKILL.md` + `.claude/skills/council/actions/*.md` |
| How it was made portable to Copilot CLI | `docs/copilot-portability-spike-2026-05-24.md` |
| How it was refactored (with prior research by 4 parallel agents) | `docs/research/deliberate-refactor-2026-05-25/` |
| Artifact schemas | `schemas/problem.schema.yaml`, `schemas/deliverable.schema.yaml` |

## Inspirations and references

Council borrows patterns from academic literature and production frameworks:

- **Multi-Agent Debate** (Du et al., 2023, ICML 2024) — the idea of N agents proposing and revising across rounds. [arXiv:2305.14325](https://arxiv.org/abs/2305.14325)
- **Multi-Agents Debate / Encouraging Divergent Thinking** (Liang et al., 2023) — the notion of *degeneration-of-thought* and *moderate, not maximal disagreement*. [arXiv:2305.19118](https://arxiv.org/abs/2305.19118)
- **AgentVerse** (Chen et al., 2023, ICLR 2024) — dynamic expert recruitment per problem. [arXiv:2308.10848](https://arxiv.org/abs/2308.10848)
- **MetaGPT** (Hong et al., 2024, ICLR) — structured outputs over free chat to reduce cascading hallucination. [arXiv:2308.00352](https://arxiv.org/abs/2308.00352)
- **Anthropic Agent Skills standard** — skill format, progressive disclosure, 500-line limit for SKILL.md, one-level-deep rule. [Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- **OpenAI Agents SDK** `RECOMMENDED_PROMPT_PREFIX` — "common header + role-specific delta" pattern that council applies in its `EXPERT_SPAWN_HEADER`.

Council distinguishes itself from all of them by its combo: **dynamic panel + shuttle diplomacy in debate + impartial moderator separated + filesystem-as-state + round barrier for user escalations**, implemented as **a single markdown skill** (no framework, no Python) runnable in Claude Code or Copilot CLI.

## Current state

- **v2.7 (2026-05-25)**: wrappers refactored to **thin shells**. 5 slash commands + 5 per-action custom agents, each ~5 lines (frontmatter + 1 activation line) — eliminates the ~70% body duplication that existed pre-v2.7 without losing per-action autocomplete (Claude Code) and description-based dispatch (Copilot CLI). Uniform naming: `council-iterate`, `council-import`, `council-deliberate`, `council-refine`, `council-status`. The skill also auto-creates an umbrella `/council` that accepts `<action>` as the first arg.
- **v2.6 (2026-05-25)**: runtime made multilingual via a **locale pack** + safety hardening. Locale pack at `.claude/skills/council/locales/<lang>.yaml`. Templates `.tpl` neutralized (`{{H:key}}` / `{{I:key}}` / `{{R:key}}` placeholders); all spawn-prompt bodies (`EXPERT_SPAWN_HEADER`, round deltas, moderator prompts) and few-shot examples per output role live in the locale pack. New REUSABLE SUBROUTINE `LANGUAGE_DISCIPLINE(lang, role)` appended at the END of every spawn prompt — output-language directive + canonical vocabulary pinning + one in-language few-shot (research basis: language-confusion benchmark, lost-in-the-middle, hard-to-easy ordering). **Safety**: new REUSABLE SUBROUTINE `VERIFY_OUTPUTS(expected_paths)` with retry-and-respawn logic and an INVIOLABLE rule forbidding director-side fabrication of sub-agent outputs — addresses the spawn-primitive race observed in real runs (Copilot CLI issue #1324). Two locales ship: `es`, `en`. `meta.yaml` gains a `lang:` field (schema 0.4).
- **v2.5 (2026-05-25)**: portability to Copilot CLI completed, refactor of `deliberate.md` applied (467 → 272 lines, -42%), refactor of `refine.md` applied, subroutines centralized in SKILL.md, all documentation translated to English.
- See `CLAUDE.md` for the detailed per-version changelog.

## License

To be defined — personal development repo.
