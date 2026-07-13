# Backend & Data Platform Documentation

**Audience:** Backend engineers, Data Platform, Content Platform, DevOps

**Repo:** `knowledge-compiler/`

---

## Documents

| # | Document | Topic | Status |
|---|----------|-------|--------|
| 01 | [System Architecture](./01-system-architecture.md) | Layers, components, deployment | Draft |
| 02 | [Ingestion Pipeline](./02-ingestion-pipeline.md) | Steps 1–10, deterministic, **NO AI** | ✅ Implemented |
| 03 | [Canonical Database](./03-canonical-database.md) | PostgreSQL schema, tables, migrations | ✅ Implemented |
| 04 | [Student APIs](./04-student-apis.md) | `/api/courses`, contracts, TypeScript mirror | Partial |
| 05 | [Naming & Governance](./05-naming-governance.md) | IDs, validation rules, versioning | Draft |
| 06 | [Ops & Testing](./06-ops-testing.md) | CI/CD, squads, on-call, test pyramid | Draft |

---

## Backend responsibility boundary

```
✅ PDF → Pipeline → Canonical DB
✅ Student APIs (SQL reads, no LLM)
✅ Admin APIs (pipeline, canonical load)
✅ Alembic migrations (public schema)
✅ File store (uploads, pipeline_output/)

❌ AI extraction workers → see docs/ai/
❌ Prompt engineering → AI team internal
❌ Reader UI → docs/frontend/
```

---

## Data flow (backend only)

```
PDF upload
    → Pipeline Steps 1–10
    → step10_canonical.json
    → POST /api/books/{id}/canonical/load
    → PostgreSQL (public.*)
    → GET /api/courses/* (Student APIs)
```

**AI pipeline is downstream** — reads `paragraphs`, writes `intelligence_staging.*` (see [`../ai/`](../ai/)).

---

## Key paths in repo

```
knowledge-compiler/
├── backend/
│   ├── main.py                 # FastAPI entry
│   ├── pipeline/step01–10.py   # Deterministic ingestion
│   ├── db/models.py            # Canonical ORM
│   ├── routers/courses.py      # Student APIs
│   └── services/courses.py     # Screen-oriented shaping
├── alembic/versions/           # Migrations
├── exports/supabase/           # SQL export/import
└── pipeline_output/{book_id}/  # Step artifacts
```

---

## API quick reference

| Endpoint | Purpose |
|----------|---------|
| `GET /api/health` | Health check |
| `GET /api/courses` | Course list |
| `GET /api/courses/{book}/topics/{id}/intro` | Topic intro |
| `GET /api/courses/.../steps` | Reading steps |
| `POST /api/books/{id}/canonical/load` | Load canonical JSON |
| Swagger | `http://localhost:8000/docs` |

---

## Environment

| Variable | Purpose |
|----------|---------|
| `DATABASE_URL` | PostgreSQL connection |
| `OPENAI_API_KEY` | Legacy path only — do not use for ingestion |

---

## Related

- **AI contract:** [`../ai/AI-PLATFORM-GUIDE.md`](../ai/AI-PLATFORM-GUIDE.md)
- **ADRs:** [`../ai/adr/`](../ai/adr/)
- **Expo mobile API requirements:** [`../mobile/02-mobile-backend-contract.md`](../mobile/02-mobile-backend-contract.md)
