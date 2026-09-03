# Marketplace — Engineering Evidence

<p align="center">
  <img src="https://img.shields.io/badge/Marketplace-Atomic%20AI%20Services-5D7896?style=flat-square" alt="Marketplace" />
  <img src="https://img.shields.io/badge/Extraction-Shipped-5F8A7A?style=flat-square" alt="Extraction" />
  <img src="https://img.shields.io/badge/Combination-Shipped-716F9A?style=flat-square" alt="Combination" />
  <img src="https://img.shields.io/badge/Pattern%20Library-15%20Structures-8A7693?style=flat-square" alt="Pattern Library" />
  <img src="https://img.shields.io/badge/Evidence-Claim%20%E2%86%92%20Proof-5F7386?style=flat-square" alt="Evidence" />
</p>

> **What if the most valuable thing inside a SaaS product is not the product itself, but one capability trapped inside it?**

Marketplace is an atomic-service system for turning software capabilities — already built or only documented — into independently classifiable, priceable, bundleable assets.

The engineering problem is not “build a storefront.” It is:

**How do you turn a large, messy portfolio of existing work into structured commercial inventory without manually rediscovering, repricing, and repackaging every capability one by one?**

The system answers that with a pipeline:

**Extract → Classify → Match → Package / Combine → Sell / Deliver → Learn**

This public repository is an **Engineering Evidence / Technical Case Study** for the private production system. It documents architecture, implemented capabilities, design decisions, verification boundaries, and limitations without publishing proprietary production source.

**Live product:** https://marketplace.agentcraft.info  
**API docs:** https://api.marketplace.agentcraft.info/docs  
**Parent brand:** https://agentcraft.info

---

## Why this project exists

Most SaaS products are commercialized as monoliths: one subscription, one application, one large bundle of capabilities.

But real portfolios accumulate value differently. A single app may contain a domain health checker, semantic matcher, scoring engine, formal document generator, scraping pipeline, or outreach workflow that is useful outside the original product that happened to contain it first.

Marketplace treats those capabilities as **atomic assets**.

A capability can originate from:

- a working product;
- an internal engineering system;
- a documented but not yet productized study;
- a reusable structural pattern discovered in prior projects.

The system extracts those units, classifies what they are, maps them against reusable structural patterns, assigns pricing logic, places them behind review, and discovers bundle opportunities across the catalog.

That changes the economic model from:

```text
build app → sell whole app or monetize nothing
```

into:

```text
existing work
   ↓
extract atomic capability
   ↓
classify + pattern-match
   ↓
review
   ↓
sell alone or combine with other capabilities
```

The result is not merely a marketplace catalog. It is a **portfolio-to-inventory engine**.

---

## Current verified status

The source project documents the following implementation state:

| Capability | Evidence status | Current boundary |
|---|---|---|
| Foundation + marketplace data model | **IMPLEMENTED / DEPLOYED** | 13 marketplace-specific tables on the shared platform foundation. |
| Catalog + orders | **IMPLEMENTED / DEPLOYED** | Public catalog, credit checkout, service lifecycle, admin order management. |
| Extraction Engine | **IMPLEMENTED / DEPLOYED** | Text / PDF / DOCX / TXT ingestion, chunking, Arabic OCR fallback, service extraction, classification, review queue. |
| Combination Engine | **IMPLEMENTED / DEPLOYED** | Rules-first candidate generation, LLM evaluation for survivors, manual bundle builder. |
| 15-pattern structural library | **IMPLEMENTED** | Pre-seeded recurring logical structures matched through embeddings. |
| Pricing by `ai_layer` | **IMPLEMENTED** | `none` / `deterministic` / `generative` map to credit tiers. |
| SLA monitoring | **IMPLEMENTED** | Per-service delivery type + periodic approach/breach detection. |
| Market Intelligence | **SCAFFOLDED** | Data model/tables designed; external scraper integration is not claimed as complete. |
| Continuous Development | **SCAFFOLDED** | Feedback data model exists; full self-improvement loop is not claimed as complete. |
| Custom request flow | **SCAFFOLDED** | Tables/flow prepared; not represented as a shipped customer feature. |

A key portfolio rule applies here: **scaffolded engines are not presented as live engines.**

---

## Architecture thesis: one shared knowledge base, four engines

Marketplace is designed around four engines operating on one shared PostgreSQL + pgvector knowledge base.

