# ADR-001: Deterministic Ingestion First

| Field | Value |
|-------|-------|
| **ADR ID** | ADR-001 |
| **Status** | Accepted |
| **Date** | 2026-07-10 |
| **Deciders** | Principal Architect, Content Platform Lead |
| **Supersedes** | Ad-hoc LLM PDF extraction (`compiler.py`) |

---

## Context

Government exam preparation requires **factual accuracy**. Early prototypes used LLM-based PDF parsing (`backend/services/compiler.py`, `chapter_schema.json`) which produced useful drafts but exhibited:

- Non-reproducible output across runs
- Occasional fabricated dates and names
- Unstable IDs unsuitable for progress tracking and MCQ linkage

Students trust SarkariExamsAI as a **knowledge authority**, not a chatbot. Canonical reading material cannot carry hallucination risk.

---

## Decision

**All book content enters the platform through a 10-step deterministic ingestion pipeline** (`backend/pipeline/step01–10`) with **zero LLM calls**. AI may only run **after** canonical load, in offline staging, with human review (see ADR-002).

Step 9 validation **must pass** before `step10_canonical.json` is loaded to PostgreSQL.

---

## Rationale (WHY)

| Factor | Deterministic wins |
|--------|-------------------|
| Trust | Same PDF + pipeline version → same output |
| Audit | Content Ops can diff artifacts between runs |
| IDs | Stable `section_id`, `paragraph_id` for graph + progress |
| Cost | No token spend on ingestion |
| Legal | Source text traceable to PDF page |

---

## Consequences

### Positive
- Production-ready pipeline for NCERT History Class 10
- Supabase export/import reproducible
- Student APIs query trustworthy paragraphs

### Negative
- Pipeline engineering upfront cost
- Non-standard PDF layouts require pipeline code changes, not prompt tweaks
- Legacy AI path sunk cost

### Neutral
- LLM still used later for enrichment (WIKI-09)

---

## Alternatives considered

| Option | Why rejected |
|--------|--------------|
| LLM-only ingestion | Hallucination risk unacceptable for canonical text |
| Hybrid (LLM layout + rules) | Hard to audit; kept LLM out entirely |
| Manual HTML entry | Does not scale to full NCERT catalog |

---

## Implementation notes

- **Repos:** `knowledge-compiler/backend/pipeline/`
- **Docs:** [WIKI-02](../02-deterministic-ingestion-pipeline.md)
- **Legacy:** Mark `compiler.py` deprecated; do not delete until extraction path validated
- **Rollback:** N/A — foundational decision

---

## References

- [WIKI-00 Platform Vision](../00-platform-vision-and-product-strategy.md)
- [WIKI-02 Deterministic Ingestion Pipeline](../02-deterministic-ingestion-pipeline.md)
- `knowledge-compiler/backend/pipeline/step09_validation_engine.py`
