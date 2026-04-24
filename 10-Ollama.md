# Ollama — Running Local LLMs for Dev and QA

> Subject #10 of the Master Learning Plan.
> Owner: Govind Divekar. Prerequisites: Python (file 04), basic LLM concepts (file 01). Estimated time: ~30 hours.
> Target outcome: Run, customize, and integrate local LLMs via Ollama; build a local RAG and an evaluation harness; know when "local" is the right call for a QA task.

---

## 1. Overview & Learning Outcomes

By the end of this subject you should be able to:

- Install and operate Ollama on Windows/macOS/Linux with realistic hardware sizing.
- Pull, list, run, and create custom models using the Modelfile DSL.
- Call the `/api/generate`, `/api/chat`, and `/api/embeddings` endpoints from curl, Python, and TypeScript.
- Build a local RAG pipeline (loader → chunker → embeddings → vector store → retriever → LLM).
- Apply JSON-mode and tool-calling for structured QA tasks.
- Compare local models by quality, latency, and memory.
- Decide when Ollama is the right choice versus a hosted API (cost, privacy, offline, determinism).

---

## 2. Detailed Syllabus

1. What Ollama is & when to use it — 1h
2. Installation & hardware sizing — 2h
3. Model library (Llama, Mistral, Phi, Gemma, Qwen) — 2h
4. Core CLI: pull/run/list/ps/rm/show/cp — 1h
5. Modelfile DSL: FROM / SYSTEM / PARAMETER / TEMPLATE / ADAPTER — 3h
6. Quantization (GGUF Q4_K_M, Q5_K_M, Q8_0) — 2h
7. Embedding models & embeddings API — 2h
8. REST API (`/api/generate`, `/api/chat`, `/api/embeddings`, `/api/show`) — 2h
9. Python client (`ollama` pip) — 2h
10. TypeScript client (`ollama` npm) — 1h
11. LangChain `ChatOllama` and `OllamaEmbeddings` — 2h
12. Building a local RAG — 3h
13. Structured output: JSON mode & schema-constrained prompts — 2h
14. Tool / function calling with llama3.1, qwen2.5, etc. — 2h
15. Performance: num_ctx, num_gpu, concurrency, keep-alive — 1h
16. Security, env vars, Docker, exposing over LAN — 1h
17. Comparing local models: LLM-as-judge + latency benchmark — 1h
18. Costs: electricity vs token-API — 0.5h
19. QA use cases: test-case generation, flaky-test triage, offline CI — 1h
20. When to move off Ollama (vLLM, TGI, llama.cpp, LM Studio) — 0.5h

---

## 3. Topics — Explanations with Examples

### 3.1 What Ollama is and when to use it

**Concept.** Ollama is a lightweight local LLM runtime with a model registry, a Modelfile DSL, and an HTTP API. Under the hood it uses `llama.cpp`. It shines when you want privacy (data never leaves the machine), reproducibility, offline CI, or zero-per-token cost.

**Good fits.** Internal QA data, PII-heavy test inputs, deterministic CI jobs, laptops without internet, early experiments before paying for a hosted API.

**Bad fits.** Frontier-quality needed (GPT-4o, Claude Sonnet 4.6 level), huge concurrency, multi-tenant SaaS with tight latency SLOs.

### 3.2 Installation and hardware sizing

**Install.**

```bash
# macOS / Linux
curl -fsSL https://ollama.com/install.sh | sh
ollama --version
ollama serve &        # starts the daemon on :11434

# Windows: download the .exe from ollama.com/download and install.
```

**Hardware.** Rough memory estimate per quantization:

| Model size | Q4_K_M | Q5_K_M | Q8_0 |
|---|---|---|---|
| 7–8B | ~5 GB | ~6 GB | ~9 GB |
| 13B | ~8 GB | ~10 GB | ~15 GB |
| 70B | ~42 GB | ~52 GB | ~75 GB |

**Rule of thumb.** Have 1.2× the model-size in free RAM/VRAM. With 16 GB RAM / 8 GB VRAM you can comfortably run 7–8B Q4_K_M quantizations. Apple Silicon (M1/M2/M3) uses unified memory; an M2 Pro with 16 GB handles 7B well.

### 3.3 Core CLI

```bash
ollama pull llama3.1:8b           # download
ollama list                        # what's on disk
ollama run llama3.1:8b "Hi"        # one-off
ollama run llama3.1:8b              # interactive REPL
ollama ps                          # running processes
ollama show llama3.1:8b             # system prompt + params
ollama rm llama3.1:8b              # free disk
ollama cp llama3.1:8b my-llama     # clone locally
```

**Key pitfalls.**

