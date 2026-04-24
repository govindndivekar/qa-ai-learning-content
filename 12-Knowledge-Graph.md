# Knowledge Graphs — Modeling, Querying, and GraphRAG

> Subject #12 of the Master Learning Plan.
> Owner: Govind Divekar. Prerequisites: SQL basics (file 07), Python (file 04). Estimated time: ~40 hours.
> Target outcome: Design a domain ontology, build and query property graphs and RDF graphs, integrate a KG with LLMs for GraphRAG, and test KG-backed systems.

---

## 1. Overview & Learning Outcomes

- Explain the difference between a database, a graph database, and a knowledge graph.
- Model a domain with ontologies (classes, properties, relations, constraints).
- Use both the property-graph model (Neo4j/Cypher) and the RDF model (triples/SPARQL).
- Ingest structured (CSV/JSON) and unstructured (text) data into a KG.
- Perform graph analytics: shortest path, centrality, community detection.
- Integrate a KG with an LLM via GraphRAG and Text-to-Cypher.
- Test a KG for integrity, lineage, query performance, and semantic drift.
- Recognize how KGs appear in Palantir Foundry (Ontology), in AWS Neptune, and in Microsoft / Google public KGs.

---

## 2. Detailed Syllabus

1. KG fundamentals & terminology — 2h
2. Property-graph vs RDF vs hypergraph models — 2h
3. Modeling: entities, relations, ontologies, taxonomies — 3h
4. Neo4j install and Cypher — 5h
5. RDF, RDFS, OWL, SPARQL — 4h
6. Graph analytics (shortest path, PageRank, community) — 3h
7. Ingestion: CSV/JSON loaders, LOAD CSV, APOC, NLP extraction — 4h
8. Graph + LLM: Text-to-Cypher, GraphRAG — 4h
9. Neo4j GDS (Graph Data Science) library — 2h
10. RDF tooling: rdflib, SPARQL endpoints, Wikidata — 2h
11. KG platforms in industry: Palantir Ontology, Neptune, Stardog, TigerGraph — 2h
12. Schema evolution & governance — 2h
13. Testing KGs: integrity, cardinality, performance, semantic regression — 3h
14. Performance: indexes, profiling, partitioning — 2h

---

## 3. Topics — Explanations with Examples

### 3.1 What is a Knowledge Graph?

**Concept.** A KG is a semantic network of entities and typed relationships, combined with a schema (ontology) that gives meaning to them. Unlike a relational DB, the schema is a first-class citizen you can query. Unlike a document DB, relationships are as important as the entities themselves.

**Why use one.** Connected data (supply chains, fraud, org charts), knowledge reasoning, grounding LLM answers against typed facts, enterprise data fabrics.

**Examples in industry.** Google Knowledge Graph, Wikidata, DBpedia, Palantir Foundry Ontology, LinkedIn's Economic Graph, GitHub's dependency graph, pharma target-compound KGs.

### 3.2 Property-graph vs RDF

| Aspect | Property-graph | RDF |
|---|---|---|
| Node has | Labels + key/value properties | URI and triples about it |
| Relationship has | A type + properties | No native edge properties (needs reification) |
| Query language | Cypher (Neo4j, Memgraph), Gremlin | SPARQL |
| Standardized? | De facto via openCypher | Yes — W3C |
| Strength | Pragmatic for apps | Interop, federation, reasoning (OWL) |

Pick property-graph when you're building a product. Pick RDF when interop with public semantic web or heavy reasoning matters.

### 3.3 Modeling: entities, relations, ontology

**A domain example — publishing:**

- Entities: `Author`, `Book`, `Publisher`, `Genre`.
- Relations: `WROTE`, `PUBLISHED_BY`, `IN_GENRE`.
- Properties: `Author.name`, `Book.isbn`, `Book.publishedYear`.
- Constraints: every `Book` must have exactly one `Publisher`; `isbn` is unique.

The ontology is the set of these rules. You can express it informally (a README) or formally (OWL / Foundry Ontology Manager).

### 3.4 Neo4j install & first queries

```bash
# Docker
docker run --name neo4j -p 7474:7474 -p 7687:7687 \
  -e NEO4J_AUTH=neo4j/test12345 neo4j:5
# Open http://localhost:7474
```

