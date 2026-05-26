# Project: Council

Expert-council system to deliberate problems in any domain. Conversational problem capture + dynamic panel of 4-6 experts that debate across 4 rounds (proposals → critiques → lead-mediated debate → final positions) and produce a recommendation synthesized by an impartial moderator. A closed deliberation can be refined later without redoing it.

> **Language note**: this CLAUDE.md, the SKILL.md core, and the `actions/*.md` procedures are written in English (project rule for maintainability since v2.5). **Runtime language is configurable per problem**: it lives in `meta.yaml.lang` (set by `iterate`/`import`, or via `--lang=`), sticky for the problem's lifetime. Two locales ship today (`es`, `en`) under `.claude/skills/council/locales/`; the director, all spawn prompts and all materialized artifacts use that locale at runtime. Legacy problems without `lang:` default to `es`.

v2.8 (2026-05-25): **Copilot CLI agents moved to canonical path `.github/agents/`** (previously `.claude/agents/`). The prior location was outside Copilot CLI's documented agent search paths (`~/.copilot/agents/`, `.github/agents/`, `.github-private/agents/`) and simultaneously caused LLM-drift in Claude Code: Claude Code scans `.claude/agents/` natively and exposed the Copilot-only `council-expert` agent as a `subagent_type` option, leading directors to pick it instead of `general-purpose` (the prescribed primitive for Claude Code per `SKILL.md` SPAWN PRIMITIVE). The move closes both issues at the harness layer. Detail and root cause in `docs/copilot-portability-spike-2026-05-24.md` §11 (errata). No SKILL/actions/schemas behavioral change.

v2.7 (2026-05-25): **Wrappers refactored to thin shells.** Restored 5 slash commands (`.claude/commands/`) + 5 per-action custom agents (`.claude/agents/`), but each is now ~5 lines (frontmatter + 1 line that activates the skill with the action token) — no more 30-line body duplication. Uniform naming: `council-iterate`, `council-import`, `council-deliberate`, `council-refine`, `council-status`. Action token internally remains `status` (`actions/status.md` unchanged). The skill auto-creates an umbrella `/council` that accepts `<action>` as the first arg. Copilot CLI's description-based auto-dispatch benefits from 5 specific descriptions instead of one generic. `council-expert.agent.md` (sub-agent) stays unchanged. The brief mid-day excursion that consolidated everything into 1 entry point per provider was reverted after testing showed it hid the per-action autocomplete / dispatch hints.

