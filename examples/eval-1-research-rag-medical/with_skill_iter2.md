# Before picking an architecture: what's actually being chosen

Your real decision is not "vector vs graph vs hybrid." Those are implementations. The underlying choice is how retrieval mechanics will map onto your two failure surfaces — **citation integrity** (a hard constraint, since misattribution is unacceptable) and **query shape dispersion** (drug-interaction lookups behave nothing like multi-paper conceptual synthesis). Decide those surfaces first; the database follows.

## What this problem *structurally* is

Stripped of domain, you are running two different retrieval regimes over one corpus, and most of your quality will come from routing between them rather than from either one being "better." This is isomorphic to the pre-digital library system, where **the catalog** (entity-level: who wrote it, what it is about, what it cites) and **the index** (content-level: where inside a text a specific phrase appears) were always separate artifacts maintained together. Catalogers and indexers did different work, and cross-references between the two were the actual retrieval UX. A graph DB is a catalog. A vector DB is an index. Hybrid is the librarian's reading room. The question is not which artifact — it is whose job it is to route a patron between them, and how the routing is held accountable.

The [couchbase breakdown](https://www.couchbase.com/blog/graph-rag-vs-vector-rag/) shows this splits cleanly in benchmarks: vector wins specific-document queries (~54% vs 35%), graph wins cross-document reasoning (~33% vs 8%). You have both query classes. That is the load-bearing fact.

## The tensions you must hold, not resolve

- **Precision ↔ coverage.** Drug-interaction queries demand exact, citable passages (precision >> recall). Conceptual queries over 50k papers demand synthesis across weak signals (recall >> precision). A single tuning cannot serve both.
- **Legibility ↔ operational surface.** A graph schema makes reasoning legible to auditors and clinicians — it also triples the ops surface (ETL, ontology version drift, entity-resolution regressions) for a team that has never run one in production.
- **Now ↔ later.** Vector-only ships in weeks and gets you real user signal. Graph adds months of entity-extraction and ontology-mapping work before the first query. What you learn in the first 8 weeks of user queries is probably more valuable than the architectural bet itself.
- **Generality ↔ power.** Building atop [UMLS/MeSH/RxNorm](https://ubkg.docs.xconsortia.org/) gives enormous leverage but locks you to their update cadence and coverage gaps. Rolling your own is freedom at a cost you cannot estimate.

Notice what the tensions have in common: none resolves by picking a DB. They resolve by deciding what you will evaluate against, and in what order.

## Three standpoints, kept un-reconciled

**The skeptical maintainer** (12 months from now, inheriting this): the risk is not wrong architecture, it is a hybrid system where no one can tell *why* a citation was returned — whether the graph walk surfaced it, the vector retrieve surfaced it, a reranker promoted it, or a fusion heuristic merged two unrelated results. Favors: vector-first with explicit provenance, graph added only on queries where vector demonstrably fails on a held-out eval set.

**The clinician user** who will trust or distrust the tool after three bad citations: the risk is the system *sounding* confident about a drug interaction that the cited paper does not actually support. Favors: whatever architecture makes the passage-to-claim entailment check *trivial* to surface in the UI. That is closer to vector + span-level provenance than to graph traversal, because the clinician wants the exact sentence, not a subgraph.

**The researcher doing literature synthesis**: the risk is the system missing the cross-paper connection that a careful reader would catch. Favors graph — but only if the graph is built from entities *your corpus actually discusses*, not a generic UMLS import. A graph full of concepts nobody queries is dead weight.

These three disagree. Keep the disagreement. If your design doc resolves them into a single "hybrid is the answer," the decision has been made by aesthetics, not evidence.

## What the team constraint actually means

Three ML engineers with retrieval experience and zero graph-DB production: this is not a soft factor. Graph-DB failure modes (Cypher query planner regressions, entity-resolution drift under re-ingestion, ontology version migrations) are learned through outages, not reading. A team that debugs its first production Neo4j incident during a clinical user's bad experience will lose trust you cannot rebuild. The asymmetry: vector mistakes feel like "bad recall"; graph mistakes feel like "the system lied." In a medical context, the second is terminal.

This suggests a **staged commitment**, not a one-shot bet: ship vector-first with rigorous span-level provenance and an honest eval set of ~200 clinician-graded (query, paper, passage) triples. Log every query. After 6-8 weeks, cluster the queries your vector system demonstrably fails on. If those failures cluster around multi-hop reasoning, *then* add graph layer — scoped to the entity types those queries touch, sitting on UMLS rather than bespoke. If they cluster around chunking or reranking, you saved yourself a graph DB.

## The pattern, named

This is **premature-hybrid-as-risk-hedging** — a close cousin of Conway's law, where teams pick "hybrid" not because they have evidence both layers are needed but because committing feels irresponsible. The antidote is not to resist hybrid but to earn it: let your own query distribution, not benchmark posts, decide whether the second layer pays for itself. The non-negotiable foundation beneath either path is span-level provenance locked in at index time, because citation integrity is not a property you can bolt on.

Sources:
- [Couchbase: Graph RAG vs Vector RAG breakdown](https://www.couchbase.com/blog/graph-rag-vs-vector-rag/)
- [GraphRAG in Production: When Vector Search Hits Its Ceiling](https://tianpan.co/blog/2026-04-09-graphrag-production-when-vector-search-hits-ceiling)
- [Frontiers: Hybrid graph RAG for safe clinical AI](https://www.frontiersin.org/journals/digital-health/articles/10.3389/fdgth.2026.1780700/full)
- [Unified Biomedical Knowledge Graph (UBKG)](https://ubkg.docs.xconsortia.org/)
- [KRAGEN biomedical KG-enhanced RAG](https://pubmed.ncbi.nlm.nih.gov/38830083/)
- [RAG for LLMs in healthcare: systematic review](https://pmc.ncbi.nlm.nih.gov/articles/PMC12157099/)
- [Cataloguing vs. Indexing](https://lis.academy/organising-and-managing-information/cataloguing-vs-indexing-difference/)
