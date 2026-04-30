# LangGraph — Stateful, Cyclical, Multi-Agent LLM Orchestration

> Subject #17 of the Master Learning Plan.
> Owner: Govind Divekar. Prerequisites: file 04 (Python), file 18 (LangChain — strongly recommended; LangGraph composes with LangChain runnables), file 13 (Vector DB) for RAG agents. Estimated time: ~25 hours.
> Target outcome: Build production-quality agent systems with LangGraph — explicit state, conditional/cyclical control flow, checkpointing, human-in-the-loop, multi-agent supervision, and streaming — with LangSmith tracing and a clear path to deployment.

---

## 1. Overview & Learning Outcomes

LangGraph is **Pregel-style** (think: graph of nodes that read/update shared state) orchestration for LLM apps, written by the LangChain team. Where LangChain's LCEL (`|`) is great for *linear* pipelines, LangGraph is the right tool when you need:

- **State** — shared across nodes, reducer-based, persisted.
- **Cycles** — agent loops, retry-with-feedback, planning + execution.
- **Branching** — conditional edges driven by state or LLM decisions.
- **Human-in-the-loop** — `interrupt` and resume.
- **Multi-agent** — supervisor, swarm, hierarchical, plan-and-solve.
- **Reliability** — checkpoints + resumable execution + time travel.