```text
                         ┌──────────────────────────────┐
                         │      SHARED KNOWLEDGE BASE   │
                         │ PostgreSQL + pgvector        │
                         │ services · patterns ·        │
                         │ bundles · orders · feedback  │
                         └──────────────┬───────────────┘
                                        │
            ┌───────────────────────────┼────────────────────────────┐
            │                           │                            │
            ▼                           ▼                            ▼
     EXTRACTION ENGINE          COMBINATION ENGINE          MARKET INTELLIGENCE
     docs → services            services → bundles          external demand signals
     shipped                    shipped                     scaffolded
            │                           │                            │
            └───────────────────────────┼────────────────────────────┘
                                        ▼
                              CONTINUOUS DEVELOPMENT
                              rejections / refunds / support
                              scaffolded
```

The choice to share one knowledge base is deliberate. A successful extraction enriches the reusable pattern vocabulary; richer pattern knowledge improves future classification; stronger classification improves combination candidates; later feedback can eventually flow back into service quality decisions.

The trade-off is coupling: a schema change is no longer local to one engine. It can affect the semantics of the entire system. That is accepted in exchange for **compounding shared knowledge instead of four isolated learning loops**.

---

# Engine 1 — Extraction

The Extraction Engine turns unstructured project material into candidate atomic services.

Supported source forms in the project record include:

- pasted text;
- PDF;
- DOCX;
- TXT;
- scanned Arabic PDFs through Tesseract OCR fallback.

Long documents are chunked before AI extraction; the project source describes 12k-character chunks respecting paragraph boundaries. Extraction runs asynchronously, and chunk-level results are merged and deduplicated by service name.

Each candidate service is classified on three axes:

### `service_type`

```text
atomic_service | platform_feature | requires_human
```

This answers whether the capability can actually become a self-serve unit.

### `ai_layer`

```text
none | deterministic | generative
```

This controls the initial credit-pricing rule.

### `structural_pattern`

The capability is matched via embeddings against the structural pattern library.

The extraction pipeline also computes confidence/readiness values and recurring-cost information before writing Pydantic-validated candidate rows to the database as `suggested`.

### Why the human review queue remains important

Pydantic can reject malformed structure. It cannot prove that a plausible classification is semantically correct.

A service can therefore be syntactically valid and still have the wrong `ai_layer`, wrong pattern, or wrong product boundary.

That means the current architecture intentionally keeps **human review between AI classification and publication**.

This is not a temporary embarrassment hidden from the portfolio. It is a real control boundary around AI authority.

---

# Engine 2 — Combination

The Combination Engine answers a different question:

**Which independently useful services become more valuable when sold together?**

The naive design would ask an LLM to inspect every possible pair in the catalog.

That was rejected because pairwise evaluation grows quadratically.

A 200-service catalog contains roughly 19,900 unordered pairs. Spending an AI call on every pair is exactly the wrong scaling behavior for a system expected to grow.

Marketplace therefore uses a two-phase design.

## Phase 1 — deterministic candidate generation

Three cheap rules generate plausible bundle candidates:

1. **`pipeline_chain`** — output from service A plausibly feeds service B.
2. **`shared_primitive_upsell`** — services from different projects share a structural primitive.
3. **`audience_bundle`** — services address different stages of one customer journey.

## Phase 2 — AI evaluation only for survivors

Only candidate bundles that survive rule filtering are passed to an LLM for genuine synergy judgment, naming, and justification.

```text
all possible pairs
      │
      ▼
cheap deterministic filters
      │
      ▼
small candidate set
      │
      ▼
LLM synergy evaluation
      │
      ▼
review queue / bundle catalog
```

The important engineering point is not simply “uses rules and AI.”

It is that **expensive intelligence is placed after cheap structure**, so cost scales with plausible opportunities rather than combinatorial possibility.

---

## The 15-pattern structural library

The pattern library is one of the strongest assets in the project because it treats domain names as surface detail and recurring logic as the reusable unit.

Two services can look completely unrelated to a user while sharing the same engineering structure.

Examples from the source project include:

