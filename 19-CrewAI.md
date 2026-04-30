# CrewAI — Role-Based Multi-Agent Orchestration

> Subject #19 of the Master Learning Plan.
> Owner: Govind Divekar. Prerequisites: file 04 (Python), file 18 (LangChain — helpful but not required), file 17 (LangGraph — useful comparison), file 13 (Vector DB) for memory/RAG, file 10 (Ollama) for local models. Estimated time: ~20 hours.
> Target outcome: Build, run, and operate role-based multi-agent "crews" with CrewAI — define agents (role/goal/backstory), tasks, processes, tools, memory; choose between Crews and Flows; wire to local or hosted LLMs; trace and evaluate; and articulate honestly when CrewAI fits vs LangGraph or n8n.

---

## 1. Overview & Learning Outcomes

CrewAI is an opinionated, **role-based** multi-agent framework (Python). Where LangGraph (file 17) is a low-level *graph runtime* you wire any way you like, and n8n (file 16) is a *visual workflow* tool, CrewAI bakes in an opinionated mental model:

- **Agents** have a `role`, `goal`, and `backstory`. They have tools, memory, and (optionally) delegation rights.
- **Tasks** are typed work items with a `description`, `expected_output`, an assigned agent, optional context, and optional `output_pydantic` schema.
- **Crew** is the team — a list of agents + a list of tasks + a `Process` (sequential or hierarchical with a manager LLM).
- **Flows** (newer) — a complementary low-level orchestration primitive (event-driven, decorator-based) when a Crew's task model is too coarse.
- **Tools** — anything callable; LangChain tools work, plus a built-in toolkit (web search, web scrape, file I/O, code interpreter).
- **Memory** — short-term (RAG over recent), long-term (persistent), entity, contextual.

CrewAI ships its own LLM abstraction (LiteLLM-based) so you can plug OpenAI, Anthropic, Bedrock, Gemini, Ollama, Groq, Azure, etc., behind one interface.

By the end of this subject you should be able to:

