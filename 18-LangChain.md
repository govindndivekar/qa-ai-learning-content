# LangChain — Composable LLM Apps in Python (and JS)

> Subject #18 of the Master Learning Plan.
> Owner: Govind Divekar. Prerequisites: file 04 (Python), file 13 (Vector DB), file 11 (Hugging Face — optional), file 10 (Ollama — optional). Estimated time: ~30 hours.
> Target outcome: Build production-shaped LLM apps with LangChain — RAG, structured output, tools, retrievers, callbacks — using LCEL fluently, traced with LangSmith, and ready to graduate to LangGraph (file 17) when state and cycles enter the picture.

---

## 1. Overview & Learning Outcomes

LangChain is a **framework for composing LLM applications**: a set of integrations (model providers, vector stores, document loaders, search tools), abstractions (prompts, parsers, retrievers, memory), and a composition language called **LCEL** (LangChain Expression Language) that lets you wire these components together with the `|` pipe operator.

LangChain has split into several packages — know which is which:

- `langchain-core` — the abstractions (Runnable, BaseChatModel, etc). Stable.
- `langchain-openai`, `langchain-anthropic`, `langchain-ollama`, `langchain-huggingface`, `langchain-aws`, etc. — provider integrations.
- `langchain-community` — community integrations (many vector stores, loaders).
- `langchain-text-splitters`, `langchain-chroma`, `langchain-postgres` (pgvector), etc. — focused packages.
- `langchain` — the umbrella package; older "chains" and "agents" live here (largely superseded by LCEL + LangGraph).
- `langgraph` — separate package for stateful/graph orchestration (file 17).
- `langsmith` — tracing and evaluation SaaS (free tier; works with self-hosted apps).

By the end of this subject you should be able to:

- Compose runnables with LCEL: `prompt | model | parser`, `RunnableParallel`, `RunnableLambda`, `RunnablePassthrough`.
- Pick the right model class (chat vs LLM, sync vs async, streaming).
- Build a RAG pipeline: load → split → embed → store → retrieve → answer with citations.
- Use structured output via `with_structured_output(...)` and Pydantic.
- Add tools and run them via `bind_tools(...)` (then graduate to LangGraph for full agent loops).
- Trace and evaluate with LangSmith.
- Deploy a chain as an HTTP API with LangServe (or FastAPI).
- Decide when to *not* use LangChain — and use the underlying SDK directly.

---

## 2. Detailed Syllabus

### Part A — Foundations (5 h)

1. The LangChain landscape, packages, versioning — 0.5h
2. Installing and configuring; environment variables; LangSmith setup — 0.5h
3. Chat Models vs LLMs; provider classes (`ChatOpenAI`, `ChatAnthropic`, `ChatOllama`) — 1h
4. Messages: System/Human/AI/Tool — 0.5h
5. Prompt templates: `ChatPromptTemplate`, `MessagesPlaceholder`, `FewShotChatMessagePromptTemplate` — 1h
6. Output parsers: string, JSON, Pydantic, structured output — 1h
7. The Runnable interface — `invoke`, `batch`, `stream`, `astream`, `with_config` — 0.5h

### Part B — LCEL & Composition (4 h)

8. `|` pipe and `RunnableSequence` — 0.5h
9. `RunnableParallel`, `RunnablePassthrough`, `RunnableLambda` — 1h
10. Passing config (tags, metadata, run_name) — 0.5h
11. Streaming end-to-end — 1h
12. Async chains, batching for throughput — 0.5h
13. Caching, retries, fallbacks — 0.5h

### Part C — Retrieval & RAG (6 h)