| Structural pattern | Generic function | Example domains |
|---|---|---|
| `strict_ai_evaluator` | Rubric-based scoring + report | recitation evaluation, mock immigration interview |
| `url_to_structured_data` | URL → normalized analysis | product analysis, idea discovery |
| `formal_text_generator` | Context-grounded formal generation | marketing copy, application letters |
| `semantic_matcher` | Meaning-based candidate matching | immigration matching, SaaS matching |
| `deterministic_scoring_engine` | Fixed weighted scoring | company quality, idea scoring |
| `domain_health_checker` | Technical site/domain verification | web-presence checks |
| `geo_scraping_pipeline` | Geography-partitioned discovery | regional prospecting |
| `outreach_automation` | Queue + scheduled follow-up | B2B workflows |
| `intent_extraction` | Free text → structured criteria | RFQ / travel / search flows |
| `comparison_report_generator` | Candidate comparison + rationale | decision-support systems |
| `personalized_roi_estimator` | Case-specific return estimate | commercial qualification |
| `policy_grounded_responder` | Response constrained by policy/KB | review and email workflows |
| `meta_agent_provisioning` | Create another agent on demand | agent factories |
| `form_to_generated_website` | Form → deployable site | digital identity systems |
| `directory_listing_setup` | External presence setup | business directories |

The engineering thesis is:

> **Do not confuse a new business label with new underlying logic.**

Pattern matching exists to reduce duplicate implementation and make reuse visible across project boundaries.

The limitation is equally important: embedding similarity is not proof of structural identity. A genuinely new service must be allowed to become “pattern 16” instead of being forced into a near-but-wrong match.

---

## Pricing architecture

Marketplace deliberately avoids manual per-service pricing as the default path.

The initial pricing rule is derived from `ai_layer`:

| AI layer | Credits | Approx. source-project USD mapping | Intended rationale |
|---|---:|---:|---|
| `none` | 1 | ~$0.25 | near-zero model cost |
| `deterministic` | 5 | ~$1.25 | rules/data execution |
| `generative` | 20 | ~$5.00 | model-backed generation |

Bundles are described in the source project as component sum minus **12.5%**, configurable at the bundle level.

This architecture solves a real operational problem: a growing catalog cannot depend on one person manually pricing every atomic capability.

But the portfolio does **not** overclaim the pricing model.

The source material explicitly acknowledges that `ai_layer` has not yet been validated across enough real conversion/refund data to prove that it predicts customer willingness to pay or actual total delivery cost.

So the correct evidence statement is:

**Pricing automation is IMPLEMENTED. Pricing-model validity at market scale is NOT YET VALIDATED.**

That distinction matters.

---

## Transaction and delivery trust

An atomic-service marketplace has a harsher trust problem than a familiar single-product SaaS. Customers may be buying many small services they have never used before; a failed first transaction can destroy confidence in the entire catalog.

Two design choices address that directly.

### Atomic credit deduction

Balance validation and deduction are designed as one transaction so the system does not intentionally create an intermediate state where an order is registered without payment or payment occurs without the corresponding order state.

The Airtable technical case study also records concurrent-order verification as a targeted failure scenario rather than assuming transactional intent alone was sufficient evidence.

### Honest SLA types

Services are assigned delivery expectations such as:

```text
instant    ≤ 15 minutes
same_day   ≤ 8 hours
custom     service-specific
```

A periodic Celery beat worker checks open orders and surfaces approaching breaches before the promised window expires.

The design principle is simple:

> **A truthful slower promise is stronger engineering than a fast promise the system cannot reliably keep.**

---

## Reliability boundaries

The source project documents several defensive behaviors:

- asynchronous extraction for long documents;
- Pydantic validation around AI-produced structures;
- OCR graceful degradation when Tesseract is unavailable;
- atomic credit handling;
- SLA monitoring;
- embeddings / pgvector for pattern matching and service similarity;
- human review before publication;
- explicit lifecycle state rather than instant AI-to-storefront publishing.

These are not decorative enterprise features. They are the boundaries that keep an AI extraction pipeline from silently turning a plausible model response into a commercial fact.

---

## 55-route API surface — with attribution

The original project documents **55 API routes**, but the number needs correct attribution:

- **23 marketplace-specific routes**;
- **32 inherited platform routes** for auth, billing, credits, and admin capabilities.

The portfolio therefore does not present “55 routes” as 55 unique Marketplace domain endpoints.

Representative Marketplace-specific surfaces include:

```text
/catalog/services
/orders
/admin/marketplace/extract/*
/admin/marketplace/combine/*
/admin/marketplace/services/*
/admin/marketplace/orders/*
```

See the source project for the detailed route inventory; this public evidence repository focuses on architecture and engineering decisions rather than reproducing private implementation.

---

## Tech stack

