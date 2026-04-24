# Vector Databases — Embeddings, ANN Search, and Production RAG

> Subject #13 of the Master Learning Plan.
> Owner: Govind Divekar. Prerequisites: Python (file 04), AI fundamentals (file 01), SQL (file 07). Estimated time: ~40 hours.
> Target outcome: Build and operate a production-grade vector search, pick the right VDB, and test retrieval quality with numbers.

---

## 1. Overview & Learning Outcomes

- Explain what an embedding is and why semantic search works.
- Pick an embedding model (size, dimension, license, domain fit).
- Choose a distance / similarity metric (cosine, dot, L2) appropriately.
- Explain ANN algorithms: flat, IVF, HNSW, PQ, DiskANN; their trade-offs.
- Build vector indexes in pgvector, Chroma, FAISS, Weaviate, Milvus, Qdrant, Pinecone.
- Design a production RAG: chunking, metadata filters, re-rankers, hybrid search.
- Measure retrieval with Recall@k, NDCG, MRR, and end-to-end answer quality.
- Test a vector search layer for correctness, latency, and drift.

---

## 2. Detailed Syllabus

1. Embeddings 101 — 2h
2. Similarity metrics: cosine / dot / L2; normalization — 2h
3. Exact vs approximate nearest neighbor (ANN) — 2h
4. ANN algorithms: flat, IVF, PQ, HNSW, DiskANN — 4h
5. Chunking strategies — 2h
6. pgvector (Postgres) — 3h
7. FAISS (local, library) — 3h
8. Chroma (dev), Qdrant (prod, open source) — 3h
9. Weaviate, Milvus, Pinecone — 3h
10. Metadata filtering (pre vs post) — 2h
11. Hybrid search (BM25 + vectors, RRF) — 2h
12. Re-rankers (cross-encoders, Cohere Rerank) — 2h
13. Production RAG architecture — 3h
14. Evaluation: Recall@k, NDCG, MRR — 2h
15. Testing VDBs: latency, drift, data-quality — 3h
16. Cost, ops, and scaling — 2h

---

## 3. Topics — Explanations with Examples

### 3.1 What is an embedding?

**Concept.** A fixed-length numeric vector that represents meaning. Two pieces of text that mean similar things end up near each other in vector space. Common dimensions: 384 (MiniLM), 768 (nomic-embed-text, BGE-base), 1024 (mxbai-embed-large, BGE-large), 1536 (OpenAI `text-embedding-3-small`), 3072 (`text-embedding-3-large`).

```python
from sentence_transformers import SentenceTransformer
model = SentenceTransformer("BAAI/bge-small-en-v1.5")
vec = model.encode("flaky integration tests", normalize_embeddings=True)
print(vec.shape)  # (384,)
```

### 3.2 Similarity metrics

- **Cosine** — angle between vectors; insensitive to magnitude. Default for most text embeddings.
- **Dot product** — magnitude-sensitive; good for models that produce unnormalized vectors (e.g., OpenAI small/large are already normalized).
- **L2 (Euclidean)** — straight distance; fine for image embeddings, less common for text.

If vectors are L2-normalized, cosine similarity equals `1 - (L2^2)/2` — many systems store only L2 under the hood.

### 3.3 Exact vs ANN

**Exact** — compare the query to every vector. O(N·d). Fine for ≤ 100k vectors on a single machine.
**Approximate** — trade a tiny amount of recall for huge speed. Essential at million-plus scale.

### 3.4 ANN algorithms

- **Flat / brute force.** Exact. No recall loss. Linear in N.
- **IVF (Inverted File).** Cluster first (k-means), search only a few clusters (`nprobe`). Recall/latency tunable.
- **PQ (Product Quantization).** Compress each vector to a code; cuts memory 8–32×; some recall loss.
- **IVF-PQ.** Combine. Workhorse for very large indexes.
- **HNSW (Hierarchical Navigable Small Worlds).** Graph-based; excellent recall/latency up to ~100M. Memory-heavier.
- **DiskANN / Vamana.** Disk-resident graph; billions of vectors on modest RAM.
- **ScaNN** (Google) — blend of quantization + anisotropic loss; strong quality.

**Rule of thumb.** Start with HNSW (`M=16, ef_construction=200`). Move to IVF-PQ only if memory forces it.

### 3.5 Chunking strategies

Nothing matters more to retrieval quality than chunking. Options:

- **Fixed size, token-based** — simple; 300–800 tokens with 50–150 overlap.
- **Recursive character splitting** — respects paragraphs/sentences.
- **Semantic chunking** — split when embedding similarity between adjacent sentences drops below a threshold.
- **Structural** — split on headings / Markdown H2, code blocks, PDF pages.