14. Document loaders (Web, PDF, Notion, Confluence, S3, Slack) — 0.5h
15. Text splitters: character, recursive, markdown, code, token, semantic — 1h
16. Embeddings: OpenAI, Cohere, HuggingFace, Ollama; normalization — 0.5h
17. Vector stores: Chroma, FAISS, pgvector, Qdrant, Weaviate, Pinecone — 1h
18. Retrievers: similarity, MMR, threshold, MultiQuery, ParentDocument, MultiVector, ContextualCompression — 1.5h
19. RAG patterns: stuff, map-reduce, refine, conversational RAG — 1h
20. Citations, source attribution, hybrid search via EnsembleRetriever — 0.5h

### Part D — Tools, Agents, and Structured Output (4 h)

21. Defining tools: `@tool`, `StructuredTool`, function calling — 1h
22. `bind_tools` and tool-calling models — 0.5h
23. Structured output via `with_structured_output` (function call vs JSON mode) — 1h
24. Legacy AgentExecutor (briefly) — why LangGraph replaced it — 0.5h
25. Toolkits: SQL, Python REPL, Tavily Search — 1h

### Part E — Memory, Sessions, Production (4 h)

26. Conversation history & message stores: `RedisChatMessageHistory`, `PostgresChatMessageHistory`, `RunnableWithMessageHistory` — 1h
27. Callbacks and the `BaseCallbackHandler` — 0.5h
28. LangSmith tracing, datasets, evaluators — 1h
29. LangServe (FastAPI wrapper) and plain FastAPI deployment — 1h
30. Concurrency, rate limits, retries, caching — 0.5h

### Part F — Testing, Cost, and Anti-Patterns (3 h)

31. Unit-testing chains with `RunnableLambda` mocks — 1h
32. Evaluation: pairwise, criteria-based, golden datasets — 1h
33. Cost & latency budgets; when to skip LangChain entirely — 1h

---

## 3. Topics — Explanations with Examples

### 3.1 Setup

```bash
pip install -U langchain langchain-core langchain-community langchain-openai langchain-anthropic langchain-ollama \
              langchain-text-splitters langchain-chroma langchain-postgres langchain-huggingface \
              langsmith
```

```bash
# .env (or your shell)
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=...
LANGSMITH_TRACING=true
LANGSMITH_API_KEY=lsv2_...
LANGSMITH_PROJECT=interview-prep
```

Always pin a small version range in `pyproject.toml` — LangChain moves fast.

### 3.2 Chat Models

The unified surface is `BaseChatModel`. Examples:

```python
from langchain_openai import ChatOpenAI
from langchain_anthropic import ChatAnthropic
from langchain_ollama import ChatOllama

oai = ChatOpenAI(model="gpt-4o-mini", temperature=0.2, max_tokens=512, timeout=30)
ant = ChatAnthropic(model="claude-3-5-sonnet-latest", temperature=0)
oll = ChatOllama(model="llama3.1:8b", base_url="http://localhost:11434")
```

All expose the **Runnable** interface: `.invoke(messages)`, `.batch([...])`, `.stream(...)`, `.astream(...)`.

### 3.3 Messages

```python
from langchain_core.messages import SystemMessage, HumanMessage, AIMessage

resp = oai.invoke([
    SystemMessage(content="You are a concise QA copilot."),
    HumanMessage(content="Summarize the test pyramid in 3 bullet points."),
])
print(resp.content)
```

`AIMessage` may carry `tool_calls` (model wants to call a tool) and `usage_metadata` (token counts).

### 3.4 Prompt templates

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a {persona} assistant. Answer in {language}."),
    MessagesPlaceholder("history"),
    ("human", "{question}"),
])

msgs = prompt.format_messages(persona="senior QA", language="English",
                              history=[], question="Define metamorphic testing.")
```

`MessagesPlaceholder` is how you inject prior turns when wiring memory.

### 3.5 LCEL — the pipe

```python
from langchain_core.output_parsers import StrOutputParser

chain = prompt | oai | StrOutputParser()
print(chain.invoke({"persona": "senior QA", "language": "English",
                    "history": [], "question": "What is drift?"}))
