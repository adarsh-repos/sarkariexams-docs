# Intelligence Engine

| Document | `AI-08` |
|----------|---------|

---

## Purpose

Merge **Knowledge Graph × Question Graph** → per-concept and per-topic **Exam Intelligence** for Student APIs and Reader UI.

Implemented primarily as **Worker 4** + SQL materialized views.

---

## Merge diagram

```
Knowledge Graph                    Question Graph
     │                                    │
     ├─ Concepts ─────────────────────────┤
     ├─ Facts                             ├─ Questions
     ├─ Entities                          ├─ Patterns
     ├─ Relationships                     ├─ Difficulty
     ├─ Timeline                          ├─ Bloom
     └─ Geography                         └─ Confusions
                    │
                    ▼
           INTELLIGENCE ENGINE
                    │
     ┌──────────────┼──────────────┐
     ▼              ▼              ▼
Per-Concept    Per-Topic      Exam Trends
Intelligence   Intelligence   (BPSC 2025)
```

---

## Per-concept output

For every concept, generate:

| Field | Source |
|-------|--------|
| Mentioned in N books | KG cross-book count |
| Paragraph count | KG |
| Atomic fact count | KG |
| PYQ count | QG |
| Question patterns | QG aggregate |
| Difficulty distribution | QG |
| Common confusions | QG + traps |
| Revision priority | W4 formula |
| Related concepts | KG 1-hop edges |
| Learning objectives | KG |

---

## Per-topic output (Reader UI)

Powers **Topic Learning Workspace** right rail:

- Why it matters
- Key points
- How BPSC tests it (PYQ patterns)
- Remember this (memory anchors)
- Avoid these traps

**Frontend type:** `ExamIntelligence` in `coursesTypes.ts`

---

## Smart highlights

Generated from:
- Keywords (W1)
- High-importance facts (W1 + W4)
- Common confusions (W3)

→ `text_highlights` table → `AnnotatedText` component

---

## Revision engine (future)

```
revision_priority =
  w1 * pyq_frequency
+ w2 * importance_score
+ w3 * user_weakness (future)
- w4 * days_since_last_review (future)
```

---

## APIs (SQL only)

| Endpoint | Engine output |
|----------|---------------|
| `/workspace` | Topic intelligence bundle |
| `/concept/{id}` | Full concept node |
| `/revision/today` | Top N concepts |
| `/practice/sessions` | Questions by concept + difficulty |

**No LLM in these paths.**
