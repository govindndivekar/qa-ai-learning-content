# Master Learning Plan — From Scratch to Expert (QA + Dev, AI-Era)

> **Owner:** Govind Divekar
> **Goal:** Reach interview-ready, independent working ability across 13 subjects spanning AI, QA automation, development, cloud, databases, knowledge graphs, and vector search.
> **Timeline:** 6 months (balanced, part-time study — ~15–20 hrs/week). If you can manage 20+ hrs/week, consider 5.5 months.
> **Assumed starting level:** QA tester with some coding exposure.
> **Deliverable style:** Structured study plans (this set of markdown files).

---

## 1. How to Use This Plan

Each subject file is structured identically so you can move between them without friction:

1. **Overview & Learning Outcomes** — what mastery looks like.
2. **Detailed Syllabus** — chapter / module breakdown.
3. **Topics with Explanations & Examples** — short, focused write-ups with runnable code or worked examples.
4. **Exercises (Hands-On)** — graded from beginner → advanced, with a capstone project.
5. **Additional Reading & Resources** — books, docs, courses, YouTube, articles.
6. **Index & References** — keyword index + cited sources.

### Study rhythm (suggested)

| Day cadence | Activity |
|---|---|
| **Mon–Fri (1–2 hrs/day)** | Read 1–2 topics from current subject + do its exercise. |
| **Sat (3–4 hrs)** | Weekly mini-project integrating the week's topics. |
| **Sun (2 hrs)** | Revise via flashcards + write a 1-page summary (Feynman technique). |

### The three-pass method

For every topic, do three passes instead of trying to understand it in one sitting:

- **Pass 1 — Read:** Skim the explanation. Don't try to memorize. Goal: get the shape.
- **Pass 2 — Do:** Run the example code yourself. Break it intentionally. Fix it.
- **Pass 3 — Explain:** Teach it to an imaginary junior, or write a short blog/README. If you can't explain it, you don't know it.

---

## 2. 6-Month Schedule

The schedule stacks foundations early and layers AI/Palantir/cloud in the second half. Interview Prep runs in parallel from Month 4 onwards so you always have fresh material to discuss in interviews.

### Month 1 — Programming Foundations

- **Weeks 1–2:** TypeScript (file 02) — core language + tooling.
- **Weeks 3–4:** Python for AI & Automation (file 04) — core Python + virtualenvs + pytest.

*Why first:* TS and Python are the "lingua franca" for modern QA automation and AI work. Everything downstream uses them.

### Month 2 — Automation & Databases

- **Weeks 5–6:** Playwright for UI + API testing (file 03) — builds directly on TS from Month 1.
- **Weeks 7–8:** SQL & NoSQL (file 07) — queries, schema design, testing data layers.

### Month 3 — Enterprise Stacks & Cloud

- **Weeks 9–10:** Java (file 06) — OOP, collections, streams, JUnit (you will see it in many enterprise shops).
- **Weeks 11–12:** AWS (file 09) — core services + AI services (Bedrock, SageMaker), CCP-level knowledge.

### Month 4 — AI Foundations, Local LLMs, Vector Search

- **Week 13:** AI Dev + QA (file 01) — ML basics, LLM fundamentals, CT-AI concepts.
- **Week 14:** Continue AI — CT-GenAI concepts (prompt engineering, RAG, LLMOps).
- **Week 15:** Ollama (file 10) — run LLMs locally, experiment freely, build a local RAG. In parallel, start Vector Databases (file 13) — embeddings, similarity, ANN fundamentals.
- **Week 16:** Hugging Face (file 11) — transformers, datasets, pipelines, Spaces. Continue file 13 — finish pgvector/Qdrant, evaluation metrics.
- **Interview Prep (file 08)**: begin — 30 min/day on DS&A patterns and behavioral.

### Month 5 — Palantir, Knowledge Graphs, Advanced AI Testing

- **Weeks 17–18:** Palantir Dev + QA (file 05) — Foundry ontology, pipelines, Workshop, AIP; testing patterns. Interleave with Knowledge Graphs (file 12): the Ontology concepts transfer directly — Cypher + RDF basics + GraphRAG.
- **Weeks 19–20:** Return to AI (file 01) — advanced AI testing: adversarial, metamorphic, drift, red teaming, AIP-ready evaluations. Finish Knowledge Graphs capstone (GraphRAG vs vector-only RAG comparison).
- **Interview Prep**: switch focus to system design + QA-leadership scenarios.

### Month 6 — Integration, Projects, Interview Sprint

