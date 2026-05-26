# ACTION · `status`  (v2.3 — read-only view of council state)

> Procedure for the `status` action. Loaded by the core (`SKILL.md`) in its STEP 0.
> Applied TOGETHER with the core: the `DATA MODEL` and `INCOMPLETE RUNS` section define
> what this procedure reads and interprets.

Invocation: `/council-status [<branch>] [<problem-id>]` (Claude Code) or `copilot --agent council-status -p "[<branch>] [<problem-id>]"` (Copilot CLI). The skill's umbrella `/council status ...` also works.

**What it is.** `status` draws the council state — which branches, problems, and runs exist and where each one is — so the user doesn't have to remember ids by heart. It is **strictly read-only**: DOES NOT create, DOES NOT modify, DOES NOT delete any file. If it detects something incomplete, it flags it and says which command would resolve it; it does not resolve it.

STEP 1 · Resolve scope

- Parse `$ARGUMENTS`:
  - **nothing** → global scope: all branches.
  - **`<branch>`** → branch scope: its problems and runs.
  - **`<branch> <problem-id>`** → problem scope: that problem in detail, with the lineage tree of its runs.
- A non-existent `<branch>` or `<problem-id>` is NOT an error: say so in plain language and list what does exist at the immediate parent level.

STEP 2 · Scan (read-only)

Walk `branches/` with read tools. Within scope:
- **Branch**: the directory name under `branches/`.
- **Problem**: each dir under `branches/<branch>/problems/`. Read its `meta.yaml` → `id`, `status`, `lang` (if missing: `es` per legacy fallback), `updated_at`. From `problem.md`, read the goal/objective section and take a summary line — **quote the user's text, do not reformulate**; if the section is still in `[GAP]`, note "objective not captured".
- **Run**: each dir under `<problem>/council/`. Read its `run_meta.yaml` → `kind`, `parent_run`, `tier`, `trigger`, `run_status`. An **incomplete** run = no `outcome.md`; a run without `run_meta.yaml` = broken / legacy run.

STEP 3 · Build the tree

- Problems ordered by `updated_at` (most recent on top).
- Runs of a problem: **deliberations** are siblings (root of the runs tree); each **refinement** hangs from its `parent_run` — nest children by walking the `parent_run` chains.
- Marks:
  - `draft` problem → `⚠ captura sin terminar`.
  - Incomplete run → `⚠ incompleto` + how far it got (derive the point from artifacts present, same criterion as the `INCOMPLETE RUNS` table).
  - Run without `run_meta.yaml` → `⚠ run roto (sin metadatos)`.

STEP 4 · Render and orient

Print the tree in chat (in Spanish) — **do not write any file**. Compact and scannable. For each item, add the natural next command:
- `draft` problem → `/council-iterate <branch> <id>` to finish capture.
- `open` problem without runs → `/council-deliberate <branch> <id> <slug>` to deliberate.
- `open` problem with complete deliberations → `/council-refine <branch> <id> <run-id> <slug>` to refine, or `/council-deliberate <branch> <id> <slug>` for another deliberation.
- Incomplete run → remind that `/council-deliberate` or `/council-refine` on that problem will offer to resume or discard.
- For another overview later → `/council-status [<branch>] [<problem-id>]` (this same action).

Guideline form (adapt to scope; literal label/heading text is in the active `lang`):

```
COUNCIL · <status-heading-in-lang>

<branch>/
  <problem-id>   ·   <status>   ·   [<lang>]   ·   "<quoted objective line>"
     ├─ <run-id>   <kind>[ · T<tier>]   <✓ complete | ⚠ incomplete: <point>>
     │    └─ <child-run-id>   refinement · T<n>   <state>
     └─ <run-id>   ...
     → <suggested command>
```

The `[<lang>]` marker is shown only when more than one locale appears across the problems in scope; for a single-locale workspace it's omitted to reduce noise.

`status` reports, does not rate: no analysis or opinion on the content of problems or outcomes.
