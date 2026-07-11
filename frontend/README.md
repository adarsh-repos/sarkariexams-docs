# Frontend Documentation

**Audience:** Frontend engineers, UX, QA

**Repo:** `sarkariexamsAI/`  
**Production:** https://guileless-crepe-c5261c.netlify.app

---

## Documents

| # | Document | Topic | Status |
|---|----------|-------|--------|
| 01 | [Reader UI & App Architecture](./01-reader-ui.md) | TopicWorkspace, Redux, smart highlights | ✅ Deployed |

---

## Stack

| Layer | Technology |
|-------|------------|
| Framework | React 18 + TypeScript 5.6 |
| UI | MUI v6 |
| State | Redux Toolkit + redux-saga |
| Build | Vite 5 |
| Deploy | Netlify PWA |

---

## Frontend boundary

```
✅ Reader UI, Practice UI, Dashboard (mock data)
✅ Redux state, sagas, API client
✅ TopicWorkspace, AnnotatedText, intelligence rail
✅ Feature flag: VITE_USE_MOCK_COURSES

❌ AI extraction / validation
❌ Canonical ingestion
❌ Database schema
```

---

## Feature modules

```
src/features/
├── learn/          ← Topic Learning Workspace (flagship)
├── courses/        ← Course picker
├── practice/       ← MCQ (mock today)
├── dashboard/      ← Today's plan (mock)
├── analysis/       ← Mistake analysis (mock)
└── navigator/      ← Future graph UI
```

---

## Data sources today

| Feature | Source |
|---------|--------|
| Courses / Learn | `coursesApi.ts` → mock OR live `/api/courses` |
| Everything else | `mockApi.ts` |

**Flip to live API:** `VITE_USE_MOCK_COURSES=false` in `.env`

---

## Key components

| Component | Path | Purpose |
|-----------|------|---------|
| `TopicWorkspace` | `features/learn/components/` | Main learning screen |
| `AnnotatedText` | `features/learn/components/` | Smart highlights + popover |
| `LearnPage` | `features/learn/pages/` | Route + Redux orchestration |

---

## API types (contract)

Mirror backend in `src/data/api/coursesTypes.ts`:

- `TopicWorkspaceResponse`
- `ExamIntelligence`
- `ReadingStep`, `TextHighlight`

**When backend changes → update types first.**

---

## Related

- **Student API spec:** [`../backend/04-student-apis.md`](../backend/04-student-apis.md)
- **AI intelligence model:** [`../ai/08-intelligence-engine.md`](../ai/08-intelligence-engine.md) (powers UI rails)
