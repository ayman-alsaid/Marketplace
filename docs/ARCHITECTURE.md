# Marketplace — Architecture

## System topology

Marketplace is built as a shared knowledge platform rather than a set of disconnected utilities.

```text
Next.js 15 frontend
      │
      ▼
FastAPI backend
      │
      ├──────────────► PostgreSQL 16 + pgvector
      │                      │
      │                      ├─ services
      │                      ├─ structural patterns
      │                      ├─ bundles
      │                      ├─ orders
      │                      ├─ extraction projects
      │                      └─ feedback / future intelligence state
      │
      ├──────────────► Celery workers + beat
      │                      │
      │                      ├─ extraction tasks
      │                      ├─ bundle generation
      │                      └─ SLA monitoring
      │
      ├──────────────► OpenRouter
      ├──────────────► OpenAI embeddings
      └──────────────► Tesseract / file parsing
```

## Four-engine model

### 1. Extraction — shipped

```text
PDF / DOCX / TXT / raw text
        │
        ├─ direct parsing
        ├─ OCR fallback for scanned Arabic/English documents
        └─ long-document chunking
        │
        ▼
LLM extraction
        │
        ▼
Pydantic validation
        │
        ▼
3-axis classification
 service_type
 ai_layer
 structural_pattern
        │
        ▼
embedding / pattern match
        │
        ▼
suggested service → human review
```

Long extraction tasks execute asynchronously and merge/deduplicate chunk results after processing.

### 2. Combination — shipped

```text
published services
        │
        ▼
deterministic candidate filters
  pipeline_chain
  shared_primitive_upsell
  audience_bundle
        │
        ▼
small candidate set
        │
        ▼
LLM synergy evaluation
        │
        ▼
review / publish bundle
```

The cost boundary is deliberate: AI operates after structural filtering, not over the complete pair space.

### 3. Market Intelligence — scaffolded

The designed model distinguishes at least:

- high demand with active competition;
- genuine rarity supported by both scarcity and complaints;
- scarcity without demand, treated as potentially dead market.

The source marks external scraping/integration as future work. This repository therefore documents the architecture but does not claim live market-intelligence results.

### 4. Continuous Development — scaffolded

The designed feedback loop captures rejection reasons, refunds, and support signals and maps them to topics such as slow delivery, low accuracy, high price, or feature request.

The intended outputs include upgrade, fix, reprice, extend, or retire recommendations. The source marks this as scaffolded, not a shipped autonomous improvement loop.

## Data/control boundaries

### AI proposal boundary

AI output is not publication authority.

```text
LLM output → schema validation → suggested lifecycle → human review → published
```

### Pricing boundary

Pricing is derived from `ai_layer`, but the rule is explicitly treated as a heuristic whose market validity remains unproven at scale.

### Pattern boundary

Embeddings provide similarity evidence; they do not prove that two capabilities are structurally identical.

### Commerce boundary

Order registration and credit deduction are treated transactionally. Delivery expectations are stored per service instead of globally promised.

## Failure/degradation paths

- malformed model structures → Pydantic validation/normalization;
- OCR unavailable → keep direct extraction rather than fail the entire document path;
- long extraction → async background processing;
- suspicious semantic classification → human review before publication;
- potential SLA breach → periodic worker surfaces it before deadline when possible;
- no plausible bundle candidate → avoid spending an LLM call merely to force a bundle.

## Architecture principle

Marketplace repeatedly uses the same pattern:

**deterministic structure first, probabilistic intelligence second, human authority where semantic or commercial consequences require it.**
