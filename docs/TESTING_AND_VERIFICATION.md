# Marketplace — Testing and Verification

This repository separates what the source project states is implemented from what is directly verified by the supplied evidence package.

## Evidence matrix

| Claim | Classification | Evidence boundary |
|---|---|---|
| 13 marketplace-specific tables exist | **IMPLEMENTED** | Documented in the primary project README and architecture inventory. |
| 23 marketplace-specific routes exist | **IMPLEMENTED** | Source project attributes 23 domain routes plus 32 inherited platform routes. |
| 55 total routes exposed by the application | **IMPLEMENTED** | Aggregate application count; must not be misread as 55 unique Marketplace domain routes. |
| Extraction supports text/PDF/DOCX/TXT ingestion | **IMPLEMENTED** | Documented source behavior. |
| Arabic OCR fallback exists | **IMPLEMENTED** | Documented Tesseract `ara`/`eng` path. |
| Long-document extraction runs async and merges/deduplicates chunk results | **IMPLEMENTED** | Documented extraction architecture. |
| Three-axis classification drives service metadata | **IMPLEMENTED** | `service_type`, `ai_layer`, `structural_pattern`. |
| 15 structural patterns are seeded | **IMPLEMENTED** | Documented seed library. |
| Combination uses three deterministic candidate rules before LLM evaluation | **IMPLEMENTED** | Documented two-phase engine. |
| Atomic credit handling was tested against concurrent-order behavior | **TESTED** | Described in the Airtable technical case study; this public repo does not include raw test transcript yet. |
| OCR graceful degradation was tested with Tesseract unavailable | **TESTED** | Described in Airtable technical case study; raw artifact is not public here yet. |
| Pricing by `ai_layer` predicts willingness to pay | **NOT YET VALIDATED** | Source explicitly says insufficient real conversion/refund volume. |
| Pattern-match threshold accuracy is measured | **NOT YET VALIDATED** | No reliable over-match/under-match rate supplied. |
| Market Intelligence is live | **SCAFFOLDED** | Tables/design exist; scraper integration is next phase. |
| Continuous Development is live | **SCAFFOLDED** | Feedback model exists; full autonomous loop is not claimed. |

## Important verification distinction

A pipeline can be correct at several different levels:

1. **Structural validity** — the LLM response conforms to the schema.
2. **Execution validity** — the workflow runs without an exception.
3. **Semantic correctness** — the service was classified correctly.
4. **Commercial validity** — the assigned price and packaging perform well in the market.

Marketplace currently has stronger evidence for levels 1–2 than for levels 3–4 at scale.

That is why human review remains part of the production boundary.

## Concurrent credit scenario

The technical case study records explicit concern for two near-simultaneous orders reading the same available balance. The intended invariant is:

```text
balance check + deduction + order state
```

must not permit both requests to independently spend the same credits.

The project content states this failure mode was tested with concurrent requests. Public raw traces can be added later if sanitized evidence becomes available.

## OCR degradation scenario

The OCR path is designed so Tesseract failure does not automatically destroy the entire extraction request. The source describes testing with OCR unavailable to verify fallback to direct extraction output where possible.

## What should be added later

When public-safe artifacts are available, add:

- sanitized extraction request/result examples;
- OCR fallback transcript;
- concurrent-credit test output;
- bundle-candidate examples showing rule reason + LLM survivor evaluation;
- pattern-match examples including one accepted match and one human rejection;
- current route inventory generated from the running application.

Until then, this repository states the strongest level supported by the supplied project record and does not invent new benchmark/test-count claims.
