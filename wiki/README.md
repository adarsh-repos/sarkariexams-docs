# Wiki (Legacy Index)

> **Documentation has been reorganized.** Use [`../README.md`](../README.md) as the entry point.

---

## New folder structure

| Folder | Audience |
|--------|----------|
| [`../business/`](../business/) | Product, PM, leadership |
| [`../ai/`](../ai/) | **AI Team — Architecture Specification (contract)** |
| [`../backend/`](../backend/) | Backend, Data, Content Platform |
| [`../frontend/`](../frontend/) | Frontend, UX |

---

## What moved where

| Old wiki doc | New location |
|--------------|--------------|
| 00 Platform Vision | `business/01-platform-vision.md` |
| 01 System Architecture | `backend/01-system-architecture.md` |
| 02 Ingestion Pipeline | `backend/02-ingestion-pipeline.md` |
| 03 Canonical DB | `backend/03-canonical-database.md` |
| 04 Student APIs | `backend/04-student-apis.md` |
| 05 Reader UI | `frontend/01-reader-ui.md` |
| 06–10 AI docs | Superseded by **`ai/`** folder (start at `ai/00-ai-architecture-v1.md`) |
| 07 Naming | `backend/05-naming-governance.md` |
| 08 Ops | `backend/06-ops-testing.md` |
| ADRs | `adr/` (still here) |

---

## AI team: start here

**[`../ai/00-ai-architecture-v1.md`](../ai/00-ai-architecture-v1.md)**

You receive an **AI Architecture Specification** — not prompts.

---

## ADRs

| ADR | Title |
|-----|-------|
| [001](./adr/001-deterministic-ingestion-first.md) | Deterministic Ingestion First |
| [002](./adr/002-offline-extraction-human-review-gate.md) | Offline Extraction + Human Review |