| Layer | Technology |
|---|---|
| Frontend | Next.js 15 · TypeScript · Tailwind CSS · Recharts · AR/EN + RTL |
| Backend | FastAPI · Python 3.12 · Pydantic v2 |
| Jobs | Celery workers + beat |
| Data | PostgreSQL 16 + pgvector |
| AI | OpenRouter for extraction/combination · OpenAI embeddings |
| OCR | Tesseract (`ara` + `eng`) · pdf2image · pdfplumber |
| Commerce foundation | credits + payment-provider infrastructure inherited from shared SaaS foundation |
| Infra | Docker Compose · CI/CD / recovery capabilities documented in the private project record |

---

## What I built vs. what I reused

The Marketplace-specific engineering includes:

- atomic-service extraction;
- three-axis classification;
- document parsing/chunking/OCR orchestration;
- structural-pattern library and matching;
- two-phase combination engine;
- service authoring/review lifecycle;
- pricing rules tied to classification;
- bundle logic;
- order/SLA domain behavior;
- marketplace-specific APIs and UI flows.

The project was built on a shared SaaS foundation that supplies platform-level concerns such as authentication, billing/credits infrastructure, admin foundations, and recovery capabilities.

The public portfolio intentionally keeps those boundaries visible instead of presenting inherited infrastructure as if it were unique Marketplace engineering.

---

## The larger architectural pattern

The generalizable part of Marketplace is not the specific 1/5/20 credit table or the 15 software patterns.

It is this pipeline:

```text
EXISTING INVENTORY
      ↓
EXTRACT atomic units
      ↓
CLASSIFY on domain-relevant axes
      ↓
MATCH against reusable structures
      ↓
REVIEW / govern
      ↓
PACKAGE alone or in combinations
      ↓
DELIVER with explicit economics + SLA
```

Airtable research developed for the project tested this architecture against adjacent domains rather than merely claiming universality. Strong structural parallels were found in API marketplaces, internal developer platforms, no-code automation template marketplaces, and university technology-transfer workflows. Fiverr-style gig marketplaces were explicitly rejected as a full structural match because they share packaging but lack automated extraction and shared cross-inventory structural learning.

That research supports **architectural analogy**, not deployment validation in those domains.

The right claim is:

**The pipeline appears structurally reusable. Cross-domain implementations remain unvalidated until actually built and tested.**

---

## Known limitations

Marketplace is intentionally documented with its weak points visible:

- `ai_layer` is an implemented pricing heuristic, not a proven willingness-to-pay model.
- Pydantic validates structure, not semantic truth.
- Pattern matching can over-match or under-match; current review remains an important safeguard.
- Market Intelligence is scaffolded, not shipped.
- Continuous Development is scaffolded, not shipped.
- Custom-request flow is scaffolded, not shipped.
- “Pattern reuse” does not mean domain behavior is interchangeable without domain-specific validation.
- The 55-route figure includes inherited platform routes and must be attributed accordingly.
- Current catalog/order volume is not presented as statistically sufficient to validate pricing, conversion, or pattern-threshold performance.

These limitations do not weaken the case study. They define the actual engineering boundary.

---

## Evidence map

- [`docs/CASE_STUDY.md`](docs/CASE_STUDY.md) — problem, design goals, trade-offs, and engineering story.
- [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) — engines, shared knowledge base, data/control flow.
- [`docs/ENGINEERING_DECISIONS.md`](docs/ENGINEERING_DECISIONS.md) — decision records and rejected alternatives.
- [`docs/TESTING_AND_VERIFICATION.md`](docs/TESTING_AND_VERIFICATION.md) — what is implemented, what is tested, and evidence boundaries.
- [`docs/SECURITY_AND_RELIABILITY.md`](docs/SECURITY_AND_RELIABILITY.md) — transaction, review, SLA, validation, and degradation boundaries.
- [`docs/LIMITATIONS.md`](docs/LIMITATIONS.md) — known technical and evidence constraints.
- [`evidence/README.md`](evidence/README.md) — public evidence index and evidence status conventions.
- [`PORTFOLIO_NOTICE.md`](PORTFOLIO_NOTICE.md) — source-code and review scope.

---

## Review path

**AgentCraft → Marketplace → Engineering Thesis → Architecture → Decisions → Evidence → Limitations → Live Product**

---

**Ayman Alsaid** · Senior AI / Product Engineer  
AgentCraft · https://agentcraft.info · contact@agentcraft.info
