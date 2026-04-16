# RAG Architecture for Clinical Research: What to Think About Before Picking

Before picking vector vs. graph vs. hybrid, I'd push back slightly on the framing. The real choice isn't "which database" — it's **what retrieval shape does clinical evidence actually have, and where does each architecture fail safely?** Here's what I'd weigh.

## The citation constraint is doing more work than it looks

"Misattribution is unacceptable" isn't a retrieval-quality issue, it's a **provenance-tracking** issue. Both vector and graph stores can support clean citation *if* you design the chunking, ID scheme, and answer-synthesis step around it. The real risk is the LLM stitching facts from chunk A and attributing them to chunk B during generation. That's solved upstream of your DB choice — via grounded generation patterns (per-claim citation, chunk-level attribution enforcement, answer verification passes). So don't overweight this factor when comparing DBs; weight it when comparing generation pipelines.

## Query-shape asymmetry is the real driver

Your two query classes pull in opposite directions:

- **Drug-interaction lookups** are relational and entity-centric: "does drug X interact with drug Y in population Z?" This is the sweet spot for graph traversal over a biomedical KG (UMLS, RxNorm, DrugBank linkages). Vector search can approximate it but degrades on negation, dosage thresholds, and multi-hop constraints.
- **Conceptual cross-paper questions** ("what mechanisms have been proposed for X?") are semantically diffuse and benefit from dense retrieval with reranking and synthesis over many chunks.

A pure vector system will handle ~70-80% of conceptual queries well but underperform on structured lookups where precision matters most medically. A pure graph system inverts that problem — and building a high-quality biomedical KG from 2M abstracts is a *large* NLP project (entity linking, relation extraction, normalization against UMLS/MeSH), not a weekend.

## The team constraint is probably decisive

Three ML engineers with retrieval experience but no graph-DB production experience is a real signal. Graph DBs (Neo4j, TigerGraph, Neptune) have meaningfully different operational characteristics: Cypher/Gremlin learning curve, index tuning, write amplification on relationship-heavy ingests, and harder observability. "We'll figure it out" for a medical-grade system is expensive. A failed graph rollout 6 months in is worse than a slightly-less-capable vector system shipped in 2 months.

## What I'd actually evaluate

1. **Query distribution in the wild.** Before architecting, instrument a prototype or survey target users. If drug-interaction-style queries are <15% of volume, a hybrid where the KG handles only those is overkill. If they're 40%+, graph capability is load-bearing.
2. **Evaluation harness first.** Build a gold set of ~200 queries spanning both classes, with ground-truth citations validated by a clinician. Without this you can't compare architectures empirically — you'll argue from intuition, which is where medical RAG projects die.
3. **Existing biomedical KGs.** Don't build from scratch. SemMedDB, Hetionet, PrimeKG, and SPOKE already exist. The question becomes "can we use one of these as the graph layer?" — which shifts the build/buy calculus significantly.
4. **Hybrid done cheaply.** The interesting middle path isn't "run both DBs." It's: vector DB as primary retrieval, with a lightweight entity-linking layer that tags queries containing drug/condition entities and enriches retrieval with structured lookups against an existing KG. No graph DB operations burden; you get 80% of the benefit.
5. **Full-text vs. abstract asymmetry.** 50k full-texts and 2M abstracts is 2 different retrieval problems. Full-texts need hierarchical chunking (section-aware); abstracts are already chunk-sized. Your DB choice matters less than your chunking and metadata strategy here.

## My honest prior

Start with a well-engineered vector system (hybrid dense + BM25, strong reranker, entity-aware query routing, rigorous citation enforcement in generation), connected to an off-the-shelf biomedical KG for structured entity queries. Only move to operating your own graph DB once you have evidence the hybrid retrieval is the bottleneck — not before. The team composition and the "misattribution is unacceptable" bar both favor operational simplicity over architectural ambition.

The question to bring to your team isn't "vector or graph?" — it's **"what's our gold eval set, and what retrieval failure modes can we not tolerate?"** Everything else follows from that.
