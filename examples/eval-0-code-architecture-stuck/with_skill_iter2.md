# Thinking through your pipeline decision

## The gap, stated more precisely than the question

Your real objective is not "pick pub/sub or streaming-first." It is: choose an architecture for the next 18–36 months that the existing team can operate, that serves 20 heterogeneous consumers without forcing them into a single semantic model, and that does not bet the platform on a paradigm the team cannot yet debug at 2am. The decision feels binary because two credible people framed it as binary. That framing is the thing to dissolve first.

A load-bearing number: 5M events/day ≈ 58 events/second average, maybe a few hundred at peak. This is small. Any modern broker or streaming engine handles it trivially. **Throughput is not your binding constraint** — which means it should not drive the choice. Consumer semantics and operational burden should.

## The abstract structure, and why an old analogue helps

Stripped of domain, this is a **fan-out-with-heterogeneous-transformation** problem — one producing surface, many consuming surfaces, each with different needs for freshness, ordering, replay, and derived state. Two strategies recur across domains:

1. **Raw-transport + per-consumer treatment** — deliver the unprocessed substance broadly; each consumer refines what it needs, when it needs it.
2. **Centralized treatment + distribution of refined product** — transform once at the center; ship the finished views downstream.

Municipal water is the cleanest isomorphism: small towns keep raw supply per-site (simple, cheap, each user treats for their own use); cities centralize treatment at plants (expensive to build, but absurd to have 20 households each running a reverse-osmosis rig). The switch point is **number of consumers times overlap of treatment needs**, not volume. Your 20 consumers is past the threshold for *some* shared treatment — but only where needs genuinely overlap.

That reframes the decision: **it is almost certainly a two-layer stack, not an either/or**. The real question is where the center-of-gravity sits.

## The tensions, held as shapes

- **Familiarity ↔ capability ceiling.** Pub/sub now is easier; streaming-first later is expensive to retrofit *only if you did not leave a durable log underneath*.
- **Operational simplicity ↔ end-to-end semantics.** "Exactly-once" is a property of a processor over its own state; it does not survive the journey to 20 heterogeneous consumers unless each is idempotent. The phrase promises more than the architecture can deliver.
- **Consumer homogeneity ↔ heterogeneity.** Analytics + BI can tolerate minutes of lag; alerting cannot; ML features often batch anyway; audit logs want immutability and replay over latency. One architecture collapsed over all of them will be wrong for most of them.
- **Reversibility.** Kafka-or-NATS first leaves streaming-on-top as a clean later move. Streaming-first without a durable log underneath is a harder retreat.

## Three standpoints that genuinely disagree

**The senior backend engineers are substantively right about one thing:** operability is a first-class feature, not a nice-to-have. A team running Flink without deep streaming experience will spend months on checkpointing, backpressure, and state-store recovery before shipping consumer value. That cost is real and usually underestimated by advocates. Starting with a durable log (Kafka, or NATS JetStream if you want lighter ops) and plain consumers is not "legacy" — it is the substrate every streaming engine sits on anyway.

**The data platform lead is substantively right about a different thing:** if you build 20 bespoke consumers each doing their own joins, windowing, and deduplication, you will re-implement a bad stream processor 20 times, inconsistently. For the handful of consumers that genuinely need shared stateful transformations (the BI rollups, the ML feature aggregations, the alerting windows), a streaming layer earns its keep by computing once and serving many. The trap is assuming *all* consumers need this.

**A skeptical maintainer six months from now** asks a question neither side is asking: "what do I do when a consumer is wrong and needs to reprocess the last 30 days?" Pub/sub topics with long retention make this trivial. Pure streaming-first without an upstream durable log makes it a migration project. Replay is the property most often missed in the original debate and most often needed in production.

## The pattern, named

This is **two-layer-masquerading-as-one-layer** — a framing artifact where the real architecture has a durable transport *and* a processing layer, but the debate collapses them into a single tool choice. The elegant handle: **log at the bottom, selective streaming on top**.

Concretely: put Kafka (or NATS JetStream if the team prefers Go-native ops and the consumer count stays under ~50) at the base. All 20 consumers subscribe. For the subset that need shared stateful transformations — probably 3 to 6 of them, not 20 — put a streaming layer above that serves materialized views. If team streaming experience is the bottleneck (you said it is), pick the lightest operational option: **Arroyo** (single binary, Rust, built explicitly for teams without dedicated streaming platform engineers) or a managed materialized-view product (RisingWave, Materialize) rather than self-hosted Flink. Flink is the right tool only if you already have the team to run it, which you have said you do not.

**The move that resolves the debate empirically:** a two-week spike replaying a week of real traffic through Kafka + {two candidate processors, one simple consumer}. Measure operator time, not throughput. The winner is whichever your team can still operate when the person who built it goes on vacation.

Sources:
- [Kafka vs Pub/Sub: Key Differences Explained — Estuary](https://estuary.dev/blog/kafka-vs-pubsub/)
- [Why Not Flink? — Arroyo blog](https://www.arroyo.dev/blog/why-not-flink/)
- [6 Apache Flink Alternatives for Simpler Real Time Analytics — Tinybird](https://www.tinybird.co/blog/flink-alternatives)
- [Materialize vs RisingWave: Comprehensive Comparison](https://materialize.com/guides/materialize-vs-risingwave/)
- [The Role of Materialized Views in Modern Data Stream Processing — Medium](https://gautambangalore.medium.com/the-role-of-materialized-views-in-modern-data-stream-processing-architectures-risingwave-bc3cf1c78c3f)
- [Water distribution system — Wikipedia](https://en.wikipedia.org/wiki/Water_distribution_system)
