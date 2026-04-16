# Pub/Sub vs Streaming-First: Thinking Through the Decision

Before picking a side, let me flag that the framing "pub/sub vs streaming-first" is slightly a false binary — most mature analytics pipelines end up running both, with the question being which one is the *backbone* and which plays a supporting role. That reframe matters because your real decision is about **where complexity lives**, not which tool wins.

## The load profile argument

5M events/day is ~60 events/second average, with realistic peaks maybe 10–20x that. This is **small** by streaming standards. Both architectures handle it trivially. Don't let vendor benchmarks drive this — throughput is not your constraint. Your constraints are:

- Team skill asymmetry (limited streaming experience)
- 20 heterogeneous consumers with very different semantics
- End-to-end latency and consistency expectations you haven't quantified yet

That last point is the one I'd push on hardest. "End-to-end latency and consistency" means something very different for alerting (seconds, at-least-once is fine) vs ML features (minutes acceptable, but exactly-once matters a lot) vs BI (hours is fine, but schema stability and reprocessing matter more than latency). Your data platform lead might be right, but *for which consumers?*

## What each architecture actually optimizes for

**Pub/Sub backbone (NATS/Kafka)** optimizes for *decoupling and consumer autonomy*. Each of the 20 consumers owns its own processing logic, failure modes, and replay strategy. This scales organizationally — teams don't block on each other. It's weaker when you need joins across streams, windowed aggregations, or exactly-once semantics that span consumers. Kafka specifically shines here because of its log-replay model; NATS JetStream is simpler but a step down on durability guarantees.

**Streaming-first (Flink/Arroyo)** optimizes for *centralized, correct transformations with strong semantics*. Materialized views are genuinely powerful — you define the derived state once, the engine handles watermarks, late data, checkpointing, exactly-once. The cost is a steep operational learning curve (state backends, checkpoint storage, job restarts, backpressure debugging), and a central transformation layer that becomes a coordination bottleneck across 20 consumers.

## The skill gap is not a minor detail

"Limited streaming experience" is load-bearing. Flink in particular has a reputation for being unforgiving when things go wrong — RocksDB state corruption, savepoint incompatibilities across versions, YARN/K8s operator quirks. A team learning streaming while also building a production pipeline for 20 consumers is a project-risk red flag, not a growth opportunity to casually accept. Arroyo is friendlier but younger and smaller ecosystem.

Senior engineers wanting "familiarity and operational simplicity" isn't conservatism — it's them pricing in the cost of on-call at 2am for systems nobody understands yet.

## A path I'd actually recommend

**Kafka (or Redpanda) as the backbone, streaming layer added surgically where it earns its keep.**

1. Kafka handles the event bus, durability, and replay. All 20 consumers subscribe here. Most of them (alerting, audit log, simple analytics, BI ingestion) are fine as plain consumers and don't need streaming semantics.
2. Introduce Flink/Arroyo *only* for the specific consumers that genuinely need materialized views, joins across streams, or exactly-once — likely ML features and possibly a few analytics aggregations. That's maybe 3–5 consumers of the 20.
3. This gives the streaming-skeptical engineers a familiar substrate, gives the data platform lead a real streaming layer to build on, and lets the team grow streaming expertise on a bounded surface area.

## What would change my mind

- If your consistency requirements include cross-consumer exactly-once (rare, but real in financial/audit contexts), streaming-first becomes harder to avoid.
- If the data platform lead has **shipped Flink in production before** and will personally own operations, the skill gap argument weakens significantly.
- If "end-to-end latency" actually means sub-second for most consumers (not just alerting), the polling/consumer-lag model in pub/sub gets awkward.

## The meta-question to take back to the team

Don't debate tools. Debate: *which specific consumers have requirements that pub/sub can't meet cleanly?* Write them down. If the list is short, hybrid with pub/sub backbone. If the list is most of them, streaming-first. The architecture should fall out of the requirements, not the other way around.
