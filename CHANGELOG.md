# Changelog

> Per-version history of Council. Newest first.
> For runtime understanding see [`CLAUDE.md`](CLAUDE.md). For architecture see [`ARCHITECTURE.md`](ARCHITECTURE.md). For end-user usage see [`README.md`](README.md).

## v2.8 (2026-05-25)

**Copilot CLI agents moved to canonical path `.github/agents/`** (previously `.claude/agents/`). The prior location was outside Copilot CLI's documented agent search paths (`~/.copilot/agents/`, `.github/agents/`, `.github-private/agents/`) and simultaneously caused LLM-drift in Claude Code: Claude Code scans `.claude/agents/` natively and exposed the Copilot-only `council-expert` agent as a `subagent_type` option, leading directors to pick it instead of `general-purpose` (the prescribed primitive for Claude Code per `SKILL.md` SPAWN PRIMITIVE). The move closes both issues at the harness layer. Detail and root cause in `docs/copilot-portability-spike-2026-05-24.md` §11 (errata). No SKILL/actions/schemas behavioral change.

## v2.7 (2026-05-25)

**Wrappers refactored to thin shells.** Restored 5 slash commands (`.claude/commands/`) + 5 per-action custom agents (originally at `.claude/agents/`; later moved to `.github/agents/` in v2.8), but each is now ~5 lines (frontmatter + 1 line that activates the skill with the action token) — no more 30-line body duplication. Uniform naming: `council-iterate`, `council-import`, `council-deliberate`, `council-refine`, `council-status`. Action token internally remains `status` (`actions/status.md` unchanged). The skill auto-creates an umbrella `/council` that accepts `<action>` as the first arg. Copilot CLI's description-based auto-dispatch benefits from 5 specific descriptions instead of one generic. `council-expert.agent.md` (sub-agent) stays unchanged. The brief mid-day excursion that consolidated everything into 1 entry point per provider was reverted after testing showed it hid the per-action autocomplete / dispatch hints.

## v2.6 (2026-05-25)

Runtime is multilingual. **Locale pack** at `.claude/skills/council/locales/<lang>.yaml` resolves headings, instructional placeholders, hard rules, spawn-prompt bodies (`EXPERT_SPAWN_HEADER`, `LANGUAGE_DISCIPLINE`, moderator prompts, round deltas), few-shot examples per output role, AND schema-driven section titles/purposes (SCHEMA category, see below). `.tpl` files are language-neutral (`{{H:key}}` / `{{I:key}}` / `{{R:key}}` placeholders). New REUSABLE SUBROUTINE `LANGUAGE_DISCIPLINE(lang, role)` appended at the END of every spawn prompt (output-language directive + canonical vocabulary pinning + one in-language few-shot per role — research basis: language-confusion benchmark, lost-in-the-middle, hard-to-easy ordering).

**Safety**: new REUSABLE SUBROUTINE `VERIFY_OUTPUTS(expected_paths)` with retry-and-respawn logic and an INVIOLABLE rule forbidding director-side fabrication of sub-agent outputs — addresses the `/fleet` spawn-primitive race observed in real runs (Copilot CLI issue #1324).

**Schemas multilingualized**: `problem.schema.yaml` bumps to v0.5, `deliverable.schema.yaml` bumps to v0.2 — each section's `title` + `purpose` replaced by a `locale_ref:` that resolves into `SCHEMA.<schema>.<name>` under `locales/<lang>.yaml`. `meta.yaml.schema_version` bumps to `0.5` for new problems (adds `lang:` + schema-as-locale-ref). Renamed `apendice-padre.md.tpl` → `parent-appendix.md.tpl`. Template frontmatter `nombre:` → `name:` for new personas (legacy `nombre:` in existing `branches/` files remains valid — the LLM reads frontmatter as text).

## v2.5 (2026-05-25)

Documentation and procedures translated to English (levels 1+2). Runtime language unchanged at the time (level 3 — Spanish only). Renamed subroutine `CIERRE_DE_RONDA` → `CLOSE_ROUND`. Subroutines centralized in SKILL.md as "REUSABLE SUBROUTINES" section. Templates `.tpl` initially kept in Spanish — now superseded by v2.6 locale-neutralization.

## v2.4 (2026-05-24)

Adds **portability to Copilot CLI**. Dual `.claude/commands/` (Claude Code slash commands) + `.claude/agents/` (Copilot CLI custom agents — later moved to `.github/agents/` in v2.8). SKILL.md incorporates the provider-agnostic `SPAWN PRIMITIVE` section that normalizes `Task` tool (Claude Code) and `/fleet` + `council-expert` (Copilot CLI); the `actions/*.md` and `schemas/` require no per-provider changes. Architectural validation spike completed — full parity. See `docs/copilot-portability-spike-2026-05-24.md`.

## v2.3 (2026-05-22)

Adds the `status` action — read-only view of state (branches, problems, runs, what's incomplete) that orients toward the next command. Problems now have **human names** (a slug the user chooses) instead of timestamp-id. `deliberate` and `refine` now **detect and resume interrupted runs** instead of leaving them orphaned. The core's `WHAT NOT TO DO` block is reduced to an index — specific rules live in each `actions/*.md`.

## v2.2 (2026-05-22)

Adds the `refine` action — iterates on a closed deliberation in tiers (1 clarification / 2 refinement / 3 out of scope), reusing the parent run's panel and verified work. SKILL.md is split into **core + action files** (`actions/`): the core — role, data model, cross-cutting disciplines — always loaded; each action's procedure is read on demand.

## v2.1 (2026-05-12)

No agent-teams, no peer-to-peer between experts, no persistent teammates. Each round spawns fresh sub-agents with the environment's spawn primitive (`Task` tool in Claude Code; `/fleet` + the `council-expert` custom agent in Copilot CLI). The final synthesis is produced by an impartial moderator sub-agent fed by a `lead_notes.md` bridge that the lead writes at the debate's close.
