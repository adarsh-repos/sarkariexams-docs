# Online AI Tutor (Future)

| Document | `AI-09` |
|----------|---------|
| **Status** | Out of scope for AI v1 |

---

## Rule

**This is the ONLY place LLM runs in production** (student-facing, real-time).

---

## Architecture

```
Student question
      ↓
Query classifier (intent: doubt / practice help / navigation)
      ↓
Retriever
   ├── Vector search on paragraphs (canonical)
   ├── Concept lookup (KG)
   ├── Fact retrieval (KG)
   └── PYQ patterns (QG)
      ↓
Context packer (max tokens, cite sources)
      ↓
LLM (grounded generation only)
      ↓
Response + citations [paragraph_id, concept_id, fact_id]
      ↓
Optional: safety filter
```

---

## Constraints

| Rule | Why |
|------|-----|
| Must cite `paragraph_id` or `fact_id` | Traceability |
| Cannot contradict published facts | Trust |
| Cannot answer if retrieval empty | "Not in your syllabus yet" |
| No web search in v1 | Exam syllabus bounded |
| Log all Q&A for audit | Quality loop |

---

## Not in v1

Build only after:
1. Knowledge Graph published for ≥1 full book
2. Question Graph with ≥500 PYQs
3. Student APIs stable
4. Validation + review pipeline proven

---

## AI team v1 focus

**Offline 4 workers only.** Online tutor is Phase 2 (separate spec).