v2.6 (2026-05-25): runtime is multilingual. **Locale pack** at `.claude/skills/council/locales/<lang>.yaml` resolves headings, instructional placeholders, hard rules, spawn-prompt bodies (`EXPERT_SPAWN_HEADER`, `LANGUAGE_DISCIPLINE`, moderator prompts, round deltas), few-shot examples per output role, AND schema-driven section titles/purposes (SCHEMA category, see below). `.tpl` files are language-neutral (`{{H:key}}` / `{{I:key}}` / `{{R:key}}` placeholders). New REUSABLE SUBROUTINE `LANGUAGE_DISCIPLINE(lang, role)` appended at the END of every spawn prompt (output-language directive + canonical vocabulary pinning + one in-language few-shot per role — research basis: language-confusion benchmark, lost-in-the-middle, hard-to-easy ordering). **Safety**: new REUSABLE SUBROUTINE `VERIFY_OUTPUTS(expected_paths)` with retry-and-respawn logic and an INVIOLABLE rule forbidding director-side fabrication of sub-agent outputs — addresses the `/fleet` spawn-primitive race observed in real runs (Copilot CLI issue #1324). **Schemas multilingualized**: `problem.schema.yaml` bumps to v0.5, `deliverable.schema.yaml` bumps to v0.2 — each section's `title` + `purpose` replaced by a `locale_ref:` that resolves into `SCHEMA.<schema>.<name>` under `locales/<lang>.yaml`. `meta.yaml.schema_version` bumps to `0.5` for new problems (adds `lang:` + schema-as-locale-ref). Renamed `apendice-padre.md.tpl` → `parent-appendix.md.tpl`. Template frontmatter `nombre:` → `name:` for new personas (legacy `nombre:` in existing `branches/` files remains valid — the LLM reads frontmatter as text).

v2.5 (2026-05-25): documentation and procedures translated to English (levels 1+2). Runtime language unchanged at the time (level 3 — Spanish only). Renamed subroutine `CIERRE_DE_RONDA` → `CLOSE_ROUND`. Subroutines centralized in SKILL.md as "REUSABLE SUBROUTINES" section. Templates `.tpl` initially kept in Spanish — now superseded by v2.6 locale-neutralization.

v2.4 (2026-05-24): adds **portability to Copilot CLI**. Dual `.claude/commands/` (Claude Code slash commands) + `.claude/agents/` (Copilot CLI custom agents). SKILL.md incorporates the provider-agnostic `SPAWN PRIMITIVE` section that normalizes `Task` tool (Claude Code) and `/fleet` + `council-expert` (Copilot CLI); the `actions/*.md` and `schemas/` require no per-provider changes. Architectural validation spike completed — full parity. See `docs/copilot-portability-spike-2026-05-24.md`.

v2.3 (2026-05-22): adds the `status` action — read-only view of state (branches, problems, runs, what's incomplete) that orients toward the next command. Problems now have **human names** (a slug the user chooses) instead of timestamp-id. `deliberate` and `refine` now **detect and resume interrupted runs** instead of leaving them orphaned. The core's `WHAT NOT TO DO` block is reduced to an index — specific rules live in each `actions/*.md`.

v2.2 (2026-05-22): adds the `refine` action — iterates on a closed deliberation in tiers (1 clarification / 2 refinement / 3 out of scope), reusing the parent run's panel and verified work. SKILL.md is split into **core + action files** (`actions/`): the core — role, data model, cross-cutting disciplines — always loaded; each action's procedure is read on demand.

v2.1 (2026-05-12): no agent-teams, no peer-to-peer between experts, no persistent teammates. Each round spawns fresh sub-agents with the environment's spawn primitive (`Task` tool in Claude Code; `/fleet` + the `council-expert` custom agent in Copilot CLI). The final synthesis is produced by an impartial moderator sub-agent fed by a `lead_notes.md` bridge that the lead writes at the debate's close.

## Structure

```
council/
├── .claude/
│   ├── skills/council/
│   │   ├── SKILL.md                         # CORE: role, data model, cross-cutting disciplines,
│   │   │                                    #   reusable subroutines, routing (STEP 0). Always loaded.
│   │   ├── locales/                         # v2.6 — locale pack (multilingual runtime)
│   │   │   ├── _spec.yaml                   # contract: required keys per category (H/I/R/S/F)
│   │   │   ├── es.yaml                      # Spanish locale
│   │   │   └── en.yaml                      # English locale
│   │   └── actions/                         # STEP-by-STEP procedures, read on demand
│   │       ├── iterate.md
│   │       ├── import.md
│   │       ├── deliberate.md
│   │       ├── deliberate/
│   │       │   ├── panel-design.md          # STEP 2 extracted from deliberate.md
│   │       │   └── templates/*.tpl          # language-neutral skeletons (resolved against locale pack)
│   │       ├── refine.md
│   │       ├── refine/
│   │       │   └── templates/*.tpl          # follow_up.md + parent-appendix.md (language-neutral)
│   │       └── status.md
│   └── commands/                            # slash commands (Claude Code) — thin shells per action
│       ├── council-iterate.md
│       ├── council-import.md
│       ├── council-deliberate.md
│       ├── council-refine.md
│       └── council-status.md
├── .github/
│   ├── copilot-instructions.md              # analogous to CLAUDE.md for Copilot CLI
│   └── agents/                              # custom agents (Copilot CLI — canonical project-level path per Copilot CLI docs)
│       ├── council-iterate.agent.md         # 5 thin per-action director agents, mirror of commands/
│       ├── council-import.agent.md
│       ├── council-deliberate.agent.md
│       ├── council-refine.agent.md
│       ├── council-status.agent.md
│       └── council-expert.agent.md          # generic sub-agent, invoked by the director via /fleet
├── schemas/
│   ├── problem.schema.yaml                  # v0.5 — problem.md sections (locale-resolved) + meta.yaml.lang
│   └── deliverable.schema.yaml              # v0.2 — deliverable shape (locale-resolved), per run
├── docs/                                    # design documents + research
└── branches/<branch>/problems/<id>/        # local storage per branch+problem
    ├── problem.md                           # stable
    ├── meta.yaml                            # status: draft | open; lang: es|en|...
    └── council/
        └── <YYYY-MM-DD>-<slug>/             # one subfolder PER run (deliberation or refinement)
            ├── run_meta.yaml                # kind, parent_run, tier, trigger...
            ├── hypothesis.md                # [deliberation] iterated or auto-generated
            ├── follow_up.md                 # [refinement] trigger + new info + anchor
            ├── deliverable.md               # deliverable shape
            ├── panel.md                     # (inherited from parent in a refinement)
            ├── experts/<expert>/persona.md
            ├── proposals/expert_*.md        # Round A / A'  (fresh spawn primitive)
            ├── critiques/expert_*.md        # Round B / B'  (fresh spawn primitive)
            ├── debate/                      # Round C / C'  (lead shuttle diplomacy)
            │   └── <conflict-id>__expert_*.md
            ├── final_positions/expert_*.md  # Round D / D'  (fresh spawn primitive)
            ├── escalations/                 # user Q&A batched per round
            │   └── round_{a,b,c,d}.md
            ├── user_directives.md           # user directives (if any)
            ├── debate_mediated.md           # debate synthesis (lead)
            ├── lead_notes.md                # lead → moderator bridge
            ├── debate_summary.md            # debate synthesis (moderator, fixed format)
            └── outcome.md                   # final recommendation (moderator, shape per deliverable.md)
```

The same `<id>` (problem.md) can have N runs. A **deliberation** (`kind: deliberation`) is an independent run; multiple are siblings, comparable because they share problem.md. A **refinement** (`kind: refinement`) is a child run of another run — iterates on a closed deliberation without redoing it. A Tier 1 refinement (clarification) is a minimal run (`run_meta.yaml` + `follow_up.md` + `outcome.md`); a Tier 2 has an inherited panel and compressed rounds. When complete, the parent run's `outcome.md` receives a navigation appendix materialized from `parent-appendix.md.tpl` (heading resolved against the parent's locale — `## Refinamientos posteriores` in Spanish, `## Subsequent refinements` in English).

## Invocation

Each action has a dedicated thin wrapper per provider. Bodies are minimal (frontmatter + 1 line that activates the skill); the substantive procedure lives in `actions/<action>.md`.

- **Claude Code**: 5 slash commands (autocompleted with per-action `description` + `argument-hint`). The skill ALSO auto-creates `/council` as an umbrella fallback — `/council <action> <args>` works too.
- **Copilot CLI**: 5 user-invocable custom agents. Copilot's description-based auto-dispatch can route natural-language prompts to the right one.

| Action | Slash command (Claude Code) | Custom agent (Copilot CLI) |
|---|---|---|
| `iterate` | `/council-iterate <branch> [<id>] [--lang=es\|en]` | `--agent council-iterate -p "<branch> [<id>] [--lang=...]"` |
| `import` | `/council-import <branch> --from-file=<path> [--lang=es\|en]` | `--agent council-import -p "<branch> --from-file=<path> ..."` |
| `deliberate` | `/council-deliberate <branch> <problem-id> [<slug>]` | `--agent council-deliberate -p "<branch> <problem-id> [<slug>]"` |
| `refine` | `/council-refine <branch> <problem-id> [<parent-run-id>] [<slug>]` | `--agent council-refine -p "<branch> <problem-id> [<parent-run-id>] [<slug>]"` |
| `status` | `/council-status [<branch>] [<problem-id>]` | `--agent council-status -p "[<branch>] [<problem-id>]"` |

## Typical flow

1. `/council-iterate <branch>` — capture the problem in chat. State `draft → open`.
2. `/council-deliberate <branch> <problem-id> <slug>` — the director iterates hypothesis (skippable) + iterates deliverable (mandatory) + generates panel + spawns experts via the environment's spawn primitive round by round + mediates debate + spawns moderator → synthesis. Creates `council/<date>-<slug>/`. The problem remains `open` (you can launch another deliberation with a different slug).
3. `/council-refine <branch> <problem-id> <run-id> <slug>` — *(optional)* if a closed deliberation left something not cleanly resolved: refine that run without redoing it. Creates a child run and adds a navigation appendix to the parent's `outcome.md`.

At any moment, `/council-status [<branch>]` shows what problems and runs exist and where each one is — including if something is incomplete.

## Project rules

- **User-facing language: sticky from `meta.yaml.lang`.** Detected by `iterate`/`import` on new problems (from the user's first input, or via `--lang=`) and persisted; honored by every subsequent action. Legacy problems without the field default to `es`.
- **Experts respond in `meta.yaml.lang`** (their output files materialize in that locale via `EXPERT_SPAWN_HEADER` + `LANGUAGE_DISCIPLINE`).
- The user has the final word. If they reject something, respect it.
- Experts NEVER talk directly to the user or to each other. The director (YOU) is the only channel.
- User escalations are a **round barrier** — experts annotate questions in their file; the lead batches them at each round's close.
- The final synthesis (outcome.md + debate_summary.md) is produced by an impartial moderator sub-agent, not the lead.
- No hooks. No scripts. No sub-skills. No agent-teams. The entire system is **one skill** (`council`): a `SKILL.md` core always loaded + procedure files in `actions/` read on demand + the locale pack at `locales/<lang>.yaml`. The `actions/*.md` are NOT sub-skills — they are packaged reference files, a single skill entry.

## Prerequisite

Nothing special. v2.3+ does NOT require `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS`. The spawn primitive is the environment's standard:
- **Claude Code**: tool `Task` (subagent_type="general-purpose"). Five per-action slash commands live in `.claude/commands/council-<action>.md` (thin wrappers); the skill also auto-creates `/council` as an umbrella fallback.
- **Copilot CLI** (v1.0.51+): `/fleet` + the `council-expert` custom agent (defined in `.github/agents/`). Five per-action director agents live in `.github/agents/council-<action>.agent.md` (thin wrappers). Each activates the skill with the matching action token. The `.github/agents/` path is the canonical project-level location per [Copilot CLI docs](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/create-custom-agents-for-cli). Claude Code does NOT read this path, so the agents are not exposed as subagent_types in Claude Code — preventing accidental directors from picking them instead of `general-purpose`.

See the **SPAWN PRIMITIVE** section of `.claude/skills/council/SKILL.md` for invocation details.
