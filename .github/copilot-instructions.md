# Project: Council

Expert-council system to deliberate problems in any domain. Conversational problem capture + dynamic panel of 4-6 experts that debate across 4 rounds (proposals → critiques → lead-mediated debate → final positions) and produce a recommendation synthesized by an impartial moderator. A closed deliberation can be refined later without redoing it.

> You are running in **Copilot CLI**. This project has the main SKILL at `.claude/skills/council/SKILL.md` (compatible with Agent Skills standard, Copilot reads it from `.claude/skills/`). Detailed procedures are in `.claude/skills/council/actions/*.md`. The environment's spawn primitive here is **`/fleet` + the `council-expert` custom agent** (defined in `.github/agents/`), not Claude Code's `Task` tool.

> **Language note**: this file and the SKILL/actions are in English (project rule for maintainability since v2.5). **Runtime language is multilingual via a locale pack** (since v2.6) at `.claude/skills/council/locales/<lang>.yaml`. Each problem has its own `lang` (in `meta.yaml`), set on creation by `iterate`/`import` (inferred from user input or via `--lang=`) and sticky for the problem's lifetime. Two locales ship: `es` (Spanish) and `en` (English). Legacy problems without `lang:` default to `es`.

## Invoking the council (5 user-invocable custom agents)

Each director mode has its own thin user-invocable custom agent. Copilot's description-based auto-dispatch can route natural-language prompts to the matching agent.

| Mode | How to invoke |
|---|---|
| Conversational problem capture | `copilot --agent council-iterate -p "<branch> [<problem-id>] [--lang=es\|en]"` |
| Import external draft | `copilot --agent council-import -p "<branch> --from-file=<path> [--lang=es\|en]"` |
| Launch deliberation | `copilot --agent council-deliberate -p "<branch> <problem-id> [<slug>]"` |
| Refine a closed run | `copilot --agent council-refine -p "<branch> <problem-id> [<parent-run-id>] [<slug>]"` |
| View state (read-only) | `copilot --agent council-status -p "[<branch>] [<problem-id>]"` |

In interactive mode (`copilot` with no args), use `/agent` to browse and pick from the list.

The generic sub-agent for experts (`council-expert`) is NOT user-invocable: only the directors (`council-deliberate`, `council-refine`) invoke it via `/fleet` during rounds A/B/C/D or as a moderator in synthesis.

## Structure

```
council/
├── .claude/
│   ├── skills/council/
│   │   ├── SKILL.md                         # CORE: role, data model, cross-cutting disciplines,
│   │   │                                    #   reusable subroutines, routing (STEP 0). Always loaded.
│   │   └── actions/                         # STEP-by-STEP procedures, read on demand
│   │       ├── iterate.md
│   │       ├── import.md
│   │       ├── deliberate.md
│   │       ├── deliberate/                  # sidecar: panel-design.md + templates/*.tpl
│   │       ├── refine.md
│   │       ├── refine/                      # sidecar: templates/*.tpl
│   │       └── status.md
│   └── commands/                            # slash commands (Claude Code — does not apply in Copilot)
│       └── council-{iterate,import,deliberate,refine,status}.md
├── .github/
│   ├── copilot-instructions.md              # this file — project instructions for Copilot CLI
│   └── agents/                              # custom agents (Copilot CLI — canonical project-level path)
│       ├── council-{iterate,import,deliberate,refine,status}.agent.md  # 5 thin director wrappers
│       └── council-expert.agent.md          # generic sub-agent, invoked by the director via /fleet
├── schemas/
│   ├── problem.schema.yaml                  # v0.5 — problem.md sections (locale-resolved) + meta.yaml.lang
│   └── deliverable.schema.yaml              # v0.2 — deliverable shape (locale-resolved), per run
├── .copilot/
│   └── settings.json                        # per-repo config (allowedUrls: ["*"] — permissive)
└── branches/<branch>/problems/<id>/         # local storage per branch+problem
    ├── problem.md
    ├── meta.yaml                            # status: draft | open
    └── council/
        └── <YYYY-MM-DD>-<slug>/             # one subfolder PER run (deliberation or refinement)
            ├── run_meta.yaml
            ├── hypothesis.md                # [deliberation]
            ├── follow_up.md                 # [refinement]
            ├── deliverable.md
            ├── panel.md
            ├── experts/<expert>/persona.md
            ├── proposals/expert_*.md        # Round A / A'  (fresh spawn primitive)
            ├── critiques/expert_*.md        # Round B / B'  (fresh spawn primitive)
            ├── debate/<conflict-id>__expert_*.md  # Round C / C' (lead shuttle)
            ├── final_positions/expert_*.md  # Round D / D'  (fresh spawn primitive)
            ├── escalations/round_{a,b,c,d}.md     # user Q&A (round barrier)
            ├── user_directives.md
            ├── debate_mediated.md
            ├── lead_notes.md                # lead → moderator bridge
            ├── debate_summary.md
            └── outcome.md
```

