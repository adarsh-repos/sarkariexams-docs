# Validation Layer

| Document | `AI-06` |
|----------|---------|

**No AI output enters the database without passing this pipeline.**

---

## Flow

```
AI Worker JSON
      ↓
┌─────────────────────────────────────┐
│ Level 1: JSON Validation            │
│  - Parseable JSON                   │
│  - JSON Schema per worker output    │
└─────────────────────────────────────┘
      ↓
┌─────────────────────────────────────┐
│ Level 2: Schema Validation          │
│  - Required fields                  │
│  - Enum values                      │
│  - ID format regex                  │
└─────────────────────────────────────┘
      ↓
┌─────────────────────────────────────┐
│ Level 3: Ontology Validation        │
│  - Valid predicates                 │
│  - DAG for prerequisite/precedes    │
│  - FK references exist              │
└─────────────────────────────────────┘
      ↓
┌─────────────────────────────────────┐
│ Level 4: Grounding Validation       │
│  - paragraph_id exists              │
│  - facts derivable from text        │
│  - no new dates/names               │
│  - highlight terms in paragraph     │
└─────────────────────────────────────┘
      ↓
┌─────────────────────────────────────┐
│ Level 5: Duplicate Detection        │
│  - Near-duplicate questions         │
│  - Duplicate concept slugs          │
└─────────────────────────────────────┘
      ↓
┌─────────────────────────────────────┐
│ Level 6: Confidence Score             │
│  - Route < 0.70 → priority review     │
│  - Route < 0.50 → auto-reject       │
└─────────────────────────────────────┘
      ↓
   Review Queue (human)
      ↓
   Publish Worker → intelligence.*
```

---

## Grounding rules (critical)

| ID | Rule | Example failure |
|----|------|-----------------|
| `G-001` | `source_paragraph_id` exists | P99999 |
| `G-002` | Fact statement ⊆ paragraph meaning | Invents "steel production" |
| `G-003` | Years in output ⊆ years in paragraph | Adds 1920 when only 1919 |
| `G-004` | New proper nouns flagged | Invents "Smith Commission" |
| `G-005` | MCQ explanation cites known facts | Option B reason uses unknown treaty |

---

## Ontology rules

| ID | Rule |
|----|------|
| `O-001` | `predicate` ∈ allowed list |
| `O-002` | `PREREQUISITE` / `precedes` → DAG |
| `O-003` | `concept.category` ∈ ontology |
| `O-004` | `pattern.type` ∈ question pattern taxonomy |
| `O-005` | `bloom.level` ∈ Bloom enum |

---

## Output: validation report

```json
{
  "worker": "content_intelligence",
  "paragraph_id": "P00421",
  "passed": false,
  "errors": [
    { "level": "grounding", "id": "G-003", "message": "Year 1500 BCE not in paragraph" }
  ],
  "warnings": [
    { "level": "confidence", "message": "Concept confidence 0.68 — priority review" }
  ]
}
```

---

## Ownership

**AI team implements validators** in `backend/intelligence/validators/`.

**Content Ops** owns review queue SLA and approval.

---

## Testing

- Unit test per rule ID
- Golden files: 20 paragraphs with expected pass/fail
- Regression on prompt version bump
