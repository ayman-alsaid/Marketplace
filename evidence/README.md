# Marketplace — Evidence Index

This directory is the public index for claim-to-evidence artifacts that can be shared without exposing production source, credentials, or proprietary internals.

## Current evidence categories

### Implemented / deployed behavior documented by the primary project record

- 13 marketplace-specific domain tables.
- 23 marketplace-specific routes inside a 55-route application surface.
- Extraction from text/PDF/DOCX/TXT.
- Arabic OCR fallback.
- Three-axis service classification.
- 15-pattern structural library.
- Two-phase bundle discovery.
- Credit-based pricing rules.
- Catalog/order lifecycle and SLA monitoring.

### Targeted verification described by the technical case study

- concurrent credit/order scenario;
- OCR graceful-degradation scenario.

Raw sanitized transcripts are not yet included here, so these are cited as source-supported verification claims rather than pretending public artifacts already exist.

## Scaffolded — not promoted to implemented

- Market Intelligence.
- Continuous Development.
- Custom Request flow.

## Recommended future public artifacts

1. Sanitized Extraction run: input excerpt → candidate services → classifications.
2. Arabic scanned-PDF OCR fallback example.
3. Pattern match example with similarity evidence and human review decision.
4. Combination run showing deterministic rule reason and surviving LLM judgment.
5. Concurrent order test output.
6. SLA state-transition example.
7. Route inventory generated from the live application.

## Evidence rule

Every significant public claim should be expressible as:

```text
Claim → Status → Evidence → Scope → Limitation
```

The strongest claim supported by the evidence is preferred over the strongest claim that would make the project sound impressive.
