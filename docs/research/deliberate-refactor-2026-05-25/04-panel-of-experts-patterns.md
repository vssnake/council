# Report 4/4 — Panel-of-experts patterns

> Background agent investigating panel/jury/multi-agent-debate systems (academic papers + open source frameworks) to identify patterns applicable to council.

## Relevant findings per system

**1. Du et al. 2023 — "Multiagent Debate" (ICML 2024)** [paper](https://arxiv.org/abs/2305.14325) · [code](https://github.com/composable-models/llm_multiagent_debate)
N copies of the **same** model propose an answer, see those of the others, and revise it across R rounds. **Identical prompt every round** ("These are the solutions from other agents… update your answer"). No personas: diversity arises from sampling. Synthesis = majority. Radical simplicity, but expensive (N×R calls) and prone to premature convergence.

**2. Liang et al. 2023 — "Encouraging Divergent Thinking" / MAD** [paper](https://arxiv.org/abs/2305.19118) · [code](https://github.com/Skytliang/Multi-Agents-Debate)
2 polarized debaters (Affirmative/Negative) + **Judge** that decides when to stop and extracts the answer. Meta-prompt fixes the role; round prompts are **identical** but the judge introduces an **adaptive break** (cuts off when there is no more novelty). Contributes two key concepts: **Degeneration-of-Thought** (an LLM does not generate novelty on its own) and **"moderate, not maximal disagreement"** — more friction is not better.

**3. AgentVerse (Chen et al. 2023, ICLR 2024)** [paper](https://arxiv.org/abs/2308.10848) · [repo](https://github.com/OpenBMB/AgentVerse) (under refactor; stable branch `release-0.1`)
4 phases: **Expert Recruitment → Collaborative Decision-Making → Action Execution → Evaluation**. Recruitment is dynamic — an agent designer generates the list of roles from the problem. It combines **vertical and horizontal communication**. It is the closest reference to council: dynamic panel + rounds + synthesis.

**4. MetaGPT (Hong et al. 2024, ICLR)** [paper](https://arxiv.org/abs/2308.00352) · [repo](https://github.com/FoundationAgents/MetaGPT)
Fixed roles (PM, Architect, Engineer, QA) codify an **SOP (Standardized Operating Procedure)** as a pipeline. Each role = `{name, profile, goal, constraints} + Actions` with a reusable `PROMPT_TEMPLATE`. Communication via structured artifacts (PRD, design doc, code) — not free chat. Lesson: **structured outputs > free chat** to reduce cascading hallucination.

**5. ChatDev (Qian et al. 2024, ACL)** [paper](https://arxiv.org/abs/2307.07924) · [repo](https://github.com/OpenBMB/ChatDev)
"Chat chain" = sequential phases, each with an instructor/assistant pair inception-prompted via system prompts (role + objective + protocol + termination condition). Introduces **"communicative dehallucination"**: the assistant asks for clarification before acting — temporarily inverts the instructor↔assistant direction.

**6. CAMEL (Li et al. 2023, NeurIPS)** [paper](https://arxiv.org/abs/2303.17760) · [repo](https://github.com/camel-ai/camel)
**Role-playing via inception prompts**: each agent receives a task description + a single system prompt that fixes identity/constraints and then becomes autonomous. Canonical pattern for "inline persona in prompt".

**7. AutoGen GroupChat (Microsoft)** [docs](https://microsoft.github.io/autogen/stable//user-guide/core-user-guide/design-patterns/group-chat.html)
**`GroupChatManager`** = an LLM that chooses the next speaker on each turn. "Centralized orchestrator" pattern. Useful as a reference, but turn-based serial — does not fit the parallel Round A/B/D of council.

## Patterns applicable to `deliberate.md`

**P1 · Shared prompt headers + role-specific deltas (solves the "467 lines")** — MetaGPT, ChatDev and CAMEL define a `PROMPT_TEMPLATE` per Action and only vary the **delta** per round. In `deliberate.md`, the identity header ("You are `<name>`, specialist in `<role_category>`… DO NOT communicate with other experts… Spanish language…") is repeated verbatim in STEPs 3, 4, 5, 6. Extract it to a referenced block (e.g. an `EXPERT_SPAWN_HEADER` section at the top of the file, cited by each STEP). Reduces ~30-40% of the file without losing semantics. Applies to STEPs 3, 4, 5, 6.

**P2 · Adaptive break in Round C (from MAD)** — Liang 2023 cuts off the debate when there are no longer new arguments, not by a fixed counter. STEP 5.c already mentions "1-2 cycles" as a range; explicitly document the **3 termination conditions** (concedes / hold without arguments / compromise) as a mini-checklist and suppress the "typically 1-2" (no longer needed when the criterion is semantic). Applies to STEP 5.c.

**P3 · Structured outputs over free chat (from MetaGPT)** — the rule that "each concrete figure must carry `[source: …]` or `est. [verify before deciding]`" already follows this pattern. Extend it: each `proposals/expert_*.md` could have an **explicit mini-schema** (`## Position`, `## Reasons`, `## Assumptions`, `## Questions for user`) instead of staying at "align with `deliverable.md`". Reduces variance between experts and simplifies the moderator's work in STEP 7. Applies to STEP 3 (Round A) — and transitively B, D.

**P4 · Designer-Worker split of the panel (from AgentVerse)** — STEP 2 already does recruitment inline. AgentVerse explicitly separates "agent designer" from "agent worker": useful conceptually because it allows versioning the design logic separately. At minimum, move the 3 PASS (selection / persona / validation) to a **separate file** referenced (`actions/_panel_design.md` or similar) — still a single skill, not a sub-skill. Applies to STEP 2.

**P5 · Friction archetypes as deliberate "troublemakers" (from Peacemaker/Troublemaker)** — the sycophancy paper [arxiv 2509.23055](https://arxiv.org/abs/2509.23055) empirically supports the peacemaker+troublemaker mix. The line "Function: question, not propose" that you already have in `kind: friction` is exactly this. Cite it in a comment and consider adding **model/temperature diversity** between specialists and friction if the spawn primitive allows it — the papers suggest that model homogeneity amplifies groupthink.

## Council ↔ best-in-class comparison

| Feature | council | Du 2023 | MAD (Liang) | AgentVerse | MetaGPT | ChatDev |
|---|---|---|---|---|---|---|
| Distinct personas | Yes, inline | No | Yes (2) | Yes, dynamic | Yes, fixed | Yes, fixed |
| Panel size | 4-6 +0-2 | N=3-5 | 2 + judge | dynamic | 5 fixed | pairs per phase |
| Rounds | 4 (A/B/C/D) | fixed R | until break | 4 phases | pipeline | chat chain |
| Debate model | shuttle-mediated | broadcast | tit-for-tat | mixed | sequential | pairwise |
| Judge / moderator | impartial sub-agent | majority vote | LLM judge | evaluator | none | (human outside) |
| Round prompts | repeated verbatim | identical | identical | per phase | Action template | system+inception |
| Outputs | free markdown | answer string | answer string | mixed | structured docs | code+docs |
| Storage | filesystem | memory | memory | filesystem | filesystem | filesystem |

Council is the only one combining: dynamic panel + shuttle diplomacy + separate impartial moderator + filesystem-as-state + round barrier for user escalations. It is a reasoned hybrid, not a copy.

## Prioritized refactor options

**Option 1 — Extract reusable `EXPERT_SPAWN_HEADER`** (effort: low · impact: high). Define the identity/isolation/language/evidence header once; each round STEP references it. Removes ~120-150 lines from `deliberate.md`. Risk: none if the literal contract is preserved.

**Option 2 — Move STEP 2 (panel design) to `actions/_panel_design.md`** (effort: medium · impact: medium). STEP 2 is ~120 lines with its own logic (3 PASS + persona templates). Isolating it leaves `deliberate.md` focused on round orchestration. Be careful to keep it under a single skill entry (no sub-skills).

**Option 3 — YAML schemas for `proposals/critiques/final_positions/`** (effort: medium · impact: medium). Same as `problem.schema.yaml`/`deliverable.schema.yaml`. Reduces variance and makes the moderator more mechanical. Risk: excessive rigidity — the experts lose voice. Mitigate with optional sections.

**Option 4 — Virtual `RoundRunner`** (effort: high · impact: high). STEPs 3, 4, 6 are almost structurally identical (parallel spawn → wait → collect escalations → persist). Document it as a **reusable macro** ("execute a RoundPattern with these N experts, this header, these additional inputs"). STEP 5 remains as a documented exception (it's shuttle, not parallel). Risk: indirection that harms readability.

## Anti-patterns to avoid (with citation)

- **Conformity / sycophancy → premature consensus** ("LLMs' inherent sycophancy can collapse debates into premature consensus" — [Peacemaker or Troublemaker, 2025](https://arxiv.org/abs/2509.23055)). Mitigation in council: friction archetypes with mandatory adversarial voice + lead's rule "do not push consensus" + impartial moderator separate from the lead. Already covered; maintain.

- **Problem drift / overanalysis** ("overanalysis and hypercritical suggestions are highly related to problem drift… additional rounds can entrench initial errors" — [Stay Focused, 2025](https://arxiv.org/html/2502.19559v3)). Mitigation: scope-lock of `deliverable.md` (STEP 1.6) + moderator's "section mapping" audit (STEP 7.b). Already covered; **consider adding an explicit guard when moving from Round B to C**: if critiques exceed the deliverable's scope, cut and notify.

- **Persuasion over truth / role-identity bias** ("agents may prioritize winning arguments… persona-consistent actions even when they conflict with explicit goals" — [Talk Isn't Always Cheap, 2025](https://arxiv.org/abs/2509.05396) and [When Personas Override Payoffs, 2026](https://arxiv.org/abs/2601.10102)). Risk in council: a friction archetype that is too embodied may sabotage signal through adversariality. Mitigation already present: 0-2 friction limit over 4-6 specialists; the line "Function: question, not propose". **Honest caveat**: there is no benchmark in council that measures whether the current ratio is optimal — it is a design assumption.