- Spin up a Crew of 2–4 specialized agents and ship a real deliverable (research brief, test plan, PR review).
- Pick **Sequential** vs **Hierarchical** process and justify it.
- Add tools (web search, code interpreter, custom REST), memory, and structured (Pydantic) outputs.
- Use **Flows** when a Crew is the wrong shape.
- Trace runs (CrewAI's tracing + LangSmith optional) and write task-level evals.
- Compare CrewAI honestly with LangGraph, AutoGen, and n8n, and pick the right tool per problem.

---

## 2. Detailed Syllabus

### Part A — Foundations (3 h)

1. What CrewAI is, philosophy (roles, opinionated) — 0.5h
2. Install, project layout (`crewai create`), env config — 0.5h
3. The four primitives: Agent, Task, Crew, Process — 1h
4. CrewAI's LLM abstraction (LiteLLM under the hood) — 0.5h
5. CrewAI Studio / CLI / Hub overview — 0.5h

### Part B — Agents, Tasks, Tools (4 h)

6. Defining agents — role / goal / backstory / `allow_delegation` — 1h
7. Defining tasks — `description`, `expected_output`, `context`, `agent`, `output_pydantic` — 1h
8. Tools: built-in toolkit (Serper, WebsiteSearchTool, FileReadTool, CodeInterpreter) — 1h
9. Custom tools with `@tool` and BaseTool subclass — 0.5h
10. LangChain interop — using LangChain tools/retrievers — 0.5h

### Part C — Process, Delegation, Hierarchy (3 h)

11. `Process.sequential` — 0.5h
12. `Process.hierarchical` and the manager LLM — 1h
13. Delegation between agents (`allow_delegation=True`) — 0.5h
14. Async tasks and parallel execution — 1h

### Part D — Memory, Knowledge, RAG (3 h)

15. Short-term, long-term, entity, contextual memory — 1h
16. Knowledge sources (PDF, CSV, JSON, web) — 1h
17. Persistence backends; integration with pgvector / Chroma / Qdrant — 1h

### Part E — Flows & Hybrid Orchestration (3 h)

18. When Crews aren't enough — Flows motivation — 0.5h
19. `@start`, `@listen`, `@router`, `@persist` decorators — 1h
20. Calling Crews from Flows; mixing imperative + agentic — 1h
21. State in Flows — 0.5h

### Part F — Production, Eval, and Trade-offs (4 h)

22. Tracing (CrewAI's built-in), optional LangSmith — 0.5h
23. CrewAI Plus / Enterprise vs OSS — 0.5h
24. Evaluation: golden tasks, criteria evals, regression on outputs — 1h
25. Cost & latency budgets — 0.5h
26. Self-hosting vs CrewAI Enterprise — 0.5h
27. Honest comparison: CrewAI vs LangGraph vs AutoGen vs n8n — 1h

---

## 3. Topics — Explanations with Examples

### 3.1 Setup

```bash
pip install -U crewai 'crewai[tools]'
# optional, for local models via Ollama:
pip install -U langchain-ollama
# optional, for tracing:
export LANGSMITH_TRACING=true
export LANGSMITH_API_KEY=lsv2_...
```

```bash
# Scaffold a project
crewai create crew qa-research-crew
cd qa-research-crew
# yields:
# src/qa_research_crew/{crew.py, main.py, tools/, config/agents.yaml, config/tasks.yaml}
```

The scaffold is **YAML-first** for agent and task definitions; Python wires it. You can also do everything in pure Python.

### 3.2 Configuring the LLM

CrewAI uses LiteLLM, so you typically just set env vars and a model string:

```bash
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=...
# Local via Ollama:
OPENAI_API_BASE=http://localhost:11434/v1
OPENAI_API_KEY=ollama  # placeholder; LiteLLM accepts any
```

```python
from crewai import LLM
llm_fast = LLM(model="openai/gpt-4o-mini", temperature=0.2)
llm_smart = LLM(model="anthropic/claude-3-5-sonnet-latest", temperature=0)
llm_local = LLM(model="ollama/llama3.1:8b", base_url="http://localhost:11434")
```

### 3.3 Agents

```python
from crewai import Agent

researcher = Agent(
    role="Senior Research Analyst",
    goal="Find primary sources and synthesize findings on {topic}.",
    backstory=("You are a former librarian-turned-analyst with a habit of "
               "checking three independent sources before quoting a fact."),
    llm=llm_smart,
    allow_delegation=False,
    verbose=True,
    max_iter=10,             # cap reasoning loops
    max_execution_time=120,  # seconds
    memory=True,
)
```

Choosing `role`/`goal`/`backstory`:

- **Role** is short and human-readable ("QA Test Strategist", not "Agent_2").
- **Goal** is a one-sentence outcome the agent owns.
- **Backstory** is *task-relevant flavor* that nudges priors. Don't write a novel — 2–4 sentences.

`allow_delegation=True` lets an agent assign work to peers (in `hierarchical` process or via a manager).

### 3.4 Tasks

```python
from crewai import Task
from pydantic import BaseModel

class Brief(BaseModel):
    summary: str
    citations: list[str]
    risks: list[str]

research_task = Task(
    description=("Produce a 1-page brief on '{topic}'. Include 5 citations and "
                 "3 risks."),
    expected_output="A markdown brief matching the schema.",
    agent=researcher,
    output_pydantic=Brief,
    context=[],         # other tasks whose outputs feed this one
    async_execution=False,
)
```

**Critical fields:**

- `description` — what to do (templated with `{vars}`).
- `expected_output` — what "done" looks like (the model uses this to know when to stop).
- `agent` — who owns it.
- `context` — list of upstream tasks whose outputs are appended into this task's input.
- `output_pydantic` / `output_json` — typed output (preferred over free-form when downstream code consumes it).
- `async_execution=True` — let the crew run this in parallel with peers.

### 3.5 Crew + Process

```python
from crewai import Crew, Process

crew = Crew(
    agents=[researcher, writer, reviewer],
    tasks=[research_task, draft_task, review_task],
    process=Process.sequential,        # or Process.hierarchical
    memory=True,
    verbose=True,
)

result = crew.kickoff(inputs={"topic": "metamorphic testing for image classifiers"})
print(result.raw)            # final output
print(result.tasks_output)   # per-task outputs
```

`Process.sequential` runs tasks in declared order. Each task's output is automatically available as `context` for downstream tasks that list it.

`Process.hierarchical` introduces a **manager LLM** that reads the task list and routes to agents. Provide `manager_llm=` (or `manager_agent=`) on the Crew.

```python
crew = Crew(
    agents=[researcher, writer, reviewer],
    tasks=[research_task, draft_task, review_task],
    process=Process.hierarchical,
    manager_llm=llm_smart,
)
```

### 3.6 Tools

```python
from crewai_tools import SerperDevTool, WebsiteSearchTool, FileReadTool

search = SerperDevTool()                   # needs SERPER_API_KEY
site_search = WebsiteSearchTool(website="https://docs.qa.example.com/")
read_file = FileReadTool(file_path="./runbook.md")

researcher.tools = [search, site_search, read_file]
```

Custom tool — function form:

```python
from crewai.tools import tool

@tool("Get CI build status")
def ci_status(build_id: str) -> str:
    """Look up the status of a GitHub Actions build by id."""
    # ... call API ...
    return "succeeded"
```

Custom tool — class form (when you need state, schemas, or async):

```python
from pydantic import BaseModel
from crewai.tools import BaseTool

class LinearArgs(BaseModel):
    query: str

class LinearSearch(BaseTool):
    name: str = "Search Linear issues"
    description: str = "Search the team's Linear workspace by free-text query."
    args_schema: type[BaseModel] = LinearArgs

    def _run(self, query: str) -> str:
        # ... call Linear ...
        return "[ENG-12] Login flaky on iOS Safari"
```

LangChain tools (file 18) work too — pass them in `tools=[...]` and CrewAI will adapt them.

### 3.7 Memory

CrewAI's memory has four layers, all opt-in via `memory=True`:

- **Short-term** — recent execution context (RAG over last N turns).
- **Long-term** — persistent across runs (sqlite by default; pgvector/Chroma/Qdrant as advanced).
- **Entity** — facts about people, products, projects mentioned during runs.
- **Contextual** — task-level scratch.

You configure the storage backend via env (`CREWAI_STORAGE_DIR`) or by passing `embedder=` on the crew (e.g., `OpenAIEmbeddings`, `OllamaEmbeddings`, `HuggingFaceEmbeddings`).

```python
crew = Crew(
    agents=[...], tasks=[...],
    memory=True,
    embedder={"provider": "openai", "config": {"model": "text-embedding-3-small"}},
)
```

### 3.8 Knowledge sources

For grounding agents with documents at construction time (separate from runtime tools):

```python
from crewai.knowledge.source.pdf_knowledge_source import PDFKnowledgeSource
from crewai.knowledge.source.csv_knowledge_source import CSVKnowledgeSource

knowledge = [
    PDFKnowledgeSource(file_paths=["docs/QA_Strategy.pdf"]),
    CSVKnowledgeSource(file_paths=["data/test_inventory.csv"]),
]

crew = Crew(agents=[...], tasks=[...], knowledge_sources=knowledge)
```

This lets agents retrieve from those sources without you writing a retriever-tool.

### 3.9 Async tasks and parallelism

Mark independent tasks `async_execution=True`. The crew schedules them to run concurrently and *gathers* their outputs before any task that lists them in `context`:

```python
gather_market = Task(..., async_execution=True)
gather_competitors = Task(..., async_execution=True)
synthesize = Task(..., context=[gather_market, gather_competitors])
```

### 3.10 The hierarchical process — when to reach for it

Sequential is the right default; reach for hierarchical when:

- Tasks are **adaptive** — the right next step depends on intermediate findings.
- You want a manager LLM to delegate based on agent roles, not pre-wired ordering.
- You're willing to pay for an extra LLM step (the manager).

Don't reach for it just because "it sounds smarter" — sequential is faster, cheaper, and easier to test.

### 3.11 Outputs and post-processing

Every kickoff returns a `CrewOutput` with:

- `.raw` — final task's text output.
- `.pydantic` — parsed model if the final task had `output_pydantic`.
- `.tasks_output` — list of `TaskOutput`, each with `.raw`, `.pydantic`, `.summary`.
- `.token_usage` — `{prompt_tokens, completion_tokens, total_tokens}` per agent/task.

For determinism in CI, **always** prefer `output_pydantic` for the final task and assert against the typed object.

### 3.12 Flows — when a Crew is the wrong shape

Crews are great when work decomposes into a *handful* of role-owned tasks. They get awkward when you need:

- Event-driven branching ("only run this if the previous step's score < 0.7").
- Long-lived workflows with many checkpoints.
- Mixing imperative code (DB writes, HTTP calls) with agent steps cleanly.

CrewAI's **Flows** are a complementary primitive:

```python
from crewai.flow.flow import Flow, start, listen, router

class TriageFlow(Flow):
    @start()
    def receive(self):
        return {"ticket": self.state["ticket"]}

    @listen(receive)
    def classify(self, payload):
        # call a Crew, an LLM, or any function
        return classify_with_crew(payload["ticket"])

    @router(classify)
    def route(self, label):
        return "bug_path" if label == "bug" else "feature_path"

    @listen("bug_path")
    def open_linear(self, _):
        return create_linear_issue(self.state["ticket"])

flow = TriageFlow()
flow.kickoff(inputs={"ticket": "..."})
```

Flows + Crews are how production CrewAI apps usually look: Flows handle orchestration; Crews handle the agentic chunks.

### 3.13 Local models with Ollama

```python
from crewai import LLM

llm = LLM(
    model="ollama/llama3.1:8b",
    base_url="http://localhost:11434",
    temperature=0.2,
)
```

Tool calling with smaller local models is hit-or-miss; for tool-heavy crews use a model with strong function calling (Claude, GPT-4o, or Llama 3.1 70B+).

### 3.14 Tracing and observability

CrewAI emits structured events. You can:

- Pass `verbose=True` on agents/crew for stdout traces.
- Use **LangSmith** by setting `LANGSMITH_TRACING=true` — CrewAI integrates and shows each agent/task as a span.
- Use **CrewAI's own tracing** when running on CrewAI Enterprise.
- Roll your own with callbacks on the LLM (LiteLLM exposes them).

Always log `result.token_usage` per run; agent loops can be silently expensive.

### 3.15 Testing crews

Treat each task as a unit. Tactics:

- **Stub the LLM** with a fake `LLM` that returns canned text for matching prompts (subclass `LLM` and override `call`).
- **Stub tools** with deterministic implementations.
- **Assert on `output_pydantic`** rather than free-form text.
- **Snapshot** task outputs in `tests/snapshots/`; review diffs on PR.
- **Eval suite** — a directory of `cases/*.json` with `inputs` + `expected`; loop with pytest; grade with rubric LLM or heuristic.

```python
def test_research_crew_returns_brief():
    crew = build_crew(llm=fake_llm)
    out = crew.kickoff(inputs={"topic": "x"})
    assert isinstance(out.pydantic, Brief)
    assert len(out.pydantic.citations) >= 5
```

### 3.16 Cost and latency

Each task is at minimum one LLM call; agents with tools loop. Defaults that bite:

- `max_iter` — agent's reasoning-loop cap (default ~25). Set lower for cheap crews.
- `max_execution_time` — wall-clock cap per agent.
- `Process.hierarchical` adds a **manager** LLM call per routing decision.
- Memory writes/reads can add embedding calls.

A defensible starter: cap max_iter at 5–8 for most agents, raise only when traces show the agent legitimately needs more.

### 3.17 CrewAI vs LangGraph vs AutoGen vs n8n — honest take

| Need                                                 | Pick                |
|------------------------------------------------------|---------------------|
| You think in *roles + tasks* and want fast scaffolding | **CrewAI**         |
| You need explicit graph state, cycles, HITL          | **LangGraph (17)**  |
| Conversation-style multi-agent, code execution focus | **AutoGen**         |
| Visual canvas non-engineers will edit                | **n8n (16)**        |
| Linear `prompt → model → parser` only                | **LangChain LCEL (18)** |

CrewAI is the fastest path to a usable multi-agent prototype. Once you need fine-grained control over what runs when (parallel branches, retries with feedback, human approvals on specific transitions), LangGraph is the more honest fit. The two coexist well: a Flow that calls a Crew at one node, or a LangGraph that calls a Crew as a tool.

### 3.18 Project layout that scales

Recommended layout for a non-trivial CrewAI app:

```
src/qa_crew/
├── crew.py                  # builds Crews
├── flows.py                 # Flows that orchestrate Crews
├── llms.py                  # LLM presets (cheap/fast/smart)
├── tools/
│   ├── linear.py
│   ├── ci.py
│   └── kb_retriever.py
├── schemas/
│   └── triage.py            # Pydantic models for outputs
├── config/
│   ├── agents.yaml
│   └── tasks.yaml
├── knowledge/               # PDFs, CSVs ingested at construction
└── main.py
tests/
├── unit/
└── eval/
    ├── cases/
    └── runner.py
```

YAML-driven agents/tasks are great for rapid editing and for non-engineers to tweak prompts without touching Python.

### 3.19 Common anti-patterns

- **One mega-agent** doing everything ("AI Engineer" with 12 tools and a 600-word backstory). Split into 2–3 narrow agents.
- **Free-form final outputs** consumed by code. Always use `output_pydantic`.
- **No max_iter / max_execution_time.** A single bad run can rack up dollars.
- **Memory enabled, but no embedder configured** for production — defaults may not match your scale.
- **Hierarchical process by default.** Cheaper, faster, and more deterministic to start sequential.
- **Skipping evals.** Without a golden set, every prompt change is a coin flip.

---

## 4. Exercises (Hands-On)

### Beginner

- **A1.** `crewai create crew hello-crew`. Wire two agents (researcher + writer), two sequential tasks (research, write). Run with a real model and read the trace.
  - *Acceptance:* `result.raw` produces a coherent 200-word brief.
- **A2.** Add `output_pydantic=Brief` to the writer task; assert in a pytest.
- **A3.** Swap the LLM to Ollama (`llama3.1:8b`); run offline. Compare quality and latency vs OpenAI.

### Intermediate

- **B1.** Add `SerperDevTool` to the researcher. Cap `max_iter=6`. Verify with traces that tool calls happen and stop.
- **B2.** Build a third "fact-checker" agent. Use `Process.hierarchical` with a `manager_llm`. Compare runtime/cost vs sequential.
- **B3.** Add a custom `LinearSearch` tool (class-based, with `args_schema`). Hand it to the researcher.
- **B4.** Add `memory=True` with a pgvector embedder; run the crew twice on related topics and verify entity memory carries over.

### Advanced

- **C1.** Wrap the Crew inside a Flow: `@start` reads a Slack webhook payload, `@listen` calls the Crew, `@router` branches to "create_linear_issue" or "reply_in_thread".
- **C2.** Async tasks: split research into market + competitors + tech, all `async_execution=True`. Synthesize in a final task with `context=[...]`.
- **C3.** Knowledge sources: load 5 PDFs of testing standards (e.g., ISTQB), have the crew produce a tailored test plan that cites the documents by section.
- **C4.** Eval harness: 20 golden inputs in `tests/eval/cases/`, a rubric LLM evaluator (correctness, citations valid, schema valid), and a pytest run that fails if average score drops below 0.85. Wire to CI.

### Capstone — "Test Strategy Crew"

Production-shaped CrewAI app:

1. Three agents: **Researcher** (web + KB tools), **Strategist** (writes the plan), **Reviewer** (critiques and scores).
2. Sequential tasks ending in a Pydantic `TestPlan` model with sections, risks, exit criteria, automation candidates.
3. Knowledge sources: your team's runbooks + 1–2 ISTQB chapters.
4. Memory enabled with pgvector; embeddings via OpenAI or Ollama.
5. A Flow front-end that takes a Linear ticket and emits the test plan back into a comment.
6. Eval suite of 15 golden tickets; nightly CI run; alert if score drops.
7. README comparing CrewAI vs LangGraph for this exact use case (where each would be better).

---

## 5. Additional Reading & Resources

### Official docs
- CrewAI documentation — <https://docs.crewai.com/>
- CrewAI GitHub — <https://github.com/crewAIInc/crewAI>
- CrewAI Tools — <https://github.com/crewAIInc/crewAI-tools>
- Examples repo — <https://github.com/crewAIInc/crewAI-examples>
- CrewAI Studio — <https://github.com/strnad/CrewAI-Studio>

### Tutorials
- DeepLearning.AI: *Multi AI Agent Systems with crewAI* (free) — by João Moura, the creator.
- CrewAI Hub (community templates) — linked from docs homepage.
- *Practical Multi-Agent Systems* posts on the CrewAI blog.

### YouTube
- João Moura's channel and CrewAI's official channel.
- *Sam Witteveen* — recurring CrewAI walkthroughs.
- *Matt Williams* — local-LLM oriented Crew demos.

### Adjacent / when to compare
- LangGraph (file 17) — <https://langchain-ai.github.io/langgraph/>
- AutoGen (Microsoft) — <https://microsoft.github.io/autogen/>
- LlamaIndex agents — <https://docs.llamaindex.ai/en/stable/module_guides/deploying/agents/>
- n8n AI Agent (file 16) — <https://docs.n8n.io/advanced-ai/>

### Books / long-form
- *Generative AI with LangChain*, 2nd ed., Auffarth — has comparative agent chapters.
- *AI Agents in Action* (Manning, MEAP).

---

## 6. Index

- Agent (role/goal/backstory) — §3.3
- async tasks — §3.9
- backstory authoring — §3.3
- BaseTool (custom class) — §3.6
- Crew — §3.5
- CrewOutput / TaskOutput — §3.11
- delegation — §3.3, §3.10
- embedder (memory backend) — §3.7
- Flows (`@start`, `@listen`, `@router`) — §3.12
- hierarchical process — §3.5, §3.10
- knowledge sources — §3.8
- LangChain tool interop — §3.6
- LiteLLM / `LLM` class — §3.2
- local model (Ollama) — §3.13
- manager LLM — §3.5
- max_iter / max_execution_time — §3.16
- memory (short / long / entity / contextual) — §3.7
- output_pydantic / output_json — §3.4, §3.11
- Process.sequential vs hierarchical — §3.5
- project scaffold (`crewai create`) — §3.1
- SerperDevTool — §3.6
- Task — §3.4
- testing crews — §3.15
- tracing (LangSmith) — §3.14
- vs LangGraph / AutoGen / n8n — §3.17

---

## 7. References

1. CrewAI Documentation — <https://docs.crewai.com/>
2. CrewAI GitHub — <https://github.com/crewAIInc/crewAI>
3. CrewAI Tools — <https://github.com/crewAIInc/crewAI-tools>
4. CrewAI Examples — <https://github.com/crewAIInc/crewAI-examples>
5. DeepLearning.AI — *Multi AI Agent Systems with crewAI* — <https://www.deeplearning.ai/short-courses/>
6. LiteLLM — <https://docs.litellm.ai/>
7. CrewAI Flows — <https://docs.crewai.com/concepts/flows>
8. CrewAI Memory — <https://docs.crewai.com/concepts/memory>
9. CrewAI Knowledge — <https://docs.crewai.com/concepts/knowledge>
10. LangGraph — <https://langchain-ai.github.io/langgraph/>
11. AutoGen — <https://microsoft.github.io/autogen/>
12. n8n AI — <https://docs.n8n.io/advanced-ai/>
13. *Generative AI with LangChain*, 2nd ed., Auffarth (Packt).
14. CrewAI YouTube — <https://www.youtube.com/@crewAIInc>

---

*End of Subject 19 — CrewAI.*