- Models vanish only on `rm` — `pull` of a new tag keeps the old one too.
- `ollama ps` is your friend for memory debugging.

### 3.4 Modelfile DSL

A Modelfile customizes a base model (system prompt, defaults, chat template, adapters). It is the local equivalent of a "custom GPT."

```Dockerfile
# File: Modelfile
FROM llama3.1:8b

# Defaults
PARAMETER temperature 0.2
PARAMETER top_p 0.9
PARAMETER num_ctx 8192
PARAMETER stop "<|eot_id|>"

# Persona
SYSTEM """
You are a senior QA engineer. For every requirement you read, produce a
Gherkin scenario with Given/When/Then. Use plain English. No emojis.
"""
```

Build and run:

```bash
ollama create qa-gherkin -f Modelfile
ollama run qa-gherkin "The system shall allow a user to reset a forgotten password."
```

**Key pitfalls.**

- `num_ctx` bigger than hardware comfortably supports → OOM or slow.
- Changing TEMPLATE is easy to get wrong; start from the base model's template with `ollama show --modelfile llama3.1:8b`.

### 3.5 Quantization

**Concept.** Quantization shrinks model weights (originally fp16/bf16) to smaller int formats for faster inference and less memory. GGUF is the file format; `Q4_K_M` is a common sweet spot (4-bit mixed).

Rules:

- `Q4_K_M` — smallest acceptable for most 7–13B tasks.
- `Q5_K_M` — better quality, ~20% more memory.
- `Q8_0` — near-lossless vs fp16, heavy.
- `fp16` — only on serious GPUs.

### 3.6 Embedding models

```bash
ollama pull nomic-embed-text        # 768-dim, general-purpose
ollama pull mxbai-embed-large       # 1024-dim, higher quality
```

Embeddings via REST:

```bash
curl http://localhost:11434/api/embeddings -d '{
  "model": "nomic-embed-text",
  "prompt": "The quick brown fox"
}'
```

### 3.7 REST API essentials

**/api/generate — single-shot.**
```bash
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.1:8b",
  "prompt": "List 3 unit tests for a login form.",
  "stream": false
}'
```

**/api/chat — multi-turn.**
```bash
curl http://localhost:11434/api/chat -d '{
  "model": "llama3.1:8b",
  "messages": [
    {"role":"system","content":"You are a QA copilot."},
    {"role":"user","content":"Write a boundary value test for age 18+."}
  ],
  "stream": false
}'
```

**/api/embeddings — as above in §3.6.**

Key fields: `options` (temperature, top_p, seed, num_ctx), `format: "json"` for JSON mode, `keep_alive` for how long the model stays loaded.

### 3.8 Python client

```bash
pip install ollama
```

```python
import ollama

# Streaming chat
for chunk in ollama.chat(
    model="llama3.1:8b",
    messages=[{"role": "user", "content": "Explain metamorphic testing in 3 sentences."}],
    stream=True,
    options={"temperature": 0.2, "num_ctx": 8192},
):
    print(chunk["message"]["content"], end="", flush=True)

# Embeddings
emb = ollama.embeddings(model="nomic-embed-text", prompt="login page")["embedding"]
print(len(emb))  # 768
```

### 3.9 TypeScript client

```bash
npm i ollama
```

```ts
import ollama from "ollama";

const resp = await ollama.chat({
  model: "llama3.1:8b",
  messages: [{ role: "user", content: "Give 3 negative test cases for a search box." }],
  options: { temperature: 0.2 },
});
console.log(resp.message.content);
```

### 3.10 LangChain integration

```python
from langchain_ollama import ChatOllama, OllamaEmbeddings

llm = ChatOllama(model="llama3.1:8b", temperature=0.2, num_ctx=8192)
emb = OllamaEmbeddings(model="nomic-embed-text")

print(llm.invoke("One-line definition of drift testing.").content)
vec = emb.embed_query("flaky test")
```

### 3.11 Building a local RAG

**Flow.** load docs → chunk → embed → store in FAISS/Chroma → retrieve top-k → prompt LLM with context.

```python
from pathlib import Path
from langchain_community.document_loaders import TextLoader
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_community.vectorstores import FAISS
from langchain_ollama import ChatOllama, OllamaEmbeddings

docs = []
for md in Path("docs").glob("*.md"):
    docs.extend(TextLoader(str(md)).load())

splitter = RecursiveCharacterTextSplitter(chunk_size=800, chunk_overlap=120)
chunks = splitter.split_documents(docs)

emb = OllamaEmbeddings(model="nomic-embed-text")
vs = FAISS.from_documents(chunks, emb)

llm = ChatOllama(model="llama3.1:8b", temperature=0)

def ask(q: str):
    ctx = "\n\n".join(d.page_content for d in vs.similarity_search(q, k=4))
    prompt = f"Use ONLY this context to answer.\n\n{ctx}\n\nQ: {q}\nA:"
    return llm.invoke(prompt).content

print(ask("What is our regression test policy?"))
```