Always store the source document id + offsets so you can show citations.

### 3.6 pgvector

**Why.** Postgres already stores your business data; pgvector lets you add vectors without adding a new system. Great up to ~10M vectors per index with HNSW.

```sql
CREATE EXTENSION IF NOT EXISTS vector;
CREATE TABLE chunks (
  id bigserial PRIMARY KEY,
  doc_id text NOT NULL,
  text text NOT NULL,
  embedding vector(384)
);
-- HNSW index
CREATE INDEX ON chunks USING hnsw (embedding vector_cosine_ops)
  WITH (m=16, ef_construction=200);

-- Insert
INSERT INTO chunks (doc_id, text, embedding) VALUES
  ('d1','Playwright is a test framework', '[0.11,0.02, ... ,0.03]');

-- Query (nearest cosine)
SELECT doc_id, text, 1 - (embedding <=> '[0.1,0.0, ... ,0.05]') AS sim
FROM chunks
ORDER BY embedding <=> '[0.1,0.0, ... ,0.05]'
LIMIT 5;
```

### 3.7 FAISS

**Why.** Pure library (no server). Fast, Facebook-maintained. Great for offline indexing, experiments, and embedding alongside a Python service.

```python
import numpy as np, faiss
d = 384
index = faiss.IndexHNSWFlat(d, 16)
index.hnsw.efConstruction = 200
vecs = np.random.randn(10000, d).astype("float32")
faiss.normalize_L2(vecs)
index.add(vecs)

q = np.random.randn(1, d).astype("float32"); faiss.normalize_L2(q)
D, I = index.search(q, 5)
print(I[0])
```

Persist: `faiss.write_index(index, "my.index")`; `faiss.read_index("my.index")`.

### 3.8 Chroma (dev) and Qdrant (prod, open source)

**Chroma.**
```python
import chromadb
c = chromadb.PersistentClient(path="./chroma")
col = c.get_or_create_collection("docs")
col.add(ids=["1","2"], documents=["Playwright","Selenium"], metadatas=[{"k":"qa"}, {"k":"qa"}])
print(col.query(query_texts=["test framework"], n_results=2))
```

**Qdrant.**
```python
from qdrant_client import QdrantClient
from qdrant_client.http.models import Distance, VectorParams, PointStruct

q = QdrantClient(url="http://localhost:6333")
q.recreate_collection("docs",
  vectors_config=VectorParams(size=384, distance=Distance.COSINE))
q.upsert("docs", points=[PointStruct(id=1, vector=[0.1]*384, payload={"k":"qa","text":"Playwright"})])
hits = q.search("docs", query_vector=[0.1]*384, limit=3, query_filter={"must":[{"key":"k","match":{"value":"qa"}}]})
print(hits)
```

### 3.9 Weaviate, Milvus, Pinecone

- **Weaviate.** Hybrid search + generative modules. GraphQL + REST. Open source + managed.
- **Milvus.** High scale, strong IVF/PQ story; GPU build available.
- **Pinecone.** Fully managed SaaS; the easiest to operate; pricey but boring (in a good way).

Feature cheat-sheet:

| Feature | pgvector | Chroma | Qdrant | Weaviate | Milvus | Pinecone |
|---|---|---|---|---|---|---|
| Self-host | Yes | Yes | Yes | Yes | Yes | No |
| Managed | No | Limited | Yes | Yes | Yes (Zilliz) | Yes |
| Metadata filter | Yes | Yes | Excellent | Yes | Yes | Yes |
| Hybrid search | Via Postgres FTS | Coming | Yes | Yes | Yes | Yes |
| Scale | 10M+ | 1M | 100M+ | 100M+ | billions | billions |
| Index types | HNSW, IVFFlat | HNSW | HNSW | HNSW | IVF, HNSW, DiskANN, ScaNN | proprietary |

### 3.10 Metadata filtering: pre vs post

**Pre-filter.** Apply metadata filter first, then ANN over the subset. Correct recall but slower and sometimes unsupported by the index.
**Post-filter.** ANN first, then drop non-matching hits. Fast but can return <k results — recall quietly drops.

Qdrant, Weaviate, Milvus do high-quality pre-filter. pgvector pre-filters with a partial index or a CTE.

### 3.11 Hybrid search (BM25 + vectors)

**Concept.** Keyword search (BM25) excels at rare tokens (IDs, acronyms); vectors excel at paraphrase. Hybrid combines them via **Reciprocal Rank Fusion (RRF)**:

`rrf_score(d) = sum_i 1 / (k + rank_i(d))` with `k=60` typically.