```

`a | b` builds a `RunnableSequence`. The output of one is the input of the next. `StrOutputParser` plucks `.content` from the AIMessage.

### 3.6 RunnableParallel / Passthrough / Lambda

```python
from langchain_core.runnables import RunnableParallel, RunnablePassthrough, RunnableLambda

# Run two prompts at once
parallel = RunnableParallel(
    summary=prompt_summary | oai | StrOutputParser(),
    risks=  prompt_risks   | oai | StrOutputParser(),
)
out = parallel.invoke({"document": "..."})
# out -> {"summary": "...", "risks": "..."}

# Insert arbitrary functions
upper = RunnableLambda(lambda s: s.upper())
chain2 = (RunnablePassthrough.assign(question=lambda x: x["question"].strip())
          | prompt | oai | StrOutputParser() | upper)
```

`RunnablePassthrough.assign(...)` is the workhorse for "keep the input dict but compute one new field."

### 3.7 Streaming

```python
for chunk in chain.stream({"persona":"QA", "language":"English",
                           "history":[], "question":"3 LLM red-team tactics?"}):
    print(chunk, end="", flush=True)
```

`.stream()` yields parser-aware chunks (strings if you ended with `StrOutputParser`). For SSE in FastAPI, just `yield chunk`.

### 3.8 Output parsers and structured output

**Old way:** Pydantic + `JsonOutputParser`.

```python
from pydantic import BaseModel, Field
from langchain_core.output_parsers import JsonOutputParser

class Triage(BaseModel):
    severity: str = Field(description="S1|S2|S3|S4")
    component: str
    summary: str

parser = JsonOutputParser(pydantic_object=Triage)
prompt2 = ChatPromptTemplate.from_messages([
    ("system", "Return JSON matching this schema: {schema}"),
    ("human", "{ticket}"),
]).partial(schema=parser.get_format_instructions())
chain3 = prompt2 | oai | parser
```

**Modern way (preferred):** `with_structured_output` uses the model's native function/tool calling for reliability.

```python
structured = oai.with_structured_output(Triage)
triage = structured.invoke("Login broken on iOS Safari, prod, 4xx after submit")
print(triage.severity, triage.component, triage.summary)
```

### 3.9 Document loaders

```python
from langchain_community.document_loaders import WebBaseLoader, PyPDFLoader
from langchain_community.document_loaders import GitLoader, NotionDirectoryLoader

docs = WebBaseLoader(["https://example.com/runbook"]).load()
docs += PyPDFLoader("docs/QA_Strategy.pdf").load()
```

A `Document` is `{ page_content: str, metadata: dict }`. Treat metadata as gold — page numbers, URLs, headings — for citations later.

### 3.10 Text splitters

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter, MarkdownHeaderTextSplitter

splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=150,
                                          separators=["\n\n", "\n", ". ", " ", ""])
chunks = splitter.split_documents(docs)
```

Rules of thumb:

- **Recursive char splitter** is the safe default.
- **Markdown header splitter** preserves heading structure as metadata — great for runbooks.
- **Token splitter** when you need to respect a model's context window precisely.
- **SemanticChunker** (from `langchain-experimental`) splits by embedding similarity — slower but yields nicer chunks for some corpora.

### 3.11 Embeddings

```python
from langchain_openai import OpenAIEmbeddings
from langchain_huggingface import HuggingFaceEmbeddings
from langchain_ollama import OllamaEmbeddings

emb = OpenAIEmbeddings(model="text-embedding-3-small")           # 1536 dims, cheap
# emb = HuggingFaceEmbeddings(model_name="BAAI/bge-base-en-v1.5") # local
# emb = OllamaEmbeddings(model="nomic-embed-text")                # local via Ollama
```

Match metric to model: most modern embedders are **cosine** or **dot** with normalized vectors. (See file 13.)

### 3.12 Vector stores

