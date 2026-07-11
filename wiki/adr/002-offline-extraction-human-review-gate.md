# ADR-002: Offline Extraction with Human Review Gate

| Field | Value |
|-------|-------|
| **ADR ID** | ADR-002 |
| **Status** | Accepted |
| **Date** | 2026-07-10 |
| **Deciders** | Principal Architect, AI Platform Lead, Product |
| **Depends on** | ADR-001 |

---

## Context

Exam intelligence (PYQ patterns, traps, memory anchors, MCQs, smart highlights) is **high value** but **not present in NCERT text verbatim**. Pure rules cannot author "How BPSC tests this." LLMs can draft this content quickly but introduce factual errors.

Students must never see unreviewed LLM output in the Reader or Practice flows.

---

## Decision

1. **All LLM-generated intelligence runs offline** as batch jobs after canonical load.
2. Outputs write to **`intelligence_staging.*` only** — never directly to student-facing tables.
3. **Content Ops must approve** each artifact class before publish.
4. A **Publish Worker** promotes approved rows to `intelligence.*` under a versioned `intelligence_pack`.
5. Student APIs read **`status = published`** rows only.

---

## Rationale (WHY)

| Principle | How this ADR enforces it |
|-----------|--------------------------|
| Deterministic truth first (ADR-001) | Canonical text untouched by LLM |
| Exam intelligence second | LLM accelerates authoring, humans certify |
| Rollback | Pack version decrement reverts intelligence without redeploy |
| Audit | `extraction_runs` + reviewer_id on each row |

---

## Consequences

### Positive
- Credible "How BPSC tests it" rail
- Clear squad boundary: AI proposes, Content Ops commits
- Tutor (future) grounds on published graph only

### Negative
- Review latency bottleneck at scale
- Staging schema + Review UI engineering cost
- Not fully real-time "AI magic" at launch

---

## Alternatives considered

| Option | Why rejected |
|--------|--------------|
| Real-time LLM in Reader | Latency + hallucination per page view |
| Full manual authoring | Too slow for 3-subject catalog |
| Auto-publish all LLM output | Unacceptable trust risk for exam prep |

---

## Implementation notes

- **Docs:** [WIKI-09](../09-offline-ai-knowledge-extraction.md), [WIKI-06](../06-knowledge-graph-and-question-intelligence.md)
- **Phase 1 shortcut:** Spreadsheet import to `intelligence.*` while worker built
- **Rollback:** Repoint active `intelligence_pack` version

---

## References

- [WIKI-09 Offline AI Knowledge Extraction](../09-offline-ai-knowledge-extraction.md)
- [WIKI-06 Knowledge Graph & Question Intelligence](../06-knowledge-graph-and-question-intelligence.md)
