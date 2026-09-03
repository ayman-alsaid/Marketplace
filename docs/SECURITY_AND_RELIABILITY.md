# Marketplace — Security and Reliability Boundaries

Marketplace combines AI extraction with commerce. That makes correctness boundaries around publication, payment, and delivery more important than raw generation speed.

## 1. Human publication boundary

AI-extracted services do not become published products automatically.

```text
AI extraction
   ↓
Pydantic validation
   ↓
`suggested` lifecycle state
   ↓
human review
   ↓
publish / reject
```

This boundary exists because schema validity cannot prove semantic correctness.

## 2. Transaction boundary

Orders and credit deductions are designed to avoid partial commercial state. The project record describes atomic balance validation/deduction and targeted concurrency verification around competing orders.

## 3. SLA boundary

Every service carries a delivery type rather than inheriting a global marketing promise. A periodic worker checks open orders and surfaces approaching/breached SLA state.

The product principle is reliability through truthful promises, not optimistic speed claims.

## 4. OCR degradation

Scanned-document extraction can use Tesseract with Arabic and English language data. If OCR is unavailable, the pipeline is designed to preserve useful direct-extraction output rather than fail the whole ingestion path when possible.

## 5. AI-output validation

Pydantic v2 provides structural validation and normalization around generated extraction output.

This reduces malformed-data failure but does not eliminate plausible semantic errors. The review queue remains the stronger control for those cases.

## 6. Cost boundary in Combination

The rules-first combination phase is also a reliability feature: the LLM is not asked to manufacture bundle relevance across every possible pair. Deterministic filters establish a plausible candidate set first.

## 7. Private-source boundary

The production source remains private. This evidence repository intentionally does not publish secrets, credentials, private implementation details, or legacy private-repository references.

## 8. Inherited-platform attribution

Authentication, billing/credits foundations, and recovery capabilities come in part from the shared SaaS foundation. They should be re-verified project-specifically before being elevated into Marketplace-specific security claims beyond what the project record supports.
