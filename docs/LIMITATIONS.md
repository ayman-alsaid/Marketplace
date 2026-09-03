# Marketplace — Known Limitations

This project is intentionally documented with its unresolved assumptions visible.

## Pricing is implemented; pricing validity is not proven

The `ai_layer` → credit-tier mapping is a production mechanism. It is not presented as a statistically validated model of willingness to pay or total execution cost.

A deterministic service can still depend on expensive external data. A generative service can sometimes be a cheap model call. Customer value may not correlate cleanly with model usage.

## Semantic classification can be confidently wrong

Pydantic catches malformed structure, not incorrect meaning. An extraction can produce a perfectly valid service object with the wrong `ai_layer`, wrong service boundary, or wrong structural pattern.

Human review is therefore a current safeguard, not merely administrative overhead.

## Pattern matching has threshold risk

Embeddings can:

- over-match a genuinely novel capability into an existing pattern;
- under-match duplicate logic as if it were novel.

No measured false-match rate is supplied in the current evidence package.

## Combination filters trade recall for cost

Rules-first candidate generation avoids quadratic LLM spend, but a valuable bundle that satisfies none of the three deterministic candidate rules may never reach AI evaluation.

## Market Intelligence is scaffolded

The data model and intended signal taxonomy exist. The external scraper integration is not claimed as shipped, and no demand-detection results are presented as production evidence.

## Continuous Development is scaffolded

Feedback structures exist for rejections, refunds, and support topics. A production loop that autonomously recommends and verifies upgrades/repricing/retirement is not claimed as complete.

## Custom request flow is scaffolded

The free-text → intent → instant bundle quote concept is not represented as a finished customer path.

## Structural reuse is not domain interchangeability

The pipeline can appear reusable across API catalogs, no-code workflows, internal developer platforms, or technology-transfer contexts. That does not mean Marketplace's specific classification axes, pricing rules, or 15 software patterns transfer unchanged.

## Route counts require attribution

The source project reports 55 total application routes, consisting of 23 marketplace-specific routes and 32 inherited platform routes. The full number should not be used to imply 55 unique Marketplace domain endpoints.

## Evidence artifacts are not all public yet

The private project record and Airtable case study describe implementation and targeted verification scenarios. Sanitized raw run transcripts, screenshots, and test logs are not yet present for every claim in this public repository.
