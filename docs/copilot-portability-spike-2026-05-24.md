# Spike — council portability to Copilot CLI

> Status: **implemented** — v2.4 of the system, 2026-05-24. This document records the design applied so that the council runs natively on **Copilot CLI** in addition to Claude Code, and the results of the architectural validation spike.

## 1. Motivation

The council was born on top of Claude Code and depends heavily on the `Task` tool (spawning sub-agents with isolated fresh context). The initial hypothesis was that porting to Copilot CLI would require a major refactor — pulling the "director" out of the model and putting it in code that orchestrates external invocations.

After research, we discovered that **Copilot CLI has functional parity with `Task`**: `/fleet` + custom agents provide exactly the same guarantees (isolated context, fresh spawn, no peer-to-peer, shared filesystem). This turned portability into a cheap nomenclature refactor + a new set of wrappers.

## 2. Applied design

### 2.1 Neutral layer

The council core (SKILL.md + actions/*.md + schemas/) **is not duplicated per provider**. What was done:

- **New `SPAWN PRIMITIVE` section** in `SKILL.md` that normalizes the nomenclature: the procedure says "spawn primitive" in the abstract, and the director knows how to translate to the environment's primitive (`Task` in Claude Code, `/fleet` + `council-expert` in Copilot CLI).
- **References to "Task tool"** in `actions/*.md` replaced with "spawn primitive" or with explicit dual mention ("`Task` in Claude Code; `/fleet` + `council-expert` in Copilot CLI").
- **Preserved invariants** (isolated context, no peer-to-peer, round barrier on escalations, moderator autonomy) — they belong to the SKILL, not the provider.

### 2.2 Duality of wrappers

| Mechanism | Claude Code | Copilot CLI |
|---|---|---|
| `iterate` mode | `/council-problem-iterate` (slash command) | `--agent council-problem-iterate` |
| `import` mode | `/council-problem-import` | `--agent council-problem-import` |
| `deliberate` mode | `/council` | `--agent council-deliberate` |
| `refine` mode | `/council-refine` | `--agent council-refine` |
| `status` mode | `/council-status` | `--agent council-status` |

The wrappers are **thin** (~30 lines each): they read the SKILL, delegate to the corresponding action. The logic lives only in `actions/*.md`.

### 2.3 Generic `council-expert` sub-agent

In Claude Code experts are personas injected as prompts to `Task` per run. In Copilot CLI custom agents are pre-registered. To avoid registering 5 agents per deliberation, there is **a single generic `council-expert.agent.md`** that receives the persona as part of the prompt (`persona: <path>`, `output: <path>`, `round: <X>`, paths to read, task). It is `user-invocable: false` — only the director calls it.

council-expert frontmatter:
```yaml
name: council-expert
tools: ["read", "search", "edit", "write", "web", "github/*"]
model: claude-opus-4.6
user-invocable: false
```

### 2.4 Project configuration

- **`.github/copilot-instructions.md`**: project instructions for Copilot CLI, with the invocation table for each mode.
- **`.copilot/settings.json`**: `allowedUrls: ["*"]` permissive during the spike. Restrict in continued use to specific domains if desired.

## 3. Changes to the system

### 3.1 Modified files

| File | Change |
|---|---|
| `.claude/skills/council/SKILL.md` | Added `SPAWN PRIMITIVE` section (provider-agnostic). Generalized 10+ references to "Task tool". Frontmatter description neutralized. |
| `.claude/skills/council/actions/deliberate.md` | Headers of STEP 3/4/6/7 + 5 internal invocations generalized to "spawn primitive". Dual mention of the primitive in prompts where applicable. |
| `.claude/skills/council/actions/refine.md` | Same generalizations as `deliberate.md` (Tier 1 moderator + Tier 2 compressed rounds). |
| `.claude/skills/council/actions/iterate.md` | A single line: "Do NOT spawn sub-agents (via the environment's spawn primitive)" instead of "(`Task` tool)". |
| `.claude/commands/council.md` and `council-refine.md` | Updated footers: "v2.4 uses the environment's standard spawn primitive". |
| `CLAUDE.md` | v2.4 version added. `Structure` section now shows `.claude/agents/`. `Prerequisite` section describes the two primitives. |

### 3.2 New files

```
.claude/agents/
├── council-expert.agent.md           # generic sub-agent (user-invocable: false)
├── council-deliberate.agent.md       # Copilot wrapper for deliberate mode
├── council-problem-iterate.agent.md  # Copilot wrapper for iterate mode
├── council-problem-import.agent.md   # Copilot wrapper for import mode
├── council-refine.agent.md           # Copilot wrapper for refine mode
└── council-status.agent.md           # Copilot wrapper for status mode

.github/
└── copilot-instructions.md           # analogous to CLAUDE.md for Copilot CLI

.copilot/
└── settings.json                     # allowedUrls + includeCoAuthoredBy
```

## 4. Validation spike (2026-05-24)

### 4.1 Candidate problem

`paneles-solares/2026-05-22-0644` with closed sibling run `2026-05-22-comparativa-paneles` (original Claude run with a 149-line outcome.md and 11 commercial models verified spec-by-spec).

### 4.2 Run setup

New run: `2026-05-24-spike-copilot/`. Inputs inherited from the sibling run: `panel.md`, `deliverable.md`, `hypothesis.md`, `experts/<5>/persona.md`. Decision: **do not inherit `user_directives.md`** to maintain parity with the state of the original run at the moment of Round A.

### 4.3 Two-step execution

**Step 3 — Validate Round A (external orchestration)**. I (Claude Code) launched 5 parallel `copilot --agent council-expert -p "..."` invocations with manually constructed prompts (persona path + output path + paths to read + task). Cost: **15 premium requests, ~3 minutes**. The 5 proposals produced are high quality (real commercial models, coherent figures, critical analysis of the user's hypothesis with domain argument).

**Step 4 — End-to-end deliberation (Copilot director)**. A single `copilot --agent council-deliberate -p "..."` invocation with pre-approved decisions embedded (resume the run, accept inherited panel, do not block on escalations, do not copy user_directives). The director executed:

- Detection of the incomplete run → resumed correctly.
- Re-confirmation of the inherited panel → accepted.
- Round B (5 critiques via `/fleet`).
- Detection of 3 conflicts (panel-format, real-prices, premises-to-validate).
- Shuttle diplomacy in Round C (2 turns per conflict × 3 = 6 files).
- Lead's debate synthesis (`debate_mediated.md`).
- Round D (5 final_positions via `/fleet`).
- Construction of `lead_notes.md` as a bridge.
- Invocation of the moderator → `debate_summary.md` + `outcome.md`.
- Update of `run_meta.yaml` to `run_status: complete`.

Cost: **3 premium requests, ~11 minutes, 1.3M tokens (1.1M cached)**.

### 4.4 Architectural coverage

| Aspect | Validated | How |
|---|---|---|
| SKILL load from `.claude/skills/` | ✅ | Director read SKILL.md + actions/deliberate.md |
| Incomplete run detection | ✅ | Resumed `spike-copilot` with proposals/ but no critiques/ |
| Panel + experts inheritance from sibling run | ✅ | Without modifying inputs |
| Parallel spawn primitive (`/fleet`) | ✅ | 5 critiques + 5 final_positions produced |
| Sequential shuttle diplomacy | ✅ | 6 files in debate/ |
| Lead → moderator synthesis | ✅ | lead_notes.md + debate_summary.md + outcome.md |
| Persistence of escalations as barrier | ✅ | escalations/round_a.md with questions marked [NO DISPONIBLE] |
| Context isolation between sub-agents | ✅ | Each expert only saw what the director passed |
| Web access (web_fetch + github MCP web search) | ✅ | Real commercial models in all proposals |
| Spanish language | ✅ | All files |
| Run closure (`run_status: complete`) | ✅ | run_meta.yaml updated |

**Architectural verdict: TOTAL parity with Claude Code. Zero deltas.**

### 4.5 Qualitative quality difference observed

The spike's outcome.md is **~60% of the volume** of the original (90 vs 149 lines, 7 models vs 11). The difference is attributable to the **non-interactive mode of the spike**:

1. **No inherited `user_directives.md`**. The original run had "deep, non-superficial searches" in Round A; the spike did not.
2. **13 Round A escalations persisted but unanswered**. In the original the user responded → enriched the outcome with anchor in El Perelló, white stones, bifacial scenario.
3. **No feedback on conflicts**. In the original the user said "less weight to the economist", "investigate who is right", "more data better", "interest in premium range" → enriched the outcome with extra dimensions.

The spike's director **self-documented this asymmetry** explicitly in `lead_notes.md` and `debate_mediated.md` ("non-interactive spike, advancing without user feedback"). Exemplary behavior — it respected the SKILL rule "the user has the last word" and did not invent answers.

With the user in the loop, quality would foreseeably be on par (same Opus 4.6 model, same architectural flow, only the input differs).

## 5. Extra findings

### 5.1 Billing economy in Copilot CLI

Internal invocations via `/fleet` or `--agent` **within a director session do NOT multiply the premium cost**. A complete deliberation (17+ internal spawns) costs ~3 premium requests within the parent session. This changes the expected economy:

- External orchestration (Step 3): 5 experts × 3 premiums = 15 premiums.
- Director with internal `/fleet` (Step 4): 3 premiums total.

**Conclusion**: for continued use in Copilot CLI, it is always preferable to invoke the director (`--agent council-deliberate`) — do not orchestrate externally.

### 5.2 Models available in Copilot CLI

Confirmed (via `copilot help config` + real tests in the user's account):
- ✅ `claude-opus-4.6` — used in the spike, parity with Claude Code.
- ✅ `claude-sonnet-4.6`, `claude-sonnet-4.5`, `claude-haiku-4.5`.
- ✅ GPT-5.4 / GPT-5.4 mini (default for sub-agents if another is not forced).
- ❌ `claude-opus-4.7` — listed in `--help` but not available on Pro/Pro+ plans.

Relevant feature flag (seen in `~/.copilot/config.json`):
```
copilot_cli_gpt_5_4_for_subagents: true
copilot_cli_opus_medium_effort_default: true
```

By default sub-agents run on GPT-5.4. If parity with Claude Code is desired, force `model: claude-opus-4.6` (or whichever applies) in the frontmatter of `council-expert.agent.md` and the director wrappers. That is what was done in the spike.

### 5.3 Resolved gotcha: `model: auto` is not valid

The first `--agent council-status` test revealed a warning: `Custom agent "council-status" specifies model "auto" which is not available; using "claude-sonnet-4.6" instead`. Resolved by explicitly setting `model: claude-opus-4.6` in the 6 agents.

### 5.4 No modification to the YAML schema or storage

The `schemas/*.yaml` and the `branches/<branch>/problems/<id>/...` structure required no changes. They are provider-agnostic by design.

## 6. Spike limitations

1. **Non-interactive mode**: user decisions (panel checkpoint, escalations, conflict feedback) were pre-approved in the initial prompt. In real use with an interactive session, this is covered by the normal SKILL flow.
2. **Only deliberation was tested** (`council-deliberate`). The other 4 modes (iterate, import, refine, status) are refactored but **not tested end-to-end** in Copilot CLI.
3. **Only one candidate problem** (paneles-solares). It is not known whether different problem patterns (more experts, denser conflicts, refinements) exhibit new behaviors.

## 7. Recommendations for continued use

### 7.1 To run the council on Copilot CLI

```bash
cd /Users/vssnake/_dev/cc-prompts/council
copilot --agent council-deliberate
```

At the first prompt, type: `<branch> <problem-id> [<slug>]`. The director drives the flow, asking the user at checkpoints. **Do not use `-p` for real sessions** — the flow requires conversation.

### 7.2 Refining the configuration

- `.copilot/settings.json` is currently set with `allowedUrls: ["*"]` (permissive from the spike). For continued use, consider restricting to manufacturer / distributor domains that appear in the project's problems.
- If mixing models by role is desired (e.g., Opus for directors, Sonnet for experts to bound cost), adjust `model:` in each `.agent.md` separately.

### 7.3 Pending manual tests (for the user)

The spike validated the critical path (deliberate). The tests worth doing manually to fully close out portability:

| Test | How | What we validate |
|---|---|---|
| `council-problem-iterate` end-to-end | `copilot --agent council-problem-iterate` → create a new problem in chat | Conversational capture works, immediate persistence, status draft→open |
| `council-problem-import` with external file | `copilot --agent council-problem-import` → pass a .md draft | Schema mapping works, treatment as untrusted data |
| `council-refine` Tier 1 | On a closed run, launch a clarification | Tier 1 detection, single moderator, NEEDS-TIER-2 fallback |
| `council-refine` Tier 2 | On a closed run, launch a refinement with new data | Panel inheritance, compressed rounds, appendix on parent |
| `council-deliberate` with user in the loop | Same as the spike but answering escalations in chat | Outcome quality under real conditions (expected: on par with the original Claude run) |
| Resumption of an interrupted run | Kill `copilot` mid-Round B, relaunch `council-deliberate` | The director detects and offers to resume / discard / start another |
| Comparison of outcomes between Sonnet and Opus | Same problem, launch two deliberations with different `model:` in `council-expert.agent.md` | Cost vs quality |

## 8. State of the system after the spike

- **v2.4 GA in local production.** The council runs natively on Claude Code and Copilot CLI.
- **Shared storage**: the same `branches/...` serves both providers. Runs produced in one are readable from the other.
- **Next step**: the manual tests in §7.3 whenever the user wants to execute them.

## 9. Spike files (reference)

- New run: `branches/paneles-solares/problems/2026-05-22-0644/council/2026-05-24-spike-copilot/`
- Reference run (original Claude): `branches/paneles-solares/problems/2026-05-22-0644/council/2026-05-22-comparativa-paneles/`
- Copilot invocation logs: `/private/tmp/claude-501/.../tasks/b*.output` (ephemeral, not committable)

---

## 10. Spike v2 — interactive run with user in the loop (2026-05-25)

Second iteration of the spike. Unlike the one in §4 (non-interactive, inherited panel), here **`council-deliberate` was executed end-to-end under real conditions**: the user, in another chat with Copilot CLI, was pasting the director's outputs and back the responses that Claude Code (in this chat) indicated. Validation of **flow + discipline**, not just architecture.

### 10.1 Setup

- Run: `branches/paneles-solares/problems/2026-05-22-0644/council/2026-05-24-paneles-solares-finca-v2/`.
- Same `problem.md` as the non-interactive spike and as `comparativa-paneles`. Did NOT inherit panel or deliverable or hypothesis from the sibling run — everything was iterated from scratch.
- User's hypothesis (literal): *"preferencia por los paneles con más watts"* (preference for the panels with more watts).

### 10.2 Result

Run completed (`run_status: complete`). Technical convergence with `comparativa-paneles`: **same recommended brand (JA Solar), same technology (TOPCon N-type), same €/Wp range**. The 5 experts covered the 4 rounds + impartial moderator synthesis. Different outcome form (more prescriptive, less didactic).

### 10.3 Catalog of deviations observed vs SKILL

Sorted by severity. Each one motivated a concrete change to the SKILL (§10.4).

| # | Deviation | Where | Severity |
|---|---|---|---|
| 1 | The `persona.md` of the friction archetype (`consultor-practico`) is written as a **constructive specialist**, not as an adversarial archetype: voice in positive imperative, no questions; action heuristics, not lenses of doubt. | STEP 2 (persona generation) | **High** |
| 2 | The distillation of the user's hypothesis (in STEP 1.5 branch (a)) included two leaks: the figure `7 kW` dragged in from `problem.md` and an analytical point (`"el precio por vatio es el indicador relevante"` / "price per watt is the relevant indicator") that the user did not say. | STEP 1.5 | **High** |
| 3 | The `outcome.md` expanded scope beyond `deliverable.md`: added costs of the complete system (inverter, BOS, wiring, payback) that `panel.md` declares as "out of scope". | STEP 7 (moderator) | **Medium** |
| 4 | The friction archetype was named `consultor-practico` (noun of functional role) instead of `<failure-mode>-<dimension>`. Combined with #1, this explains the drift — the name auto-prompts the persona. | STEP 2 (naming) | **Medium-High** |
| 5 | Round A proposals give ranges `~110-130 €`, `~22.4 %` without distributor, without date, without URL. The reference run had figures verified spec-by-spec with distributor and date. | STEP 3 (Round A spawn) | **Medium** |
| 6 | The 5 questions in `escalations/round_a.md` have **massive overlap**: 3 experts ask "climate zone", 3 ask "DIY or professional", 3 ask "space". The user answered the same thing several times. | STEP 3 (Round A close) | **Low-Medium** |
| 7 | v2's `panel.md` has a single generic sentence as "Panel notes". No traceability of which expert covers which variable or why dimensions are omitted. | STEP 2 (panel.md) | **Low** |
| 8 | The Copilot director **skipped STEP 1** (slug) — went straight to STEP 1.5. The user had to correct manually. | STEP 1 | **Low** (UX) |

### 10.4 Changes applied to the SKILL in response

All changes live in `actions/deliberate.md`. They are additive — they do not break previous runs.

| Deviation | Change | Modified section |
|---|---|---|
| #2 | **"Anti-leak guard"** section with good + bad example based on the `7 kW` leak. Prohibits figures from problem.md, analytical reformulations and inferred consequences in the hypothesis distillation. | STEP 1.5 |
| #8 | Slug capture **explicitly blocking** — *"WAIT FOR USER INPUT — do NOT infer a slug from the problem.md, do not reuse the problem-id, do not advance with a placeholder"*. | STEP 1 |
| #4 | **Two naming templates**: specialists = `<role>-<domain>`; friction archetypes = `arquetipo-` or `<failure-mode>-<dimension>` — never a noun of functional role. | STEP 2 |
| #1 | **Mandatory line** in friction archetype `persona.md`: *"Function: question, do not propose. Do NOT formulate constructive recommendations of your own."* + self-detection criterion: if you find yourself writing "buy X" / "choose Y" in a friction persona → start over. | STEP 2 |
| #7 | **Structured template for "Panel notes"** with 3 mandatory sub-sections: owner per variable, omitted dimensions with reason, user decisions on composition. | STEP 2 |
| #5 | **Mandatory evidence discipline** in Round A: each concrete figure carries `[source: URL/manufacturer spec/distributor + date]`, or is marked `est.` with `[verify before deciding]`. Do not mix verified and estimated without distinguishing. | STEP 3 (prompt) |
| #6 | **Deduplication + multiple attribution** before batching questions to the user. Also applies to Rounds B/C/D. | STEP 3 (escalations) |
| #3 | **Hard scope guard for the moderator** — `outcome.md` respects EXACTLY the sections of `deliverable.md`; out-of-scope content is discarded even if some expert touched on it. + Mandatory **post-synthesis audit**: 1:1 mapping against `deliverable.md` and per-cell evidence verification. | STEP 7 (moderator prompt) |

### 10.5 Pending validation

The changes are changes to the SKILL — their validation requires running `council-deliberate` with a new problem after the changes and verifying that deviations #1, #2 and #4 do not reappear. Suggested: re-run ONLY STEPS 1, 1.5 and 2 over the same `paneles-solares/2026-05-22-0644` with a different slug, compare against v2:
- Does the director wait for the slug? (#8)
- Does the distillation of "preferencia por paneles con más watts" stay at 1 faithful point without leak? (#2)
- Does the friction archetype receive a `<mode>-<dimension>` name or `arquetipo-` prefix? (#4)
- Does the friction persona have the line-guard at the start of "Limitations"? (#1)

If all four pass, the changes are effective. If any fails, iterate on the wording of the guards.

### 10.6 Spike v2 files (reference)

- v2 run: `branches/paneles-solares/problems/2026-05-22-0644/council/2026-05-24-paneles-solares-finca-v2/`
- 3-way comparison (v2 ↔ comparativa-paneles ↔ spike-copilot) executed manually in chat 2026-05-25; see history of session `cadd48e4-b706-47ad-95e7-c77ca129f669` for process-by-process detail.

---

## 11. Errata + path migration (2026-05-25)

**Issue surfaced**: a Claude Code `/council-deliberate` run on `finca-perello/renovacion-energetica` failed at Round A because Claude Code was exposing the Copilot CLI agents living in `.claude/agents/` as its own `subagent_type` options. The director (the LLM) picked `subagent_type: "council-expert"` instead of the `general-purpose` prescribed by `SKILL.md` for Claude Code. The spawn failed because the agent's frontmatter declares `model: claude-opus-4.6` — a valid Copilot CLI model identifier but not a valid Claude Code one (Claude Code uses `claude-opus-4-6` / `claude-opus-4-7`).

**Root cause analysis**:
1. `.claude/agents/` is **not** a documented Copilot CLI path. Per the official Copilot CLI docs ([create-custom-agents-for-cli](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/create-custom-agents-for-cli), [invoke-custom-agents](https://docs.github.com/en/copilot/how-tos/copilot-cli/use-copilot-cli/invoke-custom-agents)), Copilot CLI scans only three locations: `~/.copilot/agents/` (user), `.github/agents/` (project), `.github-private/agents/` (org). The choice in this spike (§3.2 above) to place Copilot CLI custom agents under `.claude/agents/` was incorrect by Copilot CLI's specification.
2. Whether the spike actually worked from `.claude/agents/` (undocumented Copilot CLI behavior, transient version detail, or via symlink/copy that was later removed) is not auditable from the recorded artifacts. The historical run `2026-05-24-spike-copilot/` exists, but the path resolution at the time of execution cannot be reconstructed.
3. Claude Code, on the other hand, **does** read `.claude/agents/` natively and exposes those files as `subagent_type` options — that's what created the LLM-drift on 2026-05-25.

**Migration applied (2026-05-25)**:
- `mv .claude/agents/*.agent.md .github/agents/` — all 6 agent files relocated to the canonical Copilot CLI project-level path.
- `.claude/agents/` removed (empty).
- Updated references in `CLAUDE.md`, `ARCHITECTURE.md`, `.claude/skills/council/SKILL.md`, `.github/copilot-instructions.md`. Historical changelog entries in `CLAUDE.md` (v2.7 / v2.4) left intact as the record of the prior path.
- Section §3.2 of this document remains as the original spike record; this §11 supersedes it on the path question.

**Why the move resolves both problems**:
- Copilot CLI gets the agents in a documented location (no reliance on undocumented behavior).
- Claude Code stops exposing them as `subagent_type` options (Claude Code does not scan `.github/agents/`). The LLM-drift vector is closed at the harness layer rather than via prose discipline in `SKILL.md`.

**Verification owed**: re-run `copilot --agent council-status` after the move to confirm Copilot CLI auto-dispatch still finds the directors at the new path. Expected: yes, per the official docs. Not executed in this errata; user to verify in a Copilot CLI session.

**Lesson for the system**: when authoring multi-provider artifacts, put each provider's files in **that provider's canonical location** — not in a shared directory that happens to contain artifacts of one of them. The convenience of "all agents in one place" hid a real conflict for months.