```python
from langchain_chroma import Chroma
from langchain_postgres import PGVector

vs = Chroma.from_documents(chunks, embedding=emb, persist_directory="./chroma")
# or pgvector
vs = PGVector.from_documents(chunks, embedding=emb,
                             connection="postgresql+psycopg://user:pass@localhost:5432/db",
                             collection_name="docs")
```

For production, pgvector or Qdrant. Chroma is great for dev.

### 3.13 Retrievers

```python
retriever = vs.as_retriever(search_type="mmr",
                            search_kwargs={"k": 6, "fetch_k": 30, "lambda_mult": 0.5})
```

Common types:

- `similarity` — top-k by vector distance.
- `mmr` — Maximal Marginal Relevance — diversifies results.
- `similarity_score_threshold` — cuts low-confidence hits.
- `MultiQueryRetriever` — LLM rewrites the question into N queries and unions results.
- `ParentDocumentRetriever` — index small chunks but return their parent.
- `MultiVectorRetriever` — index summaries/hypothetical Qs, return real docs.
- `ContextualCompressionRetriever` — re-rank or trim with an LLM/cross-encoder.

### 3.14 RAG — three classic chain shapes

**1. Stuff** (simplest, when context fits): concatenate all retrieved docs into the prompt.

```python
from langchain_core.runnables import RunnablePassthrough
from langchain_core.prompts import ChatPromptTemplate

rag_prompt = ChatPromptTemplate.from_messages([
    ("system", "Answer using ONLY the context. Cite sources by [n]."),
    ("human", "Question: {question}\n\nContext:\n{context}"),
])

def format_docs(docs):
    return "\n\n".join(f"[{i+1}] {d.page_content}\n(source: {d.metadata.get('source')})"
                       for i, d in enumerate(docs))

rag_chain = (
    {"context": retriever | format_docs, "question": RunnablePassthrough()}
    | rag_prompt | oai | StrOutputParser()
)
print(rag_chain.invoke("What's our flaky-test policy?"))
```

**2. Map-reduce** — summarize each doc separately, then combine. Use when you can't stuff.
**3. Refine** — answer with the first doc, refine the answer with each subsequent. Quality-leaning, slow, costly.

For **conversational RAG**, contextualize the question against history first (a "history-aware" retriever), then retrieve.

### 3.15 Hybrid retrieval and reranking

```python
from langchain.retrievers import EnsembleRetriever, ContextualCompressionRetriever
from langchain_community.retrievers import BM25Retriever
from langchain.retrievers.document_compressors import CrossEncoderReranker
from langchain_huggingface import HuggingFaceCrossEncoder

bm25 = BM25Retriever.from_documents(chunks); bm25.k = 10
dense = vs.as_retriever(search_kwargs={"k": 10})
hybrid = EnsembleRetriever(retrievers=[bm25, dense], weights=[0.4, 0.6])

reranker = CrossEncoderReranker(model=HuggingFaceCrossEncoder(model_name="BAAI/bge-reranker-base"), top_n=4)
final = ContextualCompressionRetriever(base_compressor=reranker, base_retriever=hybrid)
```

This is the "production RAG" recipe — cross-references file 13.

### 3.16 Tools

```python
from langchain_core.tools import tool

@tool
def get_order(order_id: str) -> dict:
    """Look up an order by its ID."""
    # ... call your API
    return {"id": order_id, "status": "shipped"}

@tool
def search_kb(query: str) -> str:
    """Search the QA knowledge base."""
    docs = retriever.invoke(query)
    return "\n\n".join(d.page_content for d in docs[:3])

tools = [get_order, search_kb]
```

Bind to a model:

```python
llm_with_tools = oai.bind_tools(tools)
ai_msg = llm_with_tools.invoke("What's the status of order 42 and our return policy?")
print(ai_msg.tool_calls)   # list of {name, args, id}
```

