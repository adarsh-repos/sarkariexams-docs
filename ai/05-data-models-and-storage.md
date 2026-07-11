# Data Models & Storage

| Document | `AI-05` |
|----------|---------|

---

## Schema layout

```
PostgreSQL
├── public.*              ← Canonical (Engineering owns, NO AI writes)
└── intelligence.*        ← Published intelligence (AI → review → publish)
└── intelligence_staging.* ← Draft (AI writes here only)
```

---

## Core entities (AI team implements)

### Concepts

| Column | Type | Purpose |
|--------|------|---------|
| `concept_id` | VARCHAR PK | `CONCEPT_{book}_{slug}` |
| `book_id` | FK | Parent book |
| `name` | VARCHAR | Display name |
| `aliases` | JSONB | Alternate names |
| `category` | ENUM | Civilization, Person, Event… |
| `description` | TEXT | Grounded definition |
| `importance` | ENUM | high / medium / low |
| `pack_version` | INT | Versioning |
| `status` | ENUM | draft / published / archived |

### Atomic Facts

| Column | Type | Purpose |
|--------|------|---------|
| `fact_id` | VARCHAR PK | `FACT_{uuid}` |
| `statement` | TEXT | Single exam-ready claim |
| `concept_id` | FK | Primary concept |
| `source_paragraph_id` | FK | **Grounding** |
| `fact_type` | ENUM | geographic, membership, feature… |
| `confidence` | FLOAT | Model score |

### Entities

| Column | Type | Purpose |
|--------|------|---------|
| `entity_id` | VARCHAR PK | `ENT_{slug}` |
| `name` | VARCHAR | Ashoka, Gujarat, etc. |
| `type` | ENUM | person, place, dynasty… |
| `concept_id` | FK nullable | Link to concept |

### Relationships

| Column | Type | Purpose |
|--------|------|---------|
| `subject_id` | VARCHAR | SPO subject |
| `predicate` | ENUM | belongs_to, ruled, capital_of… |
| `object_id` | VARCHAR | SPO object |
| `evidence` | TEXT | Why edge exists |
| `source_paragraph_id` | FK | Grounding |

### Timeline

| Column | Type | Purpose |
|--------|------|---------|
| `concept_id` | FK | |
| `start_date` | VARCHAR | ISO or BCE string |
| `end_date` | VARCHAR | |
| `precision` | ENUM | year, century, period |
| `period_label` | VARCHAR | Human label |

### Geography

| Column | Type | Purpose |
|--------|------|---------|
| `entity_id` | FK | Place entity |
| `state` | VARCHAR | |
| `country` | VARCHAR | |
| `parent_chain` | JSONB | [Gujarat, India] |

### Questions

| Column | Type | Purpose |
|--------|------|---------|
| `question_id` | VARCHAR PK | |
| `stem` | TEXT | |
| `options` | JSONB | 4 options |
| `correct_option_id` | CHAR(1) | A–D |
| `explanation` | TEXT | Required |
| `difficulty` | INT | 1–5 |
| `bloom_level` | ENUM | |
| `pattern_type` | ENUM | |
| `exam_code` | VARCHAR | BPSC |
| `year` | INT | PYQ year |

### Junction tables

| Table | Links |
|-------|-------|
| `question_concept_links` | Question ↔ Concept |
| `question_fact_links` | Question ↔ Fact |
| `concept_paragraph_mentions` | Concept ↔ Paragraph |

---

## ID standards

Full reference: [`../backend/05-naming-governance.md`](../backend/05-naming-governance.md)

---

## Example: complete concept record in DB

```json
{
  "concept_id": "CONCEPT_hist10_lothal",
  "book_id": "hist_class10",
  "name": "Lothal",
  "aliases": ["Lothal port city"],
  "category": "Archaeological Site",
  "description": "Harappan port city in Gujarat known for dockyard.",
  "importance": "high",
  "status": "published",
  "pack_version": 1
}
```

---

## Migration

- `alembic/003_intelligence_staging.py` — staging tables
- `alembic/004_intelligence_published.py` — published tables

Owned by: Data Platform + AI Platform jointly.