- **Week 21:** Capstone 1 — Playwright + Python test framework hitting a small FastAPI app backed by Postgres (tests UI, API, DB).
- **Week 22:** Capstone 2 — Hybrid RAG for test generation. Use Ollama + Hugging Face + a vector DB (file 13) + a small knowledge graph (file 12) that captures requirements → features → test cases. Compare vector-only RAG vs GraphRAG on a golden set.
- **Week 23:** Capstone 3 — Deploy something to AWS (Lambda + API Gateway + DynamoDB + Bedrock KB, or SageMaker endpoint) and write the test strategy for it.
- **Week 24:** Full interview sprint (file 08) — mock interviews, system design drills, behavioral polish, portfolio/resume pass.

---

## 3. File Index

| # | File | Subject | Target outcome |
|---|---|---|---|
| 00 | `00-Master-Learning-Plan.md` | This file | Orientation + 6-month schedule |
| 01 | `01-AI-Dev-and-QA.md` | AI (Dev & QA) | ML + LLM fundamentals; test AI systems per ISTQB CT-AI & CT-GenAI |
| 02 | `02-TypeScript.md` | TypeScript | Expert-level TS for automation + Node services |
| 03 | `03-Playwright-UI-and-API-Testing.md` | Playwright | Pro-level Playwright for UI + API + CI |
| 04 | `04-Python-for-AI-and-Automation.md` | Python | Python for ML libs, scripting, pytest, automation |
| 05 | `05-Palantir-Dev-and-QA.md` | Palantir | Foundry/AIP ontology, pipelines, Workshop; QA patterns |
| 06 | `06-Java.md` | Java | OOP, JVM, Spring basics, JUnit/REST-assured |
| 07 | `07-SQL-and-NoSQL-Databases.md` | Databases | Advanced SQL, NoSQL families, intro to vector DBs, DB testing |
| 08 | `08-Interview-Preparation.md` | Interview Prep | DS&A, system design, behavioral, QA & AI interviews |
| 09 | `09-AWS.md` | AWS | Core services + AI services; CCP/SAA readiness |
| 10 | `10-Ollama.md` | Ollama | Run LLMs locally; Modelfile; REST API; local RAG |
| 11 | `11-Hugging-Face.md` | Hugging Face | Transformers, datasets, fine-tuning, Spaces, Inference API |
| 12 | `12-Knowledge-Graph.md` | Knowledge Graph | Ontology design, Neo4j/Cypher, RDF/SPARQL, GraphRAG, KG testing |
| 13 | `13-Vector-Database.md` | Vector Database | Embeddings, ANN (HNSW/IVF/PQ), pgvector/Qdrant/Weaviate/Pinecone, RAG evaluation |

---

## 4. Prerequisites & Environment Setup

Install these before Month 1 so you never break study flow to fight tooling:

- **Git** + a **GitHub** account (you will push every exercise; a public portfolio is part of interview prep).
- **VS Code** with extensions: GitLens, ESLint, Prettier, Python, Pylance, Jupyter, Playwright Test, Java Extension Pack.
- **Node.js LTS** (via `nvm` / `nvm-windows`).
- **Python 3.11+** (via `pyenv` or Anaconda/Miniconda).
- **Java 21 LTS** (via SDKMAN or Adoptium).
- **Docker Desktop** — you will need it for databases, local services, and AWS LocalStack.
- **Postgres** (Docker), **MongoDB Atlas** free tier, **Redis** (Docker).
- **AWS Free Tier** account.
- **Hugging Face** account (free).
- **Ollama** installed locally (Month 4).
- **Postman** or **Bruno** (API testing).

### Folder layout suggestion

```
C:\Users\Govind\Documents\Claude\Projects\Interview Prep\     ← these markdown plans
C:\Dev\learning\
    ├── 01-ai\
    ├── 02-typescript\
    ├── 03-playwright\
    ├── 04-python\
    ├── 05-palantir\
    ├── 06-java\
    ├── 07-databases\
    ├── 08-interview\
    ├── 09-aws\
    ├── 10-ollama\
    ├── 11-huggingface\
    ├── 12-knowledge-graph\
    └── 13-vector-database\
```

Each folder is its own Git repo. Push daily — your commit graph becomes part of your portfolio story.

---

## 5. Progress Tracking

Create a file called `progress.md` in each subject folder with this table, and tick items as you go:

```markdown
| Module | Read | Examples run | Exercise done | Notes |
|---|---|---|---|---|
| 1.1 | ☐ | ☐ | ☐ | |
| 1.2 | ☐ | ☐ | ☐ | |
```

The three ticks form a habit: **Read → Do → Verify**. Never tick "done" if you skipped one of them.

---

## 6. Meta-Skills You Are Building In Parallel

These aren't their own files, but every subject should reinforce them:

- **Reading documentation efficiently.** Almost every interview-critical concept is in the official docs. Practise reading them with purpose.
- **Writing small test plans.** For any new API or feature, write a 1-page test plan (scope, risks, test ideas, test data). This is the single most interview-tested skill in QA.
- **Version control hygiene.** Branch-per-task, clean commit messages, PR descriptions. Employers look at GitHub.
- **Cost & perf awareness in cloud/AI work.** Always know roughly what a request costs and how long it takes. It's the difference between a junior and a senior answer.
- **Learning to prompt LLMs well.** GenAI is now a multiplier in QA. File 01 covers prompt engineering formally per ISTQB CT-GenAI.

---

## 7. How the AI Testing Content Maps to ISTQB

The AI Dev & QA file (01) is explicitly structured to cover both ISTQB syllabi you provided:

- **ISTQB CT-AI v2.0 (2026)** — AI-based systems, ML workflow, data/model testing, neural networks, metamorphic / adversarial / drift testing, ML development testing.
- **ISTQB CT-GenAI v1.0 (2025)** — GenAI foundations, prompt engineering, managing risks (hallucinations, bias, privacy), LLM-powered test infra (RAG, agents, LLMOps), adoption roadmap.

Where the official syllabus is prescriptive about terminology or scope, file 01 uses the same terms so the plan doubles as cert-exam prep if you decide to sit either certification.

---

## 8. When to Ask for Help

If any of the following happens, stop and ask questions (or raise one with me) rather than grinding:

- You've been stuck on a single bug for > 90 minutes.
- A concept "doesn't click" after two study sessions.
- A tool install is eating more than an hour.
- You can't connect how a topic shows up in a real job.

The whole plan is modular — reorder subjects if a specific interview calls for it, and don't feel obliged to finish a subject before starting the next (programming languages especially benefit from interleaved practice).

---

## 9. Success Criteria (End of 6 Months)

You should be able to, without looking things up in the moment:

- Write and debug a TypeScript Playwright test suite (UI + API) with fixtures, POM, and CI.
- Write pytest suites for a Python service, including parametrization and mocking external APIs.
- Stand up a small Java Spring service and test it with JUnit 5 + REST-assured.
- Write non-trivial SQL (window functions, CTEs, EXPLAIN), and design a basic document / KV / vector DB usage.
- Describe and run the ML workflow; explain metamorphic testing, adversarial testing, drift testing, and red-teaming an LLM in your own words.
- Build a local RAG on Ollama and evaluate it with Hugging Face datasets/metrics.
- Deploy a small service to AWS (Lambda / ECS / EC2) and articulate the test strategy for it.
- Describe Palantir Foundry's ontology, pipelines, and Workshop apps, and the QA approach for them.
- Model a domain as a knowledge graph (property graph or RDF), write Cypher/SPARQL, and explain GraphRAG vs vector-only RAG.
- Stand up a production-ish vector search (HNSW index with metadata filters + hybrid search + re-ranker) and measure Recall@k / NDCG / MRR against a golden set.
- Handle a 45-minute system design interview at a QA-leaning or full-stack-junior level.

If any of those feel uncertain after 6 months, that subject needs another focused week — the plan is flexible by design.

---

## References (master-level)

- ISTQB, *Certified Tester AI Testing Syllabus v2.0* (2026). <https://www.istqb.org/>
- ISTQB, *Certified Tester Specialist — Testing with Generative AI (CT-GenAI) v1.0* (2025). <https://www.istqb.org/>
- Playwright docs — <https://playwright.dev/>
- TypeScript handbook — <https://www.typescriptlang.org/docs/>
- Python docs — <https://docs.python.org/3/>
- Hugging Face docs — <https://huggingface.co/docs>
- Ollama docs — <https://ollama.com/library> / <https://github.com/ollama/ollama>
- AWS docs — <https://docs.aws.amazon.com/>
- Palantir learn — <https://learn.palantir.com/>
- *Designing Data-Intensive Applications*, Martin Kleppmann (O'Reilly).
- *Cracking the Coding Interview*, Gayle Laakmann McDowell.

---

*End of master plan. Open the subject files listed in section 3 to begin.*