```cypher
// Create
CREATE (a:Author {name: 'Ruth Ozeki', nationality: 'US/Canada'})
CREATE (b:Book {title: 'A Tale for the Time Being', year: 2013})
CREATE (a)-[:WROTE]->(b);

// Read
MATCH (a:Author {name: 'Ruth Ozeki'})-[:WROTE]->(b) RETURN a.name, b.title;
```

### 3.5 Cypher fundamentals

```cypher
// Pattern
MATCH (a:Author)-[:WROTE]->(b:Book)-[:IN_GENRE]->(g:Genre {name:'Fiction'})
WHERE b.year >= 2010
RETURN a.name, b.title, b.year
ORDER BY b.year DESC
LIMIT 25;

// Aggregate
MATCH (a:Author)-[:WROTE]->(b:Book)
RETURN a.name, count(b) AS books
ORDER BY books DESC;

// Upsert (MERGE)
MERGE (p:Publisher {name:'Penguin'})
  ON CREATE SET p.founded = 1935;

// Variable-length path
MATCH (s:Person {id:1})-[:KNOWS*1..3]-(f:Person)
RETURN DISTINCT f.id;
```

**Constraints and indexes.**

```cypher
CREATE CONSTRAINT book_isbn IF NOT EXISTS FOR (b:Book) REQUIRE b.isbn IS UNIQUE;
CREATE INDEX author_name IF NOT EXISTS FOR (a:Author) ON (a.name);
```

### 3.6 RDF, triples, SPARQL

**Concept.** RDF represents everything as triples `(subject, predicate, object)`. URIs identify resources; literals hold data values. RDFS adds `rdfs:Class` and `rdfs:subClassOf`; OWL adds richer constraints (equivalent classes, cardinalities, transitive properties).

**Example triples (Turtle syntax).**

```ttl
@prefix ex:   <http://example.org/> .
@prefix foaf: <http://xmlns.com/foaf/0.1/> .

ex:ruth a foaf:Person ;
        foaf:name "Ruth Ozeki" ;
        ex:wrote ex:bookTaleTime .

ex:bookTaleTime a ex:Book ;
                ex:title "A Tale for the Time Being" ;
                ex:year 2013 .
```

**SPARQL query.**

```sparql
PREFIX ex:   <http://example.org/>
PREFIX foaf: <http://xmlns.com/foaf/0.1/>

SELECT ?authorName ?title WHERE {
  ?a foaf:name ?authorName ;
     ex:wrote ?b .
  ?b ex:title ?title .
}
```

### 3.7 rdflib in Python

```python
from rdflib import Graph, Namespace, Literal, RDF

EX = Namespace("http://example.org/")
g = Graph()
g.bind("ex", EX)

g.add((EX.ruth, RDF.type, EX.Person))
g.add((EX.ruth, EX.wrote, EX.bookTaleTime))
g.add((EX.bookTaleTime, EX.title, Literal("A Tale for the Time Being")))

for s, p, o in g.triples((EX.ruth, None, None)):
    print(s, p, o)
```

### 3.8 Graph analytics

**Shortest path.**

```cypher
MATCH p = shortestPath((a:Person {id:1})-[:KNOWS*..6]-(b:Person {id:42}))
RETURN p;
```

**PageRank (Neo4j GDS).**

```cypher
CALL gds.graph.project('g', 'Person', 'KNOWS');
CALL gds.pageRank.stream('g')
YIELD nodeId, score
RETURN gds.util.asNode(nodeId).id AS person, score
ORDER BY score DESC LIMIT 10;
```

**Community detection (Louvain).**

```cypher
CALL gds.louvain.stream('g') YIELD nodeId, communityId
RETURN communityId, count(*) AS n ORDER BY n DESC LIMIT 10;
```

### 3.9 Ingestion at scale

**CSV — LOAD CSV.**

```cypher
LOAD CSV WITH HEADERS FROM 'file:///books.csv' AS row
MERGE (b:Book {isbn: row.isbn})
  ON CREATE SET b.title = row.title, b.year = toInteger(row.year);
```

**APOC batch import** for >1M rows; or `neo4j-admin import` for >100M.

**NLP-driven ingestion.** Use spaCy / HF `pipeline("ner")` to extract entities/relations, validate against the ontology, and upsert.

### 3.10 GraphRAG and Text-to-Cypher

**Text-to-Cypher.** Use an LLM to translate a natural-language question into a Cypher query:

