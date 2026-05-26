# Report 2/4 — Multi-agent orchestration frameworks

> Background agent investigating how CrewAI, AutoGen, LangGraph, OpenAI Agents SDK and Magentic-One structure multi-agent procedures — to extract patterns applicable to `actions/deliberate.md` (without migrating to another framework).

## Relevant findings by framework

**1. CrewAI** ([docs](https://docs.crewai.com/en/concepts/agents), [YAML config](https://deepwiki.com/crewAIInc/crewAI/8.2-yaml-configuration))
- **Clear persona vs task separation**: `agents.yaml` defines `role`/`goal`/`backstory` (the persona, stable); `tasks.yaml` defines `description`/`expected_output` (the changing part). The framework forces you not to mix them.
- **`{var}` interpolation**: a single task template serves N executions by injecting `inputs` into `crew.kickoff()`. There is no inheritance between tasks, but the declarative contract avoids repeating the identity block.
- **Auto-propagated sequential context**: in sequential processes, the output of the previous task enters as implicit context — you don't repeat it in the prompt.

**2. AutoGen / Magentic-One** ([multi-agent debate](https://microsoft.github.io/autogen/stable//user-guide/core-user-guide/design-patterns/multi-agent-debate.html), [Magentic-One paper](https://arxiv.org/html/2411.04468v1))
- **Single SystemMessage per agent**, persistent across rounds. The round-to-round difference is put in the **content** of the turn message, not by re-emitting identity. Verbatim quote: *"A single static SystemMessage is defined once during solver initialization … persists throughout all debate rounds rather than being regenerated per round."*
- **Magentic-One Orchestrator**: factors prompts into named constants (`ORCHESTRATOR_TASK_LEDGER_FACTS_PROMPT`, `ORCHESTRATOR_TASK_LEDGER_PLAN_PROMPT`, `*_UPDATE_PROMPT`). Each is a short reusable block. The orchestration is a **loop over a shared ledger**, not a linear sequence of prompts.

**3. LangGraph supervisor / hierarchical** ([supervisor-py](https://github.com/langchain-ai/langgraph-supervisor-py), [LangChain ChatPromptTemplate](https://python.langchain.com/api_reference/core/prompts/langchain_core.prompts.chat.ChatPromptTemplate.html))
- Each **node** of the graph encapsulates an agent + its prompt; the state flows through the edge, not through the prompt. This shifts prompt repetition toward state mutations.
- LangChain `ChatPromptTemplate.partial()` and `MessagesPlaceholder` enable **pre-baking**: fill in the stable variables once, leave gaps for those that change per turn. Identity goes in partial; the per-turn delta goes in placeholder.

**4. OpenAI Agents SDK / Swarm** ([Agents SDK](https://openai.github.io/openai-agents-python/agents/), [Swarm repo](https://github.com/openai/swarm))
- **Dynamic instructions**: `instructions` can be a callable `(context, agent) -> str`. Allows "rendering" the prompt according to context without storing N copies.
- **`RECOMMENDED_PROMPT_PREFIX` + `prompt_with_handoff_instructions()`**: canonical helper for **shared prefix** + specific body. Equivalent to `f"{PREFIX}\n\n{prompt}"`. It is exactly the "common header / specific body" pattern.

**5. LangChain prompt composition** — `ChatPromptTemplate.from_messages([...])` allows composing messages from fragments; `partial()` leaves a template partially bound. It is the primitive that is missing when everything is plain markdown.

## Patterns applicable to `deliberate.md`

1. **Common-header constant (à la `RECOMMENDED_PROMPT_PREFIX`)** — Rounds A/B/C/D repeat an identical block: "You are `<name>`, specialist in `<role_category>`. You are participating as an independent expert. DO NOT communicate with other experts — the lead is the only channel. Language: Spanish…". Extract it into a quoted section `### EXPERT_HEADER_TEMPLATE` (~10 lines) that each STEP references with *"[insert EXPERT_HEADER_TEMPLATE with `<name>` and `<role_category>`]"*. Applies to STEPs 3, 4, 5.c, 6. **Trims ~30-40 lines**.

2. **Round-specific delta-only blocks** (AutoGen pattern) — After extracting the header, each round STEP is left with ONLY its delta: new paths to read + output path + verb of the round (propose / critique / respond to conflict / consolidate). This turns 4 prompts of ~20 lines into 4 deltas of ~6 lines + a quoted header.

3. **Anti-leak / evidence discipline as a quoted block** — These rules currently live inside the Round A prompt (STEP 3, ~3 inline lines) but conceptually apply to ALL rounds. Extract `### EVIDENCE_DISCIPLINE_BLOCK` and reference it from the header. Applies to STEPs 3-6. Double benefit: factoring + correct propagation to later rounds.

4. **Escalation-batching as a named subroutine** (Magentic-One "named prompt" pattern) — The block "Collect escalations (round barrier) … dedup + multiple attribution" is quoted verbatim in STEP 3 and by reference in STEPs 4-6. Convert it into a sub-procedure `### CLOSE_ROUND(round_letter)` with the steps numbered, and let each STEP end with *"Execute CLOSE_ROUND('a')"*. This puts the procedure at the correct level and eliminates the "see Round A for detail".

5. **Schema-driven outcome (already implicit via `deliverable.md`)** — STEP 7.b already delegates the form to the deliverable. This is exactly the CrewAI `expected_output` pattern. Keep it and document it as an architectural decision.

## Prioritized refactor options (STEPs 3-6)

| # | Option | Effort | Impact |
|---|---|---|---|
| **A** | Extract `EXPERT_HEADER_TEMPLATE` + `EVIDENCE_DISCIPLINE_BLOCK` + `CLOSE_ROUND(round)` into a "Reusable templates" section at the start of the file, and leave each round STEP as a **pure delta**. | Low (~1h) | High: -100 to -150 lines, a single source of truth per block. |
| **B** | Extract the templates into `actions/_partials/` (header.md, evidence.md, close.md) and reference them with `Read` from the director. | Medium | High + reusable from `refine.md` (which also runs rounds). Risk: more files to read at runtime. |
| **C** | Convert the 4 rounds into a **declarative table** (round → files to read → file to write → verb) and a single procedure block that iterates. Magentic-One ledger pattern. | High | Very high in compactness but **high risk** of losing nuance (Round C is shuttle, not parallel — the table would have one exceptional row). |
| **D** | Only extract `EXPERT_HEADER_TEMPLATE`. Leave the rest. | Very low | Medium: -30 lines, demonstrates the pattern without risk. |

**Recommendation**: start with **A** (maximum ROI, low risk). If it works, `refine.md` benefits at zero cost by referencing the same templates. **C** is attractive but the heterogeneity of Round C (mediated, sequential, by conflict) means the table would have an exception-row that probably costs more readability than it saves.

## Documented anti-patterns

1. **Re-emitting identity per turn** (AutoGen explicitly avoids it): if the persona is stable, repeating it in each prompt forces the model to re-process it and dilutes attention toward the turn delta. *"The agent's core identity remains static, while the 'delta' comes from varying prompt construction"*.

2. **Mixing persona and task in a single block** (CrewAI separates them by design): `role`/`backstory` live in `agents.yaml`, `description`/`expected_output` in `tasks.yaml`. When both live together in each prompt, changing the deliverable forces editing N prompts. In `deliberate.md`, the paths the expert must read are part of the **task**, not the **persona** — they are well isolated; the identity header is what gets repeated.

3. **Saying "applies as in Round A but with this"** without extracting the subroutine (Magentic-One names each block: `*_FACTS_PROMPT`, `*_PLAN_UPDATE_PROMPT`). STEP 4 says *"with the same identity header + paths as in Round A"* — that is exactly the signal that a named subroutine is missing. The textual reference to "Round A" forces the reader (and the model) to jump back to verify what you inherited.

## Sources

- [CrewAI Agents](https://docs.crewai.com/en/concepts/agents) · [YAML Configuration](https://deepwiki.com/crewAIInc/crewAI/8.2-yaml-configuration)
- [AutoGen Multi-Agent Debate](https://microsoft.github.io/autogen/stable//user-guide/core-user-guide/design-patterns/multi-agent-debate.html) · [AutoGen paper](https://arxiv.org/pdf/2308.08155)
- [Magentic-One paper](https://arxiv.org/html/2411.04468v1) · [Magentic-One repo](https://github.com/microsoft/autogen/tree/staging/python/packages/autogen-magentic-one)
- [LangGraph supervisor](https://github.com/langchain-ai/langgraph-supervisor-py) · [Hierarchical teams](https://langchain-ai.github.io/langgraph/tutorials/multi_agent/hierarchical_agent_teams/)
- [OpenAI Agents SDK — Agents](https://openai.github.io/openai-agents-python/agents/) · [Handoffs](https://openai.github.io/openai-agents-python/handoffs/) · [`RECOMMENDED_PROMPT_PREFIX`](https://github.com/openai/openai-agents-python) · [OpenAI Swarm](https://github.com/openai/swarm)
- [LangChain ChatPromptTemplate](https://python.langchain.com/api_reference/core/prompts/langchain_core.prompts.chat.ChatPromptTemplate.html)