LangGraph is the **modern replacement for LangChain's legacy `AgentExecutor`**. It's a separate package (`langgraph`) and works with any LangChain `Runnable` (or any callable, really — you don't *need* LangChain to use LangGraph).

By the end of this subject you should be able to:

- Define a `StateGraph` with a typed state schema and reducer.
- Wire conditional edges, cycles, and parallel fan-out via `Send`.
- Implement a ReAct-style agent in ~50 lines.
- Add checkpointing (in-memory → SQLite → Postgres) for crash-safe runs.
- Implement human-in-the-loop with `interrupt()` and resume via `Command`.
- Compose multi-agent systems (supervisor, swarm).
- Stream tokens, intermediate steps, and state updates.
- Trace with LangSmith and deploy via LangGraph Cloud / LangGraph Platform / plain FastAPI.

---

## 2. Detailed Syllabus

### Part A — Foundations (4 h)

1. Why LangGraph (vs LCEL chains, vs AgentExecutor) — 0.5h
2. Install, set up, LangSmith env — 0.5h
3. The graph model: nodes, edges, state, START / END — 1h
4. State schemas: `TypedDict`, Pydantic, dataclasses — 0.5h
5. Reducers and the `Annotated[..., reducer]` pattern — 1h
6. `MessagesState` and `add_messages` — 0.5h

### Part B — Control Flow (4 h)

7. Linear graphs, `add_node` / `add_edge` — 0.5h
8. Conditional edges (`add_conditional_edges`) — 1h
9. Cycles and stop conditions — 0.5h
10. Parallel fan-out with `Send` — 1h
11. Subgraphs — 0.5h
12. Streaming modes: `values`, `updates`, `messages`, `debug` — 0.5h

### Part C — Agents & Tools (5 h)

13. The pre-built `create_react_agent` — 0.5h
14. Building a ReAct agent from scratch — 1.5h
15. Tool nodes, `ToolNode`, retries, error handling — 1h
16. Tool calling with structured output — 0.5h
17. Plan-and-execute pattern — 1h
18. Reflection / self-critique loop — 0.5h

### Part D — Persistence & HITL (4 h)

19. Checkpointers: `MemorySaver`, `SqliteSaver`, `PostgresSaver` — 1h
20. Threads, `thread_id`, conversation resumption — 0.5h
21. Time travel: replay from a past checkpoint — 0.5h
22. `interrupt()` for human review/approval — 1h
23. `Command(resume=...)` and `Command(goto=...)` — 0.5h
24. Long-term memory stores — 0.5h

### Part E — Multi-Agent (4 h)

25. Supervisor pattern — 1h
26. Swarm pattern — 0.5h
27. Hierarchical teams — 1h
28. Handoffs via `Command` — 0.5h
29. Shared vs per-agent state — 0.5h
30. Coordination anti-patterns — 0.5h

### Part F — Production (4 h)

31. LangSmith tracing & evaluation for graphs — 1h
32. Streaming to UIs (SSE / WebSocket) — 0.5h
33. LangGraph Platform / Cloud deployment — 1h
34. Self-hosting with FastAPI + a checkpointer — 1h
35. Testing graphs (unit + integration) — 0.5h

---

## 3. Topics — Explanations with Examples

### 3.1 Setup

```bash
pip install -U langgraph langgraph-checkpoint-sqlite langgraph-checkpoint-postgres \
              langchain langchain-openai langchain-anthropic langchain-ollama \
              langsmith
```

```bash
LANGSMITH_TRACING=true
LANGSMITH_API_KEY=lsv2_...
LANGSMITH_PROJECT=langgraph-prep
OPENAI_API_KEY=sk-...
```

### 3.2 The graph model

A LangGraph app is a graph:

- **Nodes** — Python functions (sync or async) that take *state* and return a *partial state update*.
- **Edges** — directed transitions. **Conditional edges** are functions that pick the next node.
- **State** — a typed dict-like object shared by all nodes.
- **START / END** — special sentinels.
- **Reducers** — when two nodes update the same key, the reducer says how to merge.

You **compile** a graph to get a runnable.

### 3.3 Hello, graph

```python
from typing import TypedDict
from langgraph.graph import StateGraph, START, END

class S(TypedDict):
    text: str
    upper: str

def to_upper(state: S) -> S:
    return {"upper": state["text"].upper()}

g = StateGraph(S)
g.add_node("upper", to_upper)
g.add_edge(START, "upper")
g.add_edge("upper", END)
app = g.compile()

print(app.invoke({"text": "hello"}))   # {'text': 'hello', 'upper': 'HELLO'}
```

**Mental model:** each node returns a *patch*. The graph framework merges patches into the running state using your reducers.

### 3.4 State schemas and reducers

The default reducer is "replace." For lists you usually want **append**.

```python
from typing import Annotated, TypedDict
from operator import add
from langchain_core.messages import AnyMessage
from langgraph.graph.message import add_messages

class AgentState(TypedDict):
    messages: Annotated[list[AnyMessage], add_messages]   # append-with-id-dedupe
    scratch:  Annotated[list[str], add]                   # plain append
    counter:  int                                         # replace
```

`add_messages` is the standard reducer for chat history — it appends, but if a message has an existing id it *replaces* it (essential for tool-call streaming).

`MessagesState` is a pre-baked state class with just `messages`. Use it for simple chatbots.

### 3.5 Conditional edges

```python
def route(state: AgentState) -> str:
    last = state["messages"][-1]
    if last.tool_calls:
        return "tools"
    return END

g.add_conditional_edges("agent", route, {"tools": "tools", END: END})
```

The router returns a *string* (or a list of strings), and the mapping says which node it maps to. You can omit the mapping if the returned string equals the target node's name.

### 3.6 Cycles

A cycle is just an edge that goes "backwards" in the topological sense. LangGraph allows them — that's the whole point.

```text
agent ──tool_calls──▶ tools ──▶ agent ──no tool_calls──▶ END
```

Add a **stop condition** (e.g., max iterations, time limit) so a misbehaving model can't loop forever:

```python
g = StateGraph(AgentState)
# ...
app = g.compile()
result = app.invoke({"messages": [...]}, config={"recursion_limit": 25})
```

### 3.7 Parallel fan-out with `Send`

`Send` lets one node dispatch *multiple* invocations of another node, each with its own state:

```python
from langgraph.constants import Send

def split_into_subqueries(state):
    return [Send("retrieve", {"query": q}) for q in state["queries"]]

g.add_conditional_edges("planner", split_into_subqueries)
```

Combined with append-style reducers, this gives map-reduce.

### 3.8 Streaming

```python
for event in app.stream({"messages": [HumanMessage("Hi")]}, stream_mode="updates"):
    print(event)         # {node_name: state_update}

for chunk in app.stream({...}, stream_mode="messages"):
    print(chunk)         # token-by-token messages

for state in app.stream({...}, stream_mode="values"):
    print(state)         # full state after each step
```

`stream_mode` accepts a list to multiplex (`["updates", "messages"]`).

### 3.9 The pre-built ReAct agent

The fastest path to a working agent is `create_react_agent`:

```python
from langchain_openai import ChatOpenAI
from langchain_core.tools import tool
from langgraph.prebuilt import create_react_agent

@tool
def search_orders(order_id: str) -> dict:
    """Look up an order by id."""
    return {"id": order_id, "status": "shipped"}

llm = ChatOpenAI(model="gpt-4o-mini")
agent = create_react_agent(llm, tools=[search_orders])

resp = agent.invoke({"messages": [("user", "What's the status of order 42?")]})
print(resp["messages"][-1].content)
```

Internally this is a graph: `agent → tools → agent → END`.

### 3.10 ReAct from scratch (so you understand it)

```python
from langgraph.graph import StateGraph, START, END, MessagesState
from langgraph.prebuilt import ToolNode
from langchain_core.messages import AIMessage

llm_with_tools = llm.bind_tools([search_orders])
tool_node = ToolNode([search_orders])

def call_model(state: MessagesState):
    return {"messages": [llm_with_tools.invoke(state["messages"])]}

def should_continue(state: MessagesState):
    last: AIMessage = state["messages"][-1]
    return "tools" if last.tool_calls else END

g = StateGraph(MessagesState)
g.add_node("agent", call_model)
g.add_node("tools", tool_node)
g.add_edge(START, "agent")
g.add_conditional_edges("agent", should_continue)
g.add_edge("tools", "agent")
graph = g.compile()
```

This 15-line graph is roughly what `AgentExecutor` did in 200+ lines — and now *you* control every transition.

### 3.11 ToolNode patterns

`ToolNode` runs the tool calls in the last AI message and appends `ToolMessage`s to state. You can:

- Customize error handling with `handle_tool_errors=`.
- Wrap with `RunnableWithFallbacks` for retries.
- Skip with a custom node when you want to validate args first (e.g., schema-check before hitting an external API).

### 3.12 Plan-and-execute

Pattern: a *planner* LLM decomposes the task; an *executor* loops on each step; a *replan* node revises if execution stalls.

```text
planner ──▶ executor ──done?──▶ END
                  │
                  └──stuck──▶ replan ──▶ executor
```

Implement `executor` as a sub-agent (subgraph) for clean separation.

### 3.13 Reflection / self-critique

Pattern: after an answer, a *critic* node grades or proposes edits; the *generator* revises until accepted or budget exhausted.

```text
generate ──▶ critique ──ok──▶ END
                 │
                 └──revise──▶ generate
```

LangGraph exposes both nodes' outputs to LangSmith — easy to evaluate quality vs cost.

### 3.14 Checkpointers — making runs durable

Without a checkpointer, state is lost when the process exits. With one, runs are *resumable* and addressable by `thread_id`.

```python
from langgraph.checkpoint.memory import MemorySaver
from langgraph.checkpoint.sqlite import SqliteSaver
from langgraph.checkpoint.postgres import PostgresSaver

# dev
checkpointer = MemorySaver()

# durable, single-machine
import sqlite3
conn = sqlite3.connect("checkpoints.db", check_same_thread=False)
checkpointer = SqliteSaver(conn)

# prod
checkpointer = PostgresSaver.from_conn_string("postgresql://user:pw@host/db")
checkpointer.setup()

graph = g.compile(checkpointer=checkpointer)

config = {"configurable": {"thread_id": "user-42"}}
graph.invoke({"messages": [...]}, config=config)
graph.invoke({"messages": [...]}, config=config)   # continues the same thread
```

### 3.15 Time travel

With a checkpointer you can list past checkpoints, fork from a previous one, or replay:

```python
states = list(graph.get_state_history(config))
# pick any checkpoint and resume from there with new input
graph.invoke(None, config={**config, "checkpoint_id": states[3].config["configurable"]["checkpoint_id"]})
```

### 3.16 Human-in-the-loop with `interrupt`

The modern pattern uses `interrupt(...)` inside a node and `Command(resume=...)` to continue:

```python
from langgraph.types import interrupt, Command

def needs_approval(state):
    decision = interrupt({"reason": "Approve refund?", "amount": state["amount"]})
    return {"approved": decision}

# First call -> graph pauses at interrupt; result includes the interrupt payload.
out = graph.invoke({"amount": 1200}, config=config)

# Resume after a human says yes
graph.invoke(Command(resume=True), config=config)
```

Variants:

- `Command(goto="other_node")` — jump.
- `Command(update={"key": value})` — patch state on resume.

For older code you may see `interrupt_before=["node"]` / `interrupt_after=`. Both still work, but `interrupt()` inside the node is the cleaner pattern.

### 3.17 Long-term memory stores

The `BaseStore` API is for *cross-thread* memory (think: user profile, preferences). It's separate from the per-thread checkpointer.

```python
from langgraph.store.memory import InMemoryStore
store = InMemoryStore()
graph = g.compile(checkpointer=checkpointer, store=store)

# In a node:
def remember(state, *, store):
    store.put(("users", "u123"), "fav_color", {"value": "green"})
```

Use Postgres-backed stores in production.

### 3.18 Multi-agent — supervisor pattern

```text
                ┌── researcher (tools: web search, KB)
   supervisor ──┼── coder (tools: python repl, github)
                └── writer (tools: format markdown)
```

Sketch:

```python
class TeamState(TypedDict):
    messages: Annotated[list, add_messages]
    next: str   # which worker to call

def supervisor(state: TeamState):
    decision = llm.with_structured_output(Routing).invoke(state["messages"])
    return {"next": decision.next}    # one of: researcher | coder | writer | FINISH

def route(state): return END if state["next"] == "FINISH" else state["next"]

g = StateGraph(TeamState)
g.add_node("supervisor", supervisor)
g.add_node("researcher", researcher_subgraph)
g.add_node("coder", coder_subgraph)
g.add_node("writer", writer_subgraph)
g.add_edge(START, "supervisor")
for w in ["researcher","coder","writer"]: g.add_edge(w, "supervisor")
g.add_conditional_edges("supervisor", route, ["researcher","coder","writer", END])
```

### 3.19 Multi-agent — swarm pattern

Agents handoff to *each other* directly via `Command(goto=...)` without a central supervisor.

```python
from langgraph.types import Command

def researcher(state):
    if needs_code(state):
        return Command(goto="coder", update={"messages": [...]})
    return {"messages": [...]}
```

Use when responsibilities are clear and a supervisor adds latency without value.

### 3.20 Subgraphs

Compose graphs out of graphs. The parent passes state through; the child has its own.

```python
researcher = build_researcher_graph().compile()      # a Runnable
parent.add_node("researcher", researcher)            # treat it as a node
```

Use subgraphs to keep cognitive complexity low and to test each agent in isolation.

### 3.21 Streaming tokens vs steps

For UIs you typically want *both*:

- Token stream (so the textbox feels alive) — `stream_mode="messages"`.
- Step updates (so you can show "calling tool: search_orders…") — `stream_mode="updates"`.

Use `stream_mode=["messages", "updates"]` to interleave, and tag events for the frontend.

### 3.22 Tracing with LangSmith

With `LANGSMITH_TRACING=true`, every graph run is captured: node-by-node inputs/outputs, tokens, cost, latency, and the full thread/checkpoint chain. You can fork a trace into a *dataset* for evals.

Add `tags` and `metadata` per call for filtering:

```python
graph.invoke({...}, config={"tags": ["agent","prod"], "metadata": {"user_id": "u42"}})
```

### 3.23 Deployment options

- **LangGraph Platform / Cloud** — managed hosting with persistent threads, SSE streaming, and a UI. Easiest path; pay-per-use.
- **Self-host with FastAPI** — wrap your compiled graph in endpoints; use `PostgresSaver` for checkpointing. Bring your own auth, logging, autoscaling.
- **Lambda / Cloud Run / Container Apps** — fine for stateless flows; you'll need an external checkpointer if you want persistence.

### 3.24 Testing graphs

- **Unit-test nodes** — they're just functions. Pass a fake state; assert the partial update.
- **Integration-test the graph** — `graph.invoke(...)` with stubbed LLMs (`RunnableLambda`) and stubbed tools.
- **Snapshot the trajectory** — assert the sequence of nodes visited (`graph.get_state_history`).
- **Eval in LangSmith** — golden conversations + criteria evals + heuristic checks (no PII, JSON valid, latency < N, cost < $0.0X).

```python
from langchain_core.runnables import RunnableLambda
fake_llm = RunnableLambda(lambda msgs: AIMessage(content="ok"))
def call_model(state): return {"messages": [fake_llm.invoke(state["messages"])]}
```

### 3.25 LangChain vs LangGraph — final word

| You want…                                     | Use         |
|------------------------------------------------|-------------|
| A linear `prompt | model | parser` pipeline    | LangChain LCEL (file 18) |
| A retriever, vector store, document loader    | LangChain   |
| Agent loops, tool calls, retries with state   | LangGraph   |
| Human-in-the-loop, approvals                  | LangGraph   |
| Multi-agent, planner/executor, reflection     | LangGraph   |
| A visual canvas non-engineers will edit       | n8n (file 16) |

**Most production AI apps end up using both LangChain *and* LangGraph.** That's the intended split.

---

## 4. Exercises (Hands-On)

### Beginner

- **A1.** Hello, graph. Two-node graph that uppercases text. Print `app.get_graph().draw_mermaid()` to see the diagram.
- **A2.** Use `MessagesState` and `add_messages` to build a 2-turn chatbot graph (no tools).
- **A3.** Convert it to use `create_react_agent` with a single `multiply(a:int, b:int)` tool.

### Intermediate

- **B1.** ReAct from scratch: implement §3.10 with three tools (search KB, get_order, run_python). Add a `recursion_limit` and a `max_iter` guard inside `should_continue`.
- **B2.** Add `SqliteSaver`; demonstrate that `thread_id="user-42"` retains memory across process restarts.
- **B3.** Add `interrupt()` before the `tools` node when the tool call is "delete_user"; resume with `Command(resume=True)`.
- **B4.** Plan-and-execute: planner produces a list of subtasks; an executor subgraph runs each via `Send`; results are merged.

### Advanced

- **C1.** Supervisor team (researcher / coder / writer) with three subgraphs, structured-output routing, and LangSmith tracing.
- **C2.** Reflection loop on the writer subgraph: critic node scores 1–5; if < 4, revise. Cap at 3 iterations.
- **C3.** Multi-agent swarm where researcher and coder hand off via `Command(goto=...)`. Compare latency and quality vs the supervisor version.
- **C4.** Production: deploy the graph behind FastAPI with `PostgresSaver`, SSE streaming, basic auth, and LangSmith. Add 10 LangSmith eval pairs and run them in CI.

### Capstone — "QA Triage Agent (production)"

Take the LangChain RAG capstone (file 18) and graduate it:

1. ReAct agent over `search_runbooks`, `linear_search`, `ci_last_failure` tools.
2. Reflection on the answer with a strict-format critic; revise up to 2x.
3. HITL on `create_linear_issue` — interrupt and require human "approve" before writing.
4. Postgres checkpointing; resumable threads keyed by Slack user id.
5. Streaming SSE endpoint behind FastAPI; small React UI that shows token + step events.
6. LangSmith eval suite of 30 real triage cases; nightly run; gate deploys on score.
7. README with the graph PNG (`draw_mermaid_png`), env matrix, runbook.

---

## 5. Additional Reading & Resources

### Books / long-form
- *Generative AI with LangChain* (2nd ed.) — Auffarth — covers LangGraph.
- *AI Agents in Action* — Manning (early access).

### Official docs
- LangGraph (Python) — <https://langchain-ai.github.io/langgraph/>
- LangGraph (JS) — <https://langchain-ai.github.io/langgraphjs/>
- LangGraph Concepts — <https://langchain-ai.github.io/langgraph/concepts/>
- How-to guides — <https://langchain-ai.github.io/langgraph/how-tos/>
- Reference — <https://langchain-ai.github.io/langgraph/reference/graphs/>
- LangSmith — <https://docs.smith.langchain.com/>
- LangGraph Platform — <https://langchain-ai.github.io/langgraph/cloud/>

### Tutorials
- LangGraph Academy (free) — <https://academy.langchain.com/>
- LangChain Tutorials → "Build an Agent" — <https://python.langchain.com/docs/tutorials/>
- DeepLearning.AI: *AI Agents in LangGraph*.

### YouTube
- LangChain channel — <https://www.youtube.com/@LangChain>
- Lance Martin (LangChain) — multi-agent walkthroughs.
- Greg Kamradt — Data Independent — LangGraph patterns.

### Adjacent
- AutoGen (Microsoft) — alternate multi-agent framework — <https://microsoft.github.io/autogen/>
- CrewAI — opinionated multi-agent — <https://docs.crewai.com/>
- OpenAI Assistants & Anthropic Tools — provider-native alternatives.

---

## 6. Index

- `add_messages` reducer — §3.4
- checkpointers — §3.14
- `Command(resume / goto / update)` — §3.16, §3.19
- conditional edges — §3.5
- `create_react_agent` — §3.9
- cycles & recursion limit — §3.6
- deployment — §3.23
- human-in-the-loop — §3.16
- `interrupt()` — §3.16
- long-term memory store — §3.17
- `MessagesState` — §3.4
- multi-agent (supervisor) — §3.18
- multi-agent (swarm) — §3.19
- parallel via `Send` — §3.7
- plan-and-execute — §3.12
- ReAct from scratch — §3.10
- reducers — §3.4
- reflection / self-critique — §3.13
- state schemas — §3.4
- `StateGraph` — §3.3
- streaming — §3.8, §3.21
- subgraphs — §3.20
- testing — §3.24
- time travel — §3.15
- `ToolNode` — §3.11
- when to use vs LangChain — §3.25

---

## 7. References

1. LangGraph docs (Python) — <https://langchain-ai.github.io/langgraph/>
2. LangGraph Concepts — <https://langchain-ai.github.io/langgraph/concepts/>
3. LangGraph How-to guides — <https://langchain-ai.github.io/langgraph/how-tos/>
4. LangGraph API reference — <https://langchain-ai.github.io/langgraph/reference/graphs/>
5. LangGraph JS — <https://langchain-ai.github.io/langgraphjs/>
6. LangGraph GitHub — <https://github.com/langchain-ai/langgraph>
7. LangGraph Platform — <https://langchain-ai.github.io/langgraph/cloud/>
8. LangSmith — <https://docs.smith.langchain.com/>
9. LangGraph Academy (free course) — <https://academy.langchain.com/>
10. DeepLearning.AI — *AI Agents in LangGraph* — <https://www.deeplearning.ai/short-courses/>
11. AutoGen — <https://microsoft.github.io/autogen/>
12. CrewAI — <https://docs.crewai.com/>
13. Pregel paper (background for the model) — Malewicz et al., 2010.
14. *Generative AI with LangChain*, 2nd ed. — Ben Auffarth (Packt).

---

*End of Subject 17 — LangGraph.*
