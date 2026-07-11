# AI Team — Documentation Index

**This folder is the contract between Engineering and the AI Team.**

You receive an **AI Architecture Specification** — not prompts, not ad-hoc tasks.

---

## Read in this order

| # | Document | What you learn |
|---|----------|----------------|
| **00** | [AI Architecture v1](./00-ai-architecture-v1.md) | **Start here.** Full system diagram, boundaries, 4 workers |
| **01** | [Worker 1 — Content Intelligence](./01-worker-content-intelligence.md) | Concepts, facts, entities, keywords, learning objectives |
| **02** | [Worker 2 — Relationship Intelligence](./02-worker-relationship-intelligence.md) | Graph edges, timeline, geography, cross-refs |
| **03** | [Worker 3 — Question Intelligence](./03-worker-question-intelligence.md) | PYQ/MCQ parsing, concept mapping, patterns, Bloom |
| **04** | [Worker 4 — Exam Intelligence](./04-worker-exam-intelligence.md) | Revision priority, PYQ frequency, topic weightage |
| **05** | [Data Models & Storage](./05-data-models-and-storage.md) | Every table, field, ID, relationship |
| **06** | [Validation Layer](./06-validation-layer.md) | 4-level validation — your firewall |
| **07** | [Knowledge Graph & Question Graph](./07-knowledge-graph-and-question-graph.md) | How graphs connect |
| **08** | [Intelligence Engine](./08-intelligence-engine.md) | Merge graphs → per-concept intelligence |
| **09** | [Online AI (Future)](./09-online-ai-future.md) | Only place LLM runs in production |

---

## What AI team owns

```
✅ 4 offline AI workers (batch)
✅ Validation pipeline (L1–L4)
✅ Staging schema + extraction jobs
✅ Model pinning, metrics, golden tests
✅ JSON schemas (structured output contracts)

❌ PDF ingestion pipeline
❌ Canonical database schema
❌ Student API routes
❌ Reader UI
❌ Prompts as deliverable (prompts are implementation detail inside workers)
```

---

## What AI team receives from Engineering

| Input | Source | Format |
|-------|--------|--------|
| Paragraphs | PostgreSQL `paragraphs` | `paragraph_id`, `text`, `section_id` |
| Topic hierarchy | `sections`, `chapters`, `books` | Stable IDs |
| Glossary | `glossary_entries` | term, meaning |
| Questions (PYQ bank) | CSV / DB import | stem, options, answer |
| Exam profile | Config | `BPSC`, `UPSC`, etc. |

---

## What AI team delivers to Engineering

| Output | Consumer |
|--------|----------|
| Staging rows (`intelligence_staging.*`) | Review UI |
| Validation reports (JSON) | Ops dashboard |
| Published intelligence (`intelligence.*`) | Student APIs |
| Knowledge Graph edges | Intelligence Engine |
| Question Graph edges | Practice engine |

---

## Architecture decision

**4 domain-oriented workers** (not 9 independent agents):

1. Content Intelligence Worker  
2. Relationship Intelligence Worker  
3. Question Intelligence Worker  
4. Exam Intelligence Worker  

Rationale: Same pattern used at Google, Microsoft, OpenAI, Anthropic for enterprise AI — easier to develop, test, monitor, version, and scale.

---

## Related (other folders)

- Canonical truth (no AI): [`../backend/02-ingestion-pipeline.md`](../backend/02-ingestion-pipeline.md)
- Student APIs (no LLM at runtime): [`../backend/04-student-apis.md`](../backend/04-student-apis.md)
- Product context: [`../business/01-platform-vision.md`](../business/01-platform-vision.md)
- ADRs: [`../wiki/adr/`](../wiki/adr/)
