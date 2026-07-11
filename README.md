# SarkariExamsAI — Engineering Documentation

**Single entry point.** Pick your team folder — do not read everything.

```
docs/
├── business/     → Product, vision, metrics (PM, leadership, investors)
├── ai/           → AI Architecture Specification (AI team contract)
├── backend/      → Pipeline, database, APIs, ops (Backend, Data, Content Platform)
├── frontend/     → PWA, Reader UI, Redux (Frontend squad)
└── wiki/         → Legacy index (redirects here)
```

---

## Who reads what?

| Role | Start here | Then |
|------|------------|------|
| **AI Team** | [`ai/00-ai-architecture-v1.md`](./ai/00-ai-architecture-v1.md) | `ai/01`–`04` workers, `ai/05`–`08` |
| **Backend Engineer** | [`backend/README.md`](./backend/README.md) | Ingestion → DB → Student APIs |
| **Frontend Engineer** | [`frontend/README.md`](./frontend/README.md) | Reader UI, state, API contracts |
| **Product / Business** | [`business/README.md`](./business/README.md) | Vision, metrics, roadmap |
| **New engineer (any)** | [`business/01-platform-vision.md`](./business/01-platform-vision.md) → [`backend/01-system-architecture.md`](./backend/01-system-architecture.md) | Your squad folder |

---

## Platform in one diagram

```
CONTENT SOURCES (NCERT, PYQs, Notes…)
        ↓
DETERMINISTIC PIPELINE (no AI) → PostgreSQL Canonical Store
        ↓
OFFLINE AI PIPELINE (4 Workers) → Validation → Review → Knowledge Graph
        ↓
QUESTION INTELLIGENCE PIPELINE (separate) → Question Graph
        ↓
INTELLIGENCE ENGINE (merge graphs) → Exam Intelligence
        ↓
STUDENT APIs (SQL only, no LLM)
        ↓
READER UI (SarkariExamsAI PWA)
        ↓
ONLINE AI TUTOR (future — RAG only)
```

---

## Repositories

| Repo | Docs folder | Production |
|------|-------------|------------|
| `knowledge-compiler/` | `docs/backend/` | API TBD / Supabase |
| `sarkariexamsAI/` | `docs/frontend/` | https://guileless-crepe-c5261c.netlify.app |
| Cross-cutting AI | `docs/ai/` | Offline batch jobs |
| Product | `docs/business/` | — |

---

## Core principles

1. **Not an AI chatbot** — Knowledge Intelligence Platform for government exams.
2. **Deterministic truth first** — No LLM in PDF ingestion.
3. **AI team receives Architecture Spec, not prompts** — See `docs/ai/`.
4. **Four domain workers, not nine micro-agents** — Easier to build, test, monitor.
5. **No AI output enters DB without validation + human review.**

---

## Document status (2026-07-11)

| Area | Status |
|------|--------|
| Deterministic pipeline | ✅ Implemented |
| Canonical DB | ✅ Implemented |
| Student `/api/courses` | ✅ Partial |
| Reader UI | ✅ Deployed (mock default) |
| AI Architecture v1 | 📋 Spec ready (`docs/ai/`) |
| Offline AI workers | ⬜ Not built |
| Question Intelligence | ⬜ Not built |
| Online Tutor | ⬜ Future |