```python
from langchain_neo4j import GraphCypherQAChain
from langchain_openai import ChatOpenAI
from langchain_neo4j import Neo4jGraph

graph = Neo4jGraph(url="bolt://localhost:7687", username="neo4j", password="test12345")
chain = GraphCypherQAChain.from_llm(ChatOpenAI(model="gpt-4o-mini"), graph=graph, verbose=True)
print(chain.run("Which author wrote the most books since 2010?"))
```

**GraphRAG pattern.**
1. Ingest docs → NER/RE → build KG.
2. At query time, embed the question; find semantically related entities.
3. Traverse the KG around those entities to collect context.
4. Feed context + question to the LLM.

The benefit over vector-only RAG is structured context: you can follow `WROTE → PUBLISHED_BY → FOUNDED_IN` relationships deterministically.

### 3.11 Palantir Foundry Ontology (cross-reference)

Foundry's Ontology is a managed KG with object types, link types, and Actions. See file 05 for QA patterns. The mental model from §3.3 transfers directly — `Object` ≈ property-graph node; `Link` ≈ edge; `Action` ≈ controlled writeback.

### 3.12 Neptune, Stardog, TigerGraph

- **Neptune** — AWS-managed, supports property-graph (Gremlin/openCypher) + RDF (SPARQL).
- **Stardog** — Enterprise RDF with strong reasoning + virtualization.
- **TigerGraph** — Extreme-scale property graph, GSQL language.
- **Memgraph** — In-memory Cypher-compatible; good for streaming use cases.

### 3.13 Schema evolution & governance

**Problems you will meet.**

- Renaming a relation breaks every query and every LLM prompt.
- Adding a property with a null default silently changes aggregates.
- Merging two entities (deduplication) rewires large neighborhoods.

**Practices.** Version the ontology, write migrations (APOC `apoc.refactor` or custom scripts), and keep a CHANGELOG. Represent deprecated labels as `:DeprecatedX` for a grace period.

### 3.14 Testing KGs

**What to test.**

- **Integrity.** Unique constraints, required properties, relation domain/range.
- **Cardinality.** `Book --[:PUBLISHED_BY]--> Publisher` is exactly 1-to-1 (or many-to-1); violations flagged.
- **Referential integrity.** No dangling foreign references.
- **Performance.** Profile heavy queries (`EXPLAIN`/`PROFILE` in Cypher); index coverage.
- **Semantic regression.** A golden set of Q/A must still answer correctly after ontology changes.
- **LLM-facing tests.** Text-to-Cypher correctness on 50 golden questions; GraphRAG answer quality.

**Cypher sanity tests (pytest).**

```python
from neo4j import GraphDatabase

def test_no_orphan_books():
    with GraphDatabase.driver("bolt://localhost:7687", auth=("neo4j","test12345")) as d, d.session() as s:
        r = s.run("MATCH (b:Book) WHERE NOT (b)-[:PUBLISHED_BY]->(:Publisher) RETURN count(b) AS n").single()["n"]
        assert r == 0
```

### 3.15 Query performance

- `PROFILE MATCH ... RETURN ...` shows db hits, rows, cache.
- Anchor your pattern on an indexed label+property.
- Beware `MATCH (a)-[*]-(b)` without bounds — it explodes.
- For write-heavy ingestion, batch with `UNWIND` and periodic commits.

---

## 4. Exercises

### Beginner
- **A1.** Stand up Neo4j in Docker; load the Movies sample dataset (`:play movies`) and run 10 Cypher queries.
- **A2.** Model the publishing ontology (§3.3) in Cypher with constraints; insert 5 authors and 10 books.
- **A3.** Write 5 triples about yourself in Turtle and query them in rdflib.

### Intermediate
- **B1.** Build a movies + actors + directors KG from a CSV of 10k rows using `LOAD CSV`.
- **B2.** Run PageRank with GDS to find the 10 most influential actors.
- **B3.** Implement a simple Text-to-Cypher with LangChain and a local Ollama model; evaluate on 20 questions.
- **B4.** Write 10 pytest integrity checks for your KG.

### Advanced
- **C1.** GraphRAG pipeline end-to-end: ingest 50 Wikipedia pages via HF NER, build the KG, answer 20 questions, and compare answer quality to plain vector RAG.
- **C2.** Add schema evolution: rename a relation, write a migration, keep old queries passing via a temporary alias.
- **C3.** Load 1M nodes with `neo4j-admin import`; profile a heavy query; cut latency by adding indexes.
- **C4.** Federate a local KG with Wikidata via SPARQL SERVICE.