### 3.12 Structured output (JSON mode)

```python
import ollama, json
resp = ollama.chat(
    model="llama3.1:8b",
    messages=[{"role": "user", "content":
        "Extract entities from: 'Book a cab from Mumbai to Pune for Govind tomorrow 6 AM'. "
        "Return JSON with keys origin, destination, passenger, datetime."}],
    format="json",
    options={"temperature": 0},
)
data = json.loads(resp["message"]["content"])
assert set(data) >= {"origin","destination","passenger","datetime"}
```

**Pitfall.** "JSON mode" does not validate your schema; combine with `pydantic` to assert keys and types.

### 3.13 Tool / function calling

Llama 3.1, Qwen 2.5, and Mistral Nemo support tool calls. Ollama surfaces them via the `tools` field on `/api/chat`.

```python
import ollama

tools = [{
    "type": "function",
    "function": {
        "name": "get_weather",
        "description": "Get weather for a city",
        "parameters": {
            "type": "object",
            "properties": {"city": {"type": "string"}},
            "required": ["city"]
        }
    }
}]

resp = ollama.chat(
    model="llama3.1:8b",
    messages=[{"role": "user", "content": "What's the weather in Pune?"}],
    tools=tools,
)
print(resp["message"].get("tool_calls"))
```

**Pitfall.** Tool-call reliability varies model-to-model; always validate outputs before calling the tool.

### 3.14 Performance tuning

- `num_ctx` — context window in tokens. 8192 is a fine default; bigger = more memory.
- `num_gpu` — layers offloaded to GPU. Auto-detected but you can set explicitly.
- `OLLAMA_NUM_PARALLEL` — concurrent requests per model (default 1). Set 2–4 if VRAM allows.
- `keep_alive` — how long a model stays loaded after a request (default `5m`). Set `0` to unload immediately, `-1` to keep forever.

### 3.15 Security

- Default bind: `127.0.0.1:11434`. Do not expose directly to the internet.
- `OLLAMA_HOST=0.0.0.0` only if behind an auth proxy (Caddy, nginx basic-auth, Cloudflare Tunnel).
- Ollama writes models to `~/.ollama/models` (override with `OLLAMA_MODELS`). Disk fills fast; clean unused tags.

### 3.16 Docker deployment

```bash
# CPU only
docker run -d -p 11434:11434 -v ollama:/root/.ollama --name ollama ollama/ollama
docker exec -it ollama ollama pull llama3.1:8b

# GPU (NVIDIA)
docker run -d --gpus=all -p 11434:11434 -v ollama:/root/.ollama --name ollama ollama/ollama
```

### 3.17 Model comparison methodology

1. Pick 3 models (e.g., `llama3.1:8b`, `qwen2.5:7b`, `phi3.5:3.8b`).
2. Build a 50-task eval set — each with question + "gold" answer.
3. Run each model; measure latency and length.
4. Score with LLM-as-judge using a larger hosted model (or human) against the gold answer.
5. Report quality score, median latency, p95 latency, peak memory.

### 3.18 QA use cases

- **Test-case generation** from requirement blurbs (private → local makes sense).
- **Flaky-test triage.** Feed failed pytest stdout + traceback; ask for root-cause hypotheses.
- **Log summarization** of CI logs to surface the one failing assertion.
- **Offline CI evals** for prompt-engineering changes (no API cost, deterministic if you pin seed + temperature 0).

### 3.19 When to move off Ollama

- Need **multi-tenant serving** → vLLM, Text Generation Inference (TGI).
- Need **raw llama.cpp features** (draft-model speculative decoding, specific samplers) → llama.cpp directly.
- Need **GUI for non-devs** → LM Studio.
- Need **frontier quality** → a hosted API (Anthropic, OpenAI, Google, or AWS Bedrock).

---

## 4. Exercises (Hands-On)

### Beginner
- **A1.** Install Ollama, `ollama pull llama3.1:8b`, ask 3 questions from the CLI.
- **A2.** Pull `nomic-embed-text`; get an embedding via curl; print its length.
- **A3.** Write a Modelfile with a SYSTEM persona for "senior QA mentor"; `ollama create qa-mentor -f Modelfile`; verify the tone change.