You then execute the tools and feed `ToolMessage` results back. **For non-trivial loops use LangGraph (file 17)** — it handles state, retries, and parallel tool calls cleanly. The legacy `AgentExecutor` is being phased out.

### 3.17 Memory — the modern pattern

The old `ConversationBufferMemory` API is deprecated. The modern pattern is **`RunnableWithMessageHistory`** wrapping any chain:

```python
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_core.chat_history import InMemoryChatMessageHistory

store: dict[str, InMemoryChatMessageHistory] = {}

def get_history(session_id: str) -> InMemoryChatMessageHistory:
    if session_id not in store:
        store[session_id] = InMemoryChatMessageHistory()
    return store[session_id]

with_memory = RunnableWithMessageHistory(
    chain, get_history,
    input_messages_key="question",
    history_messages_key="history",
)

with_memory.invoke({"question": "What is metamorphic testing?"},
                   config={"configurable": {"session_id": "user-42"}})
with_memory.invoke({"question": "Give me 2 examples for an image classifier."},
                   config={"configurable": {"session_id": "user-42"}})
```

Swap `InMemoryChatMessageHistory` for `RedisChatMessageHistory`, `PostgresChatMessageHistory`, or `DynamoDBChatMessageHistory` in production. For long-term memory and multi-agent state, use **LangGraph** (file 17).

### 3.18 Callbacks and tracing

```python
from langchain_core.callbacks import BaseCallbackHandler

class TimingHandler(BaseCallbackHandler):
    def on_llm_start(self, *_, **__): self.t = time.time()
    def on_llm_end(self, response, **__): print(f"LLM took {time.time()-self.t:.2f}s")

chain.invoke({"question": "..."}, config={"callbacks": [TimingHandler()]})
```

**LangSmith** is the easiest path: set `LANGSMITH_TRACING=true` and every call is captured — input, output, tokens, cost, latency, with a linkable trace UI.

### 3.19 LangServe — turning chains into HTTP

```python
# server.py
from fastapi import FastAPI
from langserve import add_routes

app = FastAPI(title="QA Copilot")
add_routes(app, chain, path="/qa")
add_routes(app, rag_chain, path="/rag")

# uvicorn server:app --reload
```

Built-ins: OpenAPI docs, streaming, batching, a Playground UI at `/qa/playground`, async support. For full control, just use plain FastAPI and call `chain.ainvoke(...)`.

### 3.20 Caching, retries, fallbacks

```python
from langchain_core.globals import set_llm_cache
from langchain_community.cache import InMemoryCache, SQLiteCache
set_llm_cache(SQLiteCache(database_path=".langchain.db"))

robust = oai.with_retry(stop_after_attempt=3, wait_exponential_jitter=True)
fallback_chain = robust.with_fallbacks([ant])     # try OpenAI; on failure, Anthropic
```

These exist on every Runnable.

### 3.21 Evaluation with LangSmith

Three useful flavors:

- **Heuristic** — exact-match, regex, length, JSON-schema validity.
- **LLM-as-judge** — pairwise preference, criteria (correctness, helpfulness, no-PII), with `langchain.evaluation`.
- **Custom** — code that grades the output (e.g., "does this Python compile and pass these tests?").

```python
from langsmith import Client
from langsmith.evaluation import evaluate

def correctness(run, example):
    return {"key": "correct", "score": int(run.outputs["answer"].lower() == example.outputs["expected"].lower())}

evaluate(lambda x: {"answer": chain.invoke(x["question"])},
         data="qa-golden-set",
         evaluators=[correctness],
         experiment_prefix="rag-baseline")
```

### 3.22 Testing chains in pytest

```python
# Mock a model with a simple Runnable
from langchain_core.runnables import RunnableLambda
fake_model = RunnableLambda(lambda msgs: AIMessage(content="ok"))
chain_under_test = prompt | fake_model | StrOutputParser()

def test_chain_returns_ok():
    assert chain_under_test.invoke({"persona":"qa","language":"en","history":[],"question":"x"}) == "ok"
```