### Capstone — "Enterprise KG + GraphRAG"

Build a KG for a domain you care about (e.g., QA tooling: tools, companies, integrations, tutorials). Ingest from:
1. A curated CSV of entities.
2. 20 blog posts processed with HF NER + relation extraction.
3. Public sources via SPARQL federation.

Deliver:
- A Cypher schema with constraints + indexes.
- A test suite (integrity, cardinality, performance).
- A GraphRAG chatbot (LangChain + Ollama or Bedrock).
- A quality-evaluation harness: 50 golden Q/A, side-by-side vs vector-only RAG.
- Documentation covering ontology, governance, and a test strategy mapping to ISTQB CT-AI concepts.

---

## 5. Additional Reading & Resources

### Books
- *Learning Neo4j 3.x* (Jérôme Baton) / *Graph Databases* (Robinson, Webber, Eifrem) — free PDF via Neo4j.
- *Knowledge Graphs* (Hogan et al., Morgan & Claypool, 2022) — comprehensive.
- *Building Knowledge Graphs: A Practitioner's Guide* (Barrasa & Webber, O'Reilly).
- *Semantic Web for the Working Ontologist* (Allemang, Hendler, Gandon).

### Docs & courses
- Neo4j docs — <https://neo4j.com/docs/>
- Neo4j GraphAcademy (free) — <https://graphacademy.neo4j.com/>
- Neo4j GDS library — <https://neo4j.com/docs/graph-data-science/>
- W3C RDF 1.1 Primer — <https://www.w3.org/TR/rdf11-primer/>
- W3C SPARQL 1.1 Query Language — <https://www.w3.org/TR/sparql11-query/>
- AWS Neptune docs — <https://docs.aws.amazon.com/neptune/>
- Stardog Academy — <https://www.stardog.com/learn/>

### Tools
- rdflib — <https://rdflib.readthedocs.io/>
- APOC — <https://neo4j.com/labs/apoc/>
- LangChain GraphCypherQAChain — <https://python.langchain.com/docs/integrations/graphs/neo4j/>
- Microsoft GraphRAG — <https://github.com/microsoft/graphrag>

### Public KGs to study
- Wikidata — <https://www.wikidata.org/>
- DBpedia — <https://www.dbpedia.org/>
- YAGO — <https://yago-knowledge.org/>

### Videos
- Neo4j YouTube channel (NODES conferences).
- "Introduction to Knowledge Graphs" by Jim Hendler (various lectures).
- Microsoft GraphRAG walkthroughs.

---

## 6. Index

- APOC — §3.9
- cardinality — §3.3, §3.14
- community detection (Louvain) — §3.8
- constraints — §3.5, §3.14
- Cypher — §3.4, §3.5
- DBpedia — §5
- GraphRAG — §3.10, capstone
- Graph Data Science (GDS) — §3.8
- governance / schema evolution — §3.13
- LOAD CSV — §3.9
- Neo4j install — §3.4
- Neptune — §3.12
- ontology — §3.3, §3.13
- OWL — §3.6
- PageRank — §3.8
- Palantir Foundry Ontology — §3.11
- property-graph vs RDF — §3.2
- PROFILE / EXPLAIN — §3.15
- RDF / Turtle — §3.6
- rdflib — §3.7
- shortestPath — §3.8
- SPARQL — §3.6, §3.7
- Stardog — §3.12
- Text-to-Cypher — §3.10
- TigerGraph — §3.12
- triples — §3.6
- Wikidata — §3.6, §5

---

## 7. References

1. Hogan et al., *Knowledge Graphs*, Morgan & Claypool, 2022 — <https://kgbook.org/>
2. W3C RDF 1.1 Primer — <https://www.w3.org/TR/rdf11-primer/>
3. W3C SPARQL 1.1 — <https://www.w3.org/TR/sparql11-query/>
4. Neo4j Documentation — <https://neo4j.com/docs/>
5. openCypher Language Reference — <https://opencypher.org/>
6. Microsoft GraphRAG research — <https://github.com/microsoft/graphrag>
7. AWS Neptune Docs — <https://docs.aws.amazon.com/neptune/>
8. Palantir Foundry Learn — <https://learn.palantir.com/>
9. Wikidata — <https://www.wikidata.org/>
10. *Graph Databases* (Robinson, Webber, Eifrem) — O'Reilly free edition.

---

*End of Subject 12 — Knowledge Graphs.*