### Intermediate
- **B1.** Python script that streams a `/api/chat` call token-by-token.
- **B2.** Build a local RAG over a folder of markdown docs using LangChain + ChatOllama + FAISS; ask 5 questions.
- **B3.** JSON-mode entity extractor validated with pydantic on 20 short sentences; report schema-compliance rate.
- **B4.** Run the same prompt on 3 models; measure latency and subjective quality.

### Advanced
- **C1.** Build an LLM-as-judge harness: one small model answers, a larger model (hosted or bigger local) scores. Report Pearson correlation with a 10-sample human gold.
- **C2.** "Test-case triager" CLI: pipe failed pytest output; get a ranked list of hypotheses with confidence.
- **C3.** Dockerize: Ollama + your RAG service in `docker-compose.yml`.
- **C4.** Fine-tune a LoRA with unsloth on a Q&A dataset, export to GGUF, load as `ADAPTER` in a Modelfile.

### Capstone — "Offline QA Assistant"

Python Typer CLI that:
1. Accepts a requirements PDF.
2. Extracts text (pdfplumber).
3. Uses a local model to generate Gherkin scenarios per requirement.
4. Uses a second local model as LLM-as-judge to score each scenario (clarity, coverage, testability).
5. Writes results to SQLite with a `scenarios` and `scores` table.
6. Produces a summary report with median quality score per requirement and overall coverage estimate.

Deliverable: GitHub repo with README, a 2-minute demo video, and a short comparison section against a hosted API (latency, quality, cost/privacy).

---

## 5. Additional Reading & Resources

### Official
- Ollama site — <https://ollama.com/>
- Ollama GitHub — <https://github.com/ollama/ollama> (see `docs/`)
- Modelfile reference — <https://github.com/ollama/ollama/blob/main/docs/modelfile.md>
- API reference — <https://github.com/ollama/ollama/blob/main/docs/api.md>
- Model library — <https://ollama.com/library>

### Complementary tools
- LM Studio — <https://lmstudio.ai/>
- llama.cpp — <https://github.com/ggerganov/llama.cpp>
- unsloth (fast LoRA fine-tuning) — <https://github.com/unslothai/unsloth>
- vLLM — <https://docs.vllm.ai/>

### Learning
- Simon Willison's blog — category "LLMs" — <https://simonwillison.net/>
- Andrej Karpathy: "LLM from scratch" + "Let's build GPT" on YouTube.
- Ollama YouTube talks & community demos (search "Matt Williams Ollama").
- LangChain docs: ChatOllama — <https://python.langchain.com/docs/integrations/chat/ollama/>

### Papers / posts for context
- *Llama 3* technical report — Meta AI.
- *Mistral Nemo / Mixtral* blog posts.
- *QLoRA* paper (for fine-tuning context).

---

## 6. Index

- /api/chat — §3.7
- /api/embeddings — §3.6, §3.7
- /api/generate — §3.7
- ADAPTER — §3.4, exercise C4
- context window (num_ctx) — §3.14
- Docker run — §3.16
- embeddings — §3.6, §3.11
- fine-tuning (LoRA) — exercise C4
- GGUF quantization — §3.5
- hardware sizing — §3.2
- JSON mode — §3.12
- keep_alive — §3.14
- LangChain — §3.10
- LM Studio — §3.19
- LoRA adapter — §3.4
- Modelfile — §3.4
- nomic-embed-text — §3.6
- OLLAMA_HOST / OLLAMA_MODELS / OLLAMA_NUM_PARALLEL — §3.14, §3.15
- performance tuning — §3.14
- Python client — §3.8
- RAG (local) — §3.11
- SYSTEM prompt — §3.4
- TypeScript client — §3.9
- tool calling — §3.13
- vLLM — §3.19

---

## 7. References

1. Ollama GitHub — <https://github.com/ollama/ollama>
2. Ollama Modelfile docs — <https://github.com/ollama/ollama/blob/main/docs/modelfile.md>
3. Ollama API docs — <https://github.com/ollama/ollama/blob/main/docs/api.md>
4. Ollama Model Library — <https://ollama.com/library>
5. LangChain ChatOllama — <https://python.langchain.com/docs/integrations/chat/ollama/>
6. llama.cpp — <https://github.com/ggerganov/llama.cpp>
7. Hugging Face GGUF conversions — <https://huggingface.co/models?sort=trending&search=gguf>
8. unsloth — <https://github.com/unslothai/unsloth>
9. Simon Willison blog, LLMs tag — <https://simonwillison.net/tags/llms/>
10. Karpathy "Let's build GPT" — <https://www.youtube.com/watch?v=kCc8FmEb1nY>

---

*End of Subject 10 — Ollama.*