```python
def rrf(ranks: list[list[str]], k: int = 60) -> dict[str, float]:
    scores = {}
    for rank in ranks:
        for pos, doc in enumerate(rank):
            scores[doc] = scores.get(doc, 0) + 1.0 / (k + pos + 1)
    return scores
```

Weaviate, Qdrant (v1.10+), Elasticsearch (with kNN + BM25) support hybrid natively.

### 3.12 Re-rankers

**Concept.** Bi-encoders (for ANN) are fast but imprecise. Cross-encoders read the query and each candidate together and score — 10–100× slower but much more accurate. Use one only on the top ~50 after ANN.

```python
from sentence_transformers import CrossEncoder
ce = CrossEncoder("cross-encoder/ms-marco-MiniLM-L-6-v2")
scores = ce.predict([(q, d) for d in top50])
```

Cohere's hosted `rerank` API is a common alternative.

### 3.13 Production RAG architecture

Reference shape:

1. **Indexing pipeline** — loaders → cleaner → chunker → embedder → VDB + metadata.
2. **Query path** — query rewriter → (optional HyDE / multi-query) → hybrid retrieval → re-ranker → LLM with citations.
3. **Observability** — log query, retrieved ids, re-rank scores, final answer; sample for human eval.
4. **Eval harness** — golden set + metrics; runs nightly.
5. **Guardrails** — prompt injection filter, sensitive-data classifier on retrieved context.

### 3.14 Evaluation: Recall@k, NDCG, MRR

- **Recall@k** = fraction of queries where the correct doc appears in top k.
- **MRR** = mean reciprocal rank of the first correct doc.
- **NDCG@k** = rewards placing relevant docs higher; normalizes by ideal ordering.

```python
def recall_at_k(expected: list[set[str]], retrieved: list[list[str]], k=5):
    hits = sum(1 for e, r in zip(expected, retrieved) if e & set(r[:k]))
    return hits / len(expected)
```

Golden-set creation: 50–200 queries with the id of the single best doc (or a small relevant set). Spend the time on this — your eval is only as good as your golden set.

### 3.15 Testing a vector DB in CI

- **Correctness.** For a known query, the closest neighbor must be the planted doc.
- **Latency.** p95 under a target (e.g., 80 ms at 10k vectors).
- **Recall regression.** Recall@5 on the golden set must not drop by >2% between releases.
- **Drift.** Re-embed + re-score periodically; compare distributions.
- **Metadata filter correctness.** Golden cases with and without filter should agree on relevant doc ids.
- **Cold cache vs warm.** Measure both.

**pytest example.**
```python
def test_recall_at_5_regression(vdb, golden):
    recalls = []
    for q in golden:
        hits = [h.id for h in vdb.search(q["text"], k=5)]
        recalls.append(1 if q["expected_id"] in hits else 0)
    assert sum(recalls)/len(recalls) >= 0.82  # previous baseline was 0.84
```

### 3.16 Cost, ops, scaling

- **Memory math.** HNSW is ~1.5–2× the raw vector memory. 10M × 768-dim float32 ≈ 30 GB raw → ~55 GB HNSW.
- **Use int8 / fp16 quantization** (pgvector `halfvec`, Qdrant scalar quantization) to halve memory.
- **Sharding.** Most managed VDBs shard for you; self-hosted Qdrant/Milvus require planning.
- **Hot-standby replicas** for HA; snapshot to object storage.
- **Re-embedding.** Changing the embedding model is a full rebuild — version the index.

### 3.17 Common pitfalls

- Storing raw text and vector in different systems and letting them drift.
- Mixing normalized and unnormalized vectors in the same index.
- Trusting cosine similarity from a model you haven't L2-normalized.
- Evaluating only on nearest neighbor, never measuring answer quality end-to-end.
- Over-chunking: tiny chunks lose context; over-large chunks dilute relevance.

---

## 4. Exercises

### Beginner
- **A1.** Generate 100 embeddings with `sentence-transformers`; compute cosine similarity between 5 pairs manually.
- **A2.** Stand up pgvector in Docker; store 1k chunks; run a k=5 query.
- **A3.** Same dataset in Chroma and FAISS; compare results across backends.

### Intermediate
- **B1.** Build a local RAG with Qdrant + Ollama embeddings + llama3 — your docs folder.
- **B2.** Add metadata (author, year, tag) and test pre- vs post-filter recall.
- **B3.** Implement hybrid search with Elasticsearch BM25 + pgvector using RRF in Python.
- **B4.** Add a cross-encoder re-ranker on top-20 and measure lift on Recall@5.