For RAG, mock the retriever the same way: `RunnableLambda(lambda q: [Document(page_content="...")])`.

### 3.23 When *not* to use LangChain

Honest take:

- **Simple, single-prompt apps** — call the provider SDK directly. LangChain adds friction.
- **Heavy-state / cycles** — use **LangGraph** (file 17) instead of legacy `AgentExecutor`.
- **You need an exact prompt format the framework doesn't expose** — drop down a layer.
- **Strict latency budgets** — measure; LCEL's overhead is small, but `with_structured_output` may add a step.
- **You're inside n8n or another orchestration tool** — you may not need LangChain in your code at all (file 16).

A defensible posture: use LangChain for **integrations** (loaders, retrievers, vector stores, LangSmith) and write the orchestration in **LangGraph** for anything stateful.

### 3.24 The 12-factor app for LLM apps

Quick checklist for production:

- Pin model versions in env; monitor drift when providers update silent defaults.
- Cap tokens, set timeouts, retry idempotently.
- Log prompts/outputs with PII redaction (or use LangSmith with field redaction).
- Cache embeddings; never re-embed the same chunk.
- Track per-request cost; alert on outliers.
- Have a "no-LLM" fallback path for outages.
- Eval before deploy: golden set + criteria evals in CI.

---

## 4. Exercises (Hands-On)

### Beginner

- **A1.** "Hello, LCEL." Build `prompt | model | StrOutputParser()` and call `.invoke`, `.batch`, `.stream`. Verify all three work.
- **A2.** Use `with_structured_output(Pydantic)` to extract `{severity, component, summary}` from 5 sample bug reports.
- **A3.** Wrap a chain in `RunnableWithMessageHistory` with an in-memory store; have a 3-turn conversation that demonstrates the model remembering the user's name.

### Intermediate

- **B1.** Build a basic RAG over your QA runbooks (Markdown). Use `RecursiveCharacterTextSplitter` + `OpenAIEmbeddings` + `Chroma`. Print citations with each answer.
- **B2.** Swap Chroma for pgvector; measure ingest latency and retrieval p95 over 1k chunks.
- **B3.** Add hybrid retrieval (`EnsembleRetriever` of BM25 + dense) and a `CrossEncoderReranker`. Compare quality on 20 golden questions.
- **B4.** Write a `Tavily Search` + `Python REPL` toolset; have the model decide which tool to call. Note where the legacy AgentExecutor pattern starts to creak — this is your trigger to learn LangGraph.

### Advanced

- **C1.** LangSmith eval suite: a golden set of 30 Q/A from your runbooks; criteria evaluator (correctness + cites a real source) + heuristic (no hallucinated URLs). Run it on two different chunking strategies; pick the winner.
- **C2.** Deploy your RAG chain via LangServe behind FastAPI; wire SSE streaming to a tiny HTML client; add Redis-backed `RunnableWithMessageHistory`.
- **C3.** Multilingual RAG: detect input language with a fast model, retrieve over the appropriate language corpus, answer in the user's language.
- **C4.** Cost & latency budget: instrument the chain to log per-call tokens & dollars; add a fallback to a cheaper model for queries shorter than N tokens.

### Capstone — "QA Copilot v1"

A production-shaped LangChain app:

1. Ingestion: GitHub repo + Notion + Confluence → loaders → splitter → pgvector. Idempotent re-runs.
2. Hybrid retrieval (BM25 + dense) with cross-encoder reranking.
3. Conversational RAG with `RunnableWithMessageHistory` (Redis).
4. Structured output for triage requests (Pydantic schema).
5. Tools: `search_kb`, `get_linear_issue`, `last_ci_failure`.
6. LangSmith tracing in dev and prod; nightly eval against a 50-question golden set.
7. LangServe HTTP API, container deployed.
8. README with cost dashboard, eval results, and a "graduate to LangGraph" note for the next iteration.