> **Examples vs personal data**: `branches/cliente-renovables/` and `branches/casa-rural/` are **published reference examples** (tracked in git), not active work. Any other `branches/<x>/` is the user's personal data (excluded by `.gitignore`). When acting on a user request, check that the `<branch>` argument refers to the user's intent, not to an example branch.

The same `<id>` (problem.md) can have N runs. A **deliberation** (`kind: deliberation`) is an independent run. A **refinement** (`kind: refinement`) is a child run. A Tier 1 refinement is minimal (`run_meta.yaml` + `follow_up.md` + `outcome.md`); a Tier 2 has an inherited panel and compressed rounds. When complete, the parent run's `outcome.md` receives a navigation appendix (materialized from `parent-appendix.md.tpl` against the parent's locale — heading is `## Refinamientos posteriores` in `es`, `## Subsequent refinements` in `en`).

## Spawn primitive in Copilot CLI

The director (YOU, when invoked with `--agent council-deliberate` or `--agent council-refine`) launches sub-agents with **`/fleet` + the `council-expert` custom agent**. Rules:

- **Parallel (Rounds A/A', B/B', D/D')**: a single `/fleet` with N invocations of `council-expert`, each with its complete prompt (persona path, output path, round, paths to read, task). Sub-agents have **fresh, isolated context** — they don't see your context.
- **Sequential (Round C/C' — shuttle diplomacy)**: invocations of `council-expert` per turn. Each turn builds the prompt with the previous response from the other expert injected inline.
- **Moderator (final synthesis)**: a final invocation of `council-expert` with `round: moderator` (reads EVERYTHING + `lead_notes.md` → produces `debate_summary.md` + `outcome.md`).

Shared filesystem without lock: coordinate that each expert writes to its own path (`proposals/expert_<name>.md`, etc.) — writing to the same file from two sub-agents in parallel causes "last writer wins silently".

## Project rules

- **User-facing language: sticky from `meta.yaml.lang`** (set on the first call to `iterate`/`import`; honored by every subsequent action). All sub-agent prompts (composed via `EXPERT_SPAWN_HEADER` + `LANGUAGE_DISCIPLINE` from `.claude/skills/council/locales/<lang>.yaml`) and all materialized artifacts under `branches/` use that locale. Legacy problems without the field default to `es`.
- The user has the final word. If they reject something, respect it.
- Experts NEVER talk directly to the user or to each other. The director (YOU) is the only channel.
- User escalations are a **round barrier** — experts annotate questions in their file; the director batches them at each round's close.
- The final synthesis (`outcome.md` + `debate_summary.md`) is produced by an impartial moderator sub-agent, not the director.
- No hooks. No scripts. No sub-skills. No agent-teams. The entire system is **one skill** (`council`): a `SKILL.md` core always loaded + procedure files in `actions/` read on demand.

## Available models in Copilot CLI

Verified in `copilot help config`: `claude-opus-4.7`, `claude-sonnet-4.6`, `claude-sonnet-4.5`, `claude-haiku-4.5`, and GPT models (5.4, etc.). Custom agents declare `model: auto` or a concrete model in their frontmatter. **Note**: by default sub-agents run with a different model from the director (feature flag `copilot_cli_gpt_5_4_for_subagents`). For parity with Claude Code, force Claude in the `council-expert.agent.md` frontmatter with `model: claude-opus-4.7` or `model: claude-sonnet-4.6`.