### Advanced
- **C1.** Build a golden-set evaluator with Recall@k / MRR / NDCG and fail the build on >2% regression.
- **C2.** Test index rebuilds: switch from IVFFlat to HNSW in pgvector; measure latency / recall.
- **C3.** Benchmark 3 embedding models (MiniLM, BGE-base, nomic) on your golden set; pick by Recall@5 + latency.
- **C4.** Drift experiment: inject a distribution shift (new jargon corpus); re-measure Recall@5; propose mitigation.

### Capstone — "Production-grade RAG"

Deliver a hosted (or Docker-compose) RAG over a corpus you own (notes, docs, tickets). Must include:
- Ingestion CLI that re-chunks on demand and versions the vector index.
- Qdrant (or pgvector) with metadata filters and a scalar-quantized collection.
- Hybrid search + cross-encoder re-ranker.
- Evaluation harness reporting Recall@k / MRR / NDCG, plus LLM-as-judge answer quality.
- CI job that fails on metric regression and alerts on p95 latency > 150 ms.
- A README explaining trade-offs made, cost at 10k / 1M / 10M docs, and a runbook.

---

## 5. Additional Reading & Resources

### Books
- *AI Engineering* (Chip Huyen, 2025).
- *LLMs in Production* (Manning).
- *Building LLM Powered Applications* (Manning).

### Docs & deep dives
- pgvector repo — <https://github.com/pgvector/pgvector>
- FAISS docs & wiki — <https://github.com/facebookresearch/faiss/wiki>
- Qdrant docs — <https://qdrant.tech/documentation/>
- Weaviate docs — <https://weaviate.io/developers/weaviate>
- Milvus docs — <https://milvus.io/docs>
- Pinecone docs — <https://docs.pinecone.io/>
- Sentence-Transformers — <https://www.sbert.net/>
- MTEB leaderboard (embeddings benchmark) — <https://huggingface.co/spaces/mteb/leaderboard>

### Papers
- HNSW — Malkov & Yashunin, 2016.
- Product Quantization — Jégou, Douze, Schmidt, 2011.
- DiskANN — Subramanya et al., 2019.
- ColBERT / ColBERTv2 — Khattab et al. (late-interaction retrieval).
- BGE / M3 — BAAI group.

### Courses / YouTube
- DeepLearning.AI "Vector Databases: from Embeddings to Applications."
- Pinecone "Learn" section.
- James Briggs YouTube channel (vector search, RAG).
- Chip Huyen's AI Engineering Substack.

---

## 6. Index

- ANN — §3.3, §3.4
- BGE (BAAI) — §3.1, §5
- chunking — §3.5
- Chroma — §3.8
- cosine similarity — §3.2
- cross-encoder re-rank — §3.12
- DiskANN — §3.4
- dot product — §3.2
- drift — §3.15, exercise C4
- embeddings — §3.1
- FAISS — §3.7
- golden-set evaluation — §3.14, §3.15
- HNSW — §3.4, §3.6, §3.7
- hybrid search / RRF — §3.11
- IVF / IVF-PQ — §3.4
- L2 distance — §3.2
- metadata filter (pre vs post) — §3.10
- Milvus — §3.9
- MRR — §3.14
- NDCG — §3.14
- normalization — §3.2
- pgvector — §3.6
- Pinecone — §3.9
- PQ (Product Quantization) — §3.4
- Qdrant — §3.8
- quantization (scalar/halfvec) — §3.16
- re-ranker — §3.12
- Recall@k — §3.14
- RRF — §3.11
- ScaNN — §3.4
- Sentence-Transformers — §3.1
- Weaviate — §3.9

---

## 7. References

1. Malkov & Yashunin, *Efficient and robust approximate nearest neighbor search using Hierarchical Navigable Small World graphs*, 2016.
2. Jégou et al., *Product Quantization for Nearest Neighbor Search*, IEEE TPAMI 2011.
3. Subramanya et al., *DiskANN: Fast Accurate Billion-point NN Search on a Single Node*, NeurIPS 2019.
4. pgvector GitHub — <https://github.com/pgvector/pgvector>
5. FAISS wiki — <https://github.com/facebookresearch/faiss/wiki>
6. Qdrant Documentation — <https://qdrant.tech/documentation/>
7. Weaviate Documentation — <https://weaviate.io/developers/weaviate>
8. Milvus Documentation — <https://milvus.io/docs>
9. Pinecone Documentation — <https://docs.pinecone.io/>
10. Sentence-Transformers Docs — <https://www.sbert.net/>
11. MTEB Leaderboard — <https://huggingface.co/spaces/mteb/leaderboard>
12. Cohere Rerank docs — <https://docs.cohere.com/docs/rerank>
13. Chip Huyen, *AI Engineering*, O'Reilly 2025.

---

*End of Subject 13 — Vector Databases.*
