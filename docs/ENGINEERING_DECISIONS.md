# Marketplace — Engineering Decisions

## ADR-01 — Treat capabilities as atomic assets

**Decision:** model a capability as a sellable unit independent of the app where it was first built.

**Why:** monolithic product boundaries hide reusable economic value.

**Trade-off:** atomic packaging adds catalog complexity and forces the system to define clear input/output and delivery boundaries for each service.

---

## ADR-02 — One shared knowledge base for all engines

**Decision:** Extraction, Combination, future Market Intelligence, and future Continuous Development share one PostgreSQL + pgvector knowledge base.

**Alternative:** isolated data stores per engine.

**Why rejected:** isolated engines cannot compound knowledge. Every subsystem would rediscover patterns and service semantics independently.

**Trade-off:** schema changes become cross-engine concerns.

---

## ADR-03 — Three-axis service classification

**Decision:** classify every extracted capability by `service_type`, `ai_layer`, and `structural_pattern`.

**Why:** the axes answer three distinct operational questions: can it be self-served, how should initial execution cost be modeled, and what reusable engineering structure does it share?

**Trade-off:** a plausible but wrong classification can propagate into pricing and discovery unless review catches it.

---

## ADR-04 — Human approval before publication

**Decision:** extracted services enter a review lifecycle rather than publishing automatically.

**Why:** schema-valid AI output is not necessarily semantically correct. Pydantic can validate form, not commercial truth.

**Trade-off:** human review limits full automation throughput, but protects the catalog from confidently wrong classifications.

---

## ADR-05 — Pattern library before rebuilding logic

**Decision:** match new services against a 15-pattern library using embeddings.

**Why:** different domain labels can hide identical logical structures.

**Trade-off:** similarity thresholds can over-match or under-match. Novel capabilities must be allowed to create new patterns instead of being forced into existing ones.

---

## ADR-06 — Rules-first bundle discovery

**Decision:** generate bundle candidates with deterministic rules and send only survivors to an LLM.

**Alternative:** LLM-evaluate every service pair.

**Why rejected:** all-pairs AI evaluation scales quadratically with catalog size.

**Trade-off:** a useful bundle that does not satisfy any deterministic candidate rule may never reach AI evaluation.

---

## ADR-07 — Classification-derived pricing

**Decision:** map `ai_layer` directly to initial credit tiers.

**Why:** manual pricing does not scale with a large automatically extracted catalog.

**Trade-off:** AI execution complexity is only a heuristic for delivery cost and customer value. The project does not claim this mapping is empirically validated across sufficient real order volume.

---

## ADR-08 — Atomic credit/order transaction boundary

**Decision:** treat balance check and deduction as part of the order transaction boundary.

**Why:** commerce cannot tolerate obvious split-brain states between payment and order creation.

**Trade-off:** correctness under concurrency must be verified explicitly; transaction syntax alone is not enough evidence.

---

## ADR-09 — Per-service SLA instead of one universal promise

**Decision:** model delivery as instant, same-day, or custom and monitor approaching breaches.

**Why:** the system sells heterogeneous services with heterogeneous execution requirements.

**Trade-off:** this creates more operational state, but avoids misleading a customer with one blanket speed claim.

---

## ADR-10 — Keep scaffolded engines visibly scaffolded

**Decision:** Market Intelligence, Continuous Development, and the Custom Request flow are documented as scaffolded until execution evidence exists.

**Why:** data models and tables are not equivalent to working product behavior.

**Portfolio rule:** `SCAFFOLDED ≠ IMPLEMENTED ≠ DEPLOYED`.
