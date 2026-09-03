# Marketplace — Technical Case Study

## The problem

A multi-product portfolio accumulates valuable capabilities faster than it accumulates clean commercial packaging. A founder can build dozens of useful subsystems — scoring engines, scrapers, generators, matchers, health checks, outreach flows — while most of them remain economically trapped inside larger applications.

The bottleneck is not capability creation. It is **systematic productization**:

- identify atomic units worth selling;
- distinguish self-serve services from platform-only features or human-dependent work;
- classify execution cost and delivery type;
- avoid rebuilding the same underlying logic under a new business label;
- price consistently;
- discover meaningful bundles without brute-force AI evaluation;
- preserve review, payment, and delivery trust.

Marketplace was designed as a portfolio-to-inventory engine rather than a conventional storefront.

## Design goals

1. Extract commercial units from material that already exists.
2. Keep AI classification behind a human publication boundary.
3. Make reusable engineering structure visible across unrelated domains.
4. Avoid pairwise LLM evaluation across the full catalog.
5. Derive an initial price systematically instead of requiring manual pricing for every service.
6. Keep order/payment state transactional.
7. Attach a realistic delivery promise to every service.
8. Share knowledge across extraction, combination, future market intelligence, and feedback engines.

## The core pipeline

```text
project docs / existing capabilities
            ↓
        EXTRACTION
            ↓
service_type + ai_layer + structural_pattern
            ↓
       PATTERN MATCH
            ↓
       HUMAN REVIEW
            ↓
  publish atomic service
            ↓
     COMBINATION ENGINE
 rules-first → LLM on survivors
            ↓
      bundle / catalog
            ↓
    orders + SLA lifecycle
```

## Decision 1 — one knowledge base, not isolated engines

Extraction, Combination, future Market Intelligence, and future Continuous Development share PostgreSQL + pgvector.

The purpose is compounding knowledge. A service extracted today becomes precedent for pattern matching tomorrow and input for bundling later.

### Alternative rejected

Separate storage and logic for every engine.

### Why rejected

That makes each subsystem rediscover the same structure independently and prevents feedback from becoming shared system knowledge.

### Trade-off accepted

Schema coupling. A classification-axis change can affect multiple engines and therefore requires broader migration discipline.

## Decision 2 — cheap structure before expensive intelligence

The Combination Engine does not send every possible service pair to an LLM.

Three deterministic candidate rules operate first:

- pipeline compatibility;
- shared structural primitive;
- shared audience journey.

Only survivors receive model-based synergy judgment.

### Why this matters

Pair counts grow quadratically. With 200 services there are roughly 19,900 unordered pairs. An LLM-per-pair architecture becomes more expensive precisely when the catalog succeeds at growing.

The selected architecture changes the cost driver from **all possible pairs** to **plausible candidate pairs**.

## Decision 3 — classification-derived pricing

Every service carries three classifications:

- `service_type`;
- `ai_layer`;
- `structural_pattern`.

The `ai_layer` determines the initial credit tier.

This removes a major operational bottleneck: manually pricing hundreds of atomic services.

### Important limitation

Implementation is not market validation. The source project explicitly acknowledges that AI-layer complexity has not yet been proven to correlate with willingness to pay across enough real orders. Therefore the pricing rule is a production mechanism, not a validated pricing science result.

## Decision 4 — structural reuse as a first-class entity

A mock interview evaluator and a recitation evaluator can be commercially different but structurally similar. Marketplace models that shared engineering shape explicitly.

The 15-pattern seed library gives the system a vocabulary for recognizing repeated logic across project boundaries.

### Why embeddings

Business labels are inconsistent. Semantic matching helps recognize related structures even when names differ.

### Why embeddings are not enough

Similarity can be wrong. The review queue therefore remains a semantic safeguard.

## Decision 5 — human review between AI and publication

The system intentionally does not publish extracted services directly.

Pydantic can enforce schema validity. It cannot determine whether a plausible `generative` classification should actually be `deterministic`, or whether a candidate is truly atomic.

The review queue therefore represents a deliberate AI-authority boundary:

```text
AI proposes → schema validates → human approves → catalog publishes
```

## Decision 6 — atomic order economics

Credit balance validation and deduction are designed inside one transaction. This protects the order lifecycle from obvious partial-state failures.

The technical content prepared for the project also describes concurrency testing around competing orders, reflecting a useful evidence principle: transactional intent is not treated as proof until the failure mode is exercised.

## Decision 7 — honest SLA instead of universal speed claims

Each service has a delivery class: instant, same-day, or custom. A periodic worker checks approaching and breached orders.

This is a product decision and an engineering decision. Atomic services are unfamiliar products; trust depends heavily on whether the first delivery promise is accurate.

## What was built specifically for Marketplace

Marketplace-specific work includes the 13-table domain model, extraction and combination services, OCR/document ingestion orchestration, pattern matching, marketplace review lifecycle, pricing rules, bundle logic, order/SLA behavior, and domain UI/API surfaces.

Platform-level authentication, billing/credit foundations, and other shared SaaS infrastructure were inherited from a private reusable base and are not presented as unique Marketplace work.

## Verification state

The primary source supports shipped Foundation, Catalog/Orders, Extraction, Combination, the 15-pattern library, and SLA/order logic. Market Intelligence, Continuous Development, and the Custom Request flow are explicitly scaffolded rather than shipped.

The Airtable technical case study additionally records targeted verification scenarios for concurrent credit deduction and OCR graceful degradation. These claims are preserved with that scope; this public evidence repository does not invent a broader automated-test suite count that the supplied source does not provide.

## Lessons

The strongest lesson is that scaling AI productization is not mostly an LLM problem. It is a **classification, governance, reuse, and cost-placement problem**.

The project becomes more credible when AI is not asked to do everything:

- deterministic rules reduce combinatorial cost;
- schema validation catches malformed responses;
- embeddings suggest structural reuse;
- human review catches plausible semantic mistakes;
- transactional boundaries protect commerce;
- delivery classes constrain promises.

The architecture is strongest precisely where each mechanism is used for the class of problem it handles well.
