# AI Team Documentation

**All AI-related documentation lives in this folder only.**

```
docs/ai/
├── AI-PLATFORM-GUIDE.md    ← START HERE (single spec)
├── README.md               ← this file
└── adr/                    ← architecture decision records
    ├── 001-deterministic-ingestion-first.md
    └── 002-offline-extraction-human-review-gate.md
```

---

## [AI Platform Guide](./AI-PLATFORM-GUIDE.md)

The complete specification (v1.2 — **BPSC Prelims + Mains dual-stage**):

- **Part 1–2** — Platform layers (high level + input/output per layer)
- **Part 3** — AI worker I/O master table
- **Part 4–8** — Validation, data models, ownership, examples

Share **`AI-PLATFORM-GUIDE.md`** with the AI team.

---

## Architecture decisions

| ADR | Title |
|-----|-------|
| [001](./adr/001-deterministic-ingestion-first.md) | No AI in PDF ingestion |
| [002](./adr/002-offline-extraction-human-review-gate.md) | Human review before publish |

---

## Not in this folder (by design)

| Topic | Location |
|-------|----------|
| PDF pipeline (no AI) | [`../backend/02-ingestion-pipeline.md`](../backend/02-ingestion-pipeline.md) |
| Canonical database | [`../backend/03-canonical-database.md`](../backend/03-canonical-database.md) |
| Student APIs (no LLM) | [`../backend/04-student-apis.md`](../backend/04-student-apis.md) |
| Reader UI | [`../frontend/01-reader-ui.md`](../frontend/01-reader-ui.md) |
