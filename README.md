# SarkariExamsAI — Engineering Documentation

**Single entry point.** Pick your team folder — do not read everything.

```
docs/
├── business/     → Product, vision, metrics (PM, leadership, investors)
├── ai/           → **All AI docs** (single guide + ADRs)
├── backend/      → Pipeline, database, APIs, ops (Backend, Data, Content Platform)
├── frontend/     → PWA, Reader UI, Redux (Frontend squad)
├── mobile/       → Expo mobile blueprint + reference-screen inventory
└── wiki/         → Legacy index (redirects here)
```

---

## Who reads what?

| Role | Start here | Then |
|------|------------|------|
| **AI Team** | [`ai/AI-PLATFORM-GUIDE.md`](./ai/AI-PLATFORM-GUIDE.md) | One doc: layers, inputs, outputs, workers |
| **Backend Engineer** | [`backend/README.md`](./backend/README.md) | Ingestion → DB → Student APIs |
| **Frontend Engineer** | [`frontend/README.md`](./frontend/README.md) | Reader UI, state, API contracts |
| **Mobile Engineer / UX** | [`mobile/README.md`](./mobile/README.md) | Expo architecture, mobile API contract, delivery plan |
| **Product / Business** | [`business/README.md`](./business/README.md) | Vision, metrics, roadmap |
| **New engineer (any)** | [`business/01-platform-vision.md`](./business/01-platform-vision.md) → [`backend/01-system-architecture.md`](./backend/01-system-architecture.md) | Your squad folder |

---

## Platform in one diagram

```
CONTENT SOURCES (NCERT, PYQs, Notes…)
        ↓
DETERMINISTIC PIPELINE (no AI) → PostgreSQL Canonical Store
        ↓
AI ORCHESTRATION → OFFLINE AI WORKERS (4) → VALIDATION ENGINE → Review
        ↓
Knowledge Graph + Question Graph → Intelligence Engine
        ↓
STUDENT APIs (SQL only, no LLM)
        ↓
WEB PWA + EXPO MOBILE CLIENT
        ↓
ONLINE AI TUTOR (future — RAG only)
```

---

## Repositories

| Repo | Docs folder | Production |
|------|-------------|------------|
| `knowledge-compiler/` | `docs/backend/` | API TBD / Supabase |
| `sarkariexamsAI/` | `docs/frontend/` | https://guileless-crepe-c5261c.netlify.app |
| Expo mobile (planned) | `docs/mobile/` | Separate client; not scaffolded |
| Cross-cutting AI | `docs/ai/` | Offline batch jobs |
| Product | `docs/business/` | — |

---

## Core principles

1. **Not an AI chatbot** — Knowledge Intelligence Platform for government exams.
2. **Deterministic truth first** — No LLM in PDF ingestion.
3. **AI team receives one spec** — [`ai/AI-PLATFORM-GUIDE.md`](./ai/AI-PLATFORM-GUIDE.md)
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
| Expo mobile blueprint | 📋 Ready — [`mobile/README.md`](./mobile/README.md) |
| AI Platform Guide | 📋 Ready — [`ai/AI-PLATFORM-GUIDE.md`](./ai/AI-PLATFORM-GUIDE.md) |
| Offline AI workers | ⬜ Not built |
| Question Intelligence | ⬜ Not built |
| Online Tutor | ⬜ Future |