---

## 5. Additional Reading & Resources

### Books
- *Generative AI with LangChain*, Ben Auffarth (Packt) — solid intro.
- *Building LLM Apps* (Valentina Alto) — broader landscape.
- *AI Engineering* (Chip Huyen) — vendor-neutral, but maps to LangChain ideas.

### Official docs
- LangChain (Python) — <https://python.langchain.com/>
- LangChain (JS) — <https://js.langchain.com/>
- LangSmith — <https://docs.smith.langchain.com/>
- LangServe — <https://python.langchain.com/docs/langserve/>
- Concepts overview — <https://python.langchain.com/docs/concepts/>
- LCEL guide — <https://python.langchain.com/docs/how_to/lcel_cheatsheet/>

### Tutorials
- LangChain tutorials index — <https://python.langchain.com/docs/tutorials/>
- DeepLearning.AI short courses (free): "LangChain for LLM Application Development", "Functions, Tools and Agents with LangChain", "Build LLM Apps with LangChain.js".

### YouTube
- LangChain official channel — <https://www.youtube.com/@LangChain>
- Greg Kamradt — *Data Independent* — best LCEL/RAG walkthroughs.
- Prompt Engineering channel (Muhammad Moin).

### Adjacent / when to leave the framework
- OpenAI Cookbook — <https://cookbook.openai.com/>
- Anthropic Cookbook — <https://github.com/anthropics/anthropic-cookbook>
- LlamaIndex (an alternative; great for indexing-heavy apps) — <https://docs.llamaindex.ai/>

---

## 6. Index

- AgentExecutor (legacy) — §3.16, §3.23
- bind_tools — §3.16
- callbacks — §3.18
- caching — §3.20
- ChatPromptTemplate — §3.4
- ContextualCompressionRetriever — §3.13, §3.15
- conversational RAG — §3.14
- cross-encoder reranker — §3.15
- document loaders — §3.9
- EnsembleRetriever — §3.15
- evaluation (LangSmith) — §3.21
- fallbacks — §3.20
- LangServe — §3.19
- LangSmith — §3.18
- LCEL — §3.5
- map-reduce / refine — §3.14
- memory (modern) — §3.17
- messages — §3.3
- MultiQueryRetriever — §3.13
- output parser — §3.8
- ParentDocumentRetriever — §3.13
- PGVector — §3.12
- Pydantic structured output — §3.8
- RAG patterns — §3.14
- retrievers — §3.13
- RunnableLambda / Parallel / Passthrough — §3.6
- streaming — §3.7
- text splitters — §3.10
- tools (`@tool`, `StructuredTool`) — §3.16
- vector stores — §3.12
- when *not* to use — §3.23

---

## 7. References

1. LangChain (Python) docs — <https://python.langchain.com/>
2. LangChain Concepts — <https://python.langchain.com/docs/concepts/>
3. LCEL Cheatsheet — <https://python.langchain.com/docs/how_to/lcel_cheatsheet/>
4. LangSmith — <https://docs.smith.langchain.com/>
5. LangServe — <https://python.langchain.com/docs/langserve/>
6. LangChain Tutorials — <https://python.langchain.com/docs/tutorials/>
7. LangChain GitHub — <https://github.com/langchain-ai/langchain>
8. LangChain JS — <https://js.langchain.com/>
9. *Generative AI with LangChain* — Ben Auffarth (Packt).
10. DeepLearning.AI short courses — <https://www.deeplearning.ai/short-courses/>
11. Greg Kamradt — Data Independent — <https://www.youtube.com/@DataIndependent>
12. Anthropic Cookbook — <https://github.com/anthropics/anthropic-cookbook>
13. OpenAI Cookbook — <https://cookbook.openai.com/>
14. LlamaIndex — <https://docs.llamaindex.ai/>

---

*End of Subject 18 — LangChain.*
