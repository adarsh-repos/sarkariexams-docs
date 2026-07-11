# SarkariExamsAI — AI Platform Guide

| Field | Value |
|-------|-------|
| **Document** | Single source of truth for AI architecture |
| **Audience** | AI Team, Engineering, Product |
| **Version** | 1.1 |
| **Status** | Approved for implementation |
| **Last updated** | 2026-07-11 |

> **This replaces** all split files in `docs/ai/01–09` and wiki AI docs `06`, `09`, `10`.  
> AI Team receives this **Architecture Specification** — not ad-hoc prompts.

---

## Part 1 — High-level platform layers

SarkariExamsAI is a **Knowledge Intelligence Platform**, not a chatbot. Content flows through **eleven layers**. **Layer 3–5** are the AI platform core (orchestration, workers, validation). **Layer 11** is the only future **online** LLM use.

```mermaid
flowchart TB
    L0[Layer 0 — Content Sources]
    L1[Layer 1 — Deterministic Pipeline<br/>NO AI]
    L2[Layer 2 — Canonical Database<br/>NO AI]
    L3[Layer 3 — AI Orchestration Layer]
    L4[Layer 4 — Offline AI Workers<br/>4 workers]
    L5[Layer 5 — Validation Engine]
    L6[Layer 6 — Human Review + Publish]
    L7[Layer 7 — Knowledge Graph + Question Graph]
    L8[Layer 8 — Intelligence Engine]
    L9[Layer 9 — Student APIs<br/>SQL only, NO LLM]
    L10[Layer 10 — Reader UI]
    L11[Layer 11 — Online AI Tutor<br/>FUTURE]

    L0 --> L1 --> L2 --> L3 --> L4 --> L5 --> L6 --> L7 --> L8 --> L9 --> L10
    L7 -.-> L11
    L8 -.-> L11
    L2 -.-> L11
```

### Layer summary table

| Layer | Name | Uses AI? | Owner | One-line purpose |
|-------|------|----------|-------|------------------|
| **0** | Content Sources | No | Content Ops | Raw material: NCERT, PYQs, notes |
| **1** | Deterministic Pipeline | **No** | Backend / Content Platform | PDF → structured text (reproducible) |
| **2** | Canonical Database | **No** | Data Platform | System of record: paragraphs, hierarchy |
| **3** | **AI Orchestration Layer** | No LLM | **AI Team** | Schedule, route, and coordinate all offline AI jobs |
| **4** | Offline AI Workers | **Yes (batch)** | **AI Team** | Extract concepts, facts, relations, questions |
| **5** | **Validation Engine** | No LLM | **AI Team** | Automated schema, ontology, grounding, dedup checks |
| **6** | Human Review + Publish | No | AI Team + Content Ops | Approve staging → publish to `intelligence.*` |
| **7** | Knowledge + Question Graph | No | Data Platform | Connected intelligence nodes |
| **8** | Intelligence Engine | Mostly deterministic | AI Team + Backend | Merge graphs → exam signals |
| **9** | Student APIs | **No LLM** | Backend | Serve SQL to PWA |
| **10** | Reader UI | No | Frontend | Topic Workspace, practice |
| **11** | Online AI Tutor | **Yes (real-time)** | AI Team (future) | Grounded Q&A only |

---

## Part 2 — Every layer: purpose, input, output

### Layer 0 — Content Sources

| | |
|---|---|
| **Purpose** | Provide trustworthy source material. AI never invents content here. |
| **Input** | NCERT PDFs, reference books, PYQ papers, mock tests, class notes, current affairs |
| **Output** | Files ready for ingestion (PDF, CSV, DOC) |
| **AI role** | None |

---

### Layer 1 — Deterministic Pipeline

| | |
|---|---|
| **Purpose** | Convert PDFs to structured canonical JSON **without LLM** — same input always gives same output. |
| **Input** | PDF file, `book_id` |
| **Output** | `step10_canonical.json` → loaded to PostgreSQL |
| **AI role** | **None. Forbidden.** |

```
PDF → OCR → Layout → Reading Order → Hierarchy → Paragraphs
    → Images → Tables → Validation → Canonical JSON
```

**Why no AI:** Exam content must be factually exact. LLM PDF parsing hallucinates dates and names.

---

### Layer 2 — Canonical Database

| | |
|---|---|
| **Purpose** | Store **truth** — what the book actually says. All AI outputs must ground here. |
| **Input** | `step10_canonical.json` via load API |
| **Output** | PostgreSQL tables: `books`, `chapters`, `sections`, `paragraphs`, `figures`, `glossary_entries` |
| **AI role** | **None. AI reads from here; never writes canonical text.** |

**Key IDs:** `section_id` = topic (e.g. `SEC_2_1`), `paragraph_id` = e.g. `P00312`

---

### Layer 3 — AI Orchestration Layer ⭐ AI TEAM

| | |
|---|---|
| **Purpose** | **Coordinate all offline AI work** — when to run which worker, in what order, with what context, and how to recover from failure. Workers do not call each other directly; orchestration owns the DAG. |
| **Input** | Canonical load event (`book_id`, `version_id`), optional PYQ import event, manual re-run command |
| **Output** | Job execution plan, `extraction_run` records, worker invocations, aggregated run status |
| **AI role** | **No LLM.** Pure orchestration code (queue, DAG, retries, lineage). |

```mermaid
flowchart TB
    TRIGGER[Trigger: book loaded / PYQ import / manual]
    ORCH[AI Orchestration Layer]
    W1[Worker 1 — Content]
    W2[Worker 2 — Relationship]
    W3[Worker 3 — Question]
    VAL[Validation Engine]
    W4[Worker 4 — Exam]

    TRIGGER --> ORCH
    ORCH -->|per paragraph batch| W1
    W1 -->|section complete| W2
    ORCH -->|parallel track| W3
    W1 & W2 & W3 --> VAL
    VAL -->|approved graphs| W4
```

**Orchestration responsibilities:**

| Responsibility | Description |
|----------------|-------------|
| **Job DAG** | Enforce order: W1 → W2 per section; W3 parallel on question bank; W4 after KG+QG published |
| **Context assembly** | Build `ParagraphContext` / `TopicContext` from canonical DB before each worker call |
| **Batching** | Group paragraphs by `section_id`; token budget per batch |
| **Idempotency** | Re-run safe: same `extraction_run_id` + inputs → replace staging rows |
| **Retry policy** | Transient LLM/API failures: 3 retries with backoff; permanent failures → error artifact |
| **Lineage** | Log `book_id`, `pipeline_version`, `model_version`, `prompt_version` per run |
| **Concurrency** | Max N parallel sections; rate-limit LLM provider |
| **Trigger W4** | Only when validation pass rate ≥ threshold for book batch |

**Input example (orchestration job):**
```json
{
  "job_id": "JOB_hist_class10_v3",
  "trigger": "canonical_load_complete",
  "book_id": "hist_class10",
  "book_version_id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
  "exam_profile": { "primary": "BPSC", "stages": ["PRE", "MAINS_GS1"] },
  "workers": ["W1", "W2", "W3", "W4"],
  "options": { "dry_run": false, "sections_filter": ["SEC_3_1", "SEC_3_2"] }
}
```

**Output example (orchestration status):**
```json
{
  "job_id": "JOB_hist_class10_v3",
  "status": "running",
  "extraction_run_id": "c9bf9e57-1685-4c89-bafb-ff5af1b7e21c",
  "progress": {
    "paragraphs_total": 842,
    "paragraphs_w1_done": 412,
    "sections_w2_done": 18,
    "questions_w3_done": 156,
    "validation_pass_rate": 0.94
  },
  "worker_status": {
    "W1": "in_progress",
    "W2": "pending",
    "W3": "completed",
    "W4": "pending"
  }
}
```

**Implementation:** `backend/jobs/orchestrator.py` + queue (Celery / ARQ / cloud batch — TBD).

---

### Layer 4 — Offline AI Workers ⭐ AI TEAM CORE

| | |
|---|---|
| **Purpose** | Transform canonical text + questions into **structured exam intelligence** (concepts, facts, graph, MCQ links). |
| **Runs** | Batch only, after canonical load. Temperature = 0. |
| **AI role** | **Full ownership — 4 domain workers** |

```mermaid
flowchart LR
    subgraph W1["Worker 1 — Content"]
        P[Paragraph] --> C[Concepts Facts Entities]
    end
    subgraph W2["Worker 2 — Relationship"]
        C --> R[Graph Timeline Geography]
    end
    subgraph W3["Worker 3 — Question"]
        Q[PYQ / MCQ] --> QM[Concept map Pattern Bloom]
    end
    subgraph W4["Worker 4 — Exam"]
        R --> E[Revision PYQ freq Weightage]
        QM --> E
    end
```

#### Worker 1 — Content Intelligence

| | |
|---|---|
| **Purpose** | Answer: *What is this paragraph about? What facts must a student know?* |
| **Input** | `ParagraphContext` (see below) |
| **Output** | Concepts, atomic facts, entities, keywords/aliases, learning objectives, paragraph difficulty |

**Input example:**
```json
{
  "paragraph_id": "P00421",
  "section_id": "SEC_3_2",
  "book_id": "hist_class10",
  "text": "Lothal was an important port city of the Harappan Civilization.",
  "glossary": [{ "term": "Harappan", "meaning": "Indus Valley Civilization" }],
  "exam_profile": { "primary": "BPSC" }
}
```

**Output example:**
```json
{
  "concepts": [
    { "name": "Harappan Civilization", "category": "Civilization", "importance": "high",
      "source_paragraph_id": "P00421" },
    { "name": "Lothal", "category": "Archaeological Site", "importance": "medium",
      "source_paragraph_id": "P00421" }
  ],
  "atomic_facts": [
    { "statement": "Lothal was a port city of the Harappan Civilization.",
      "source_paragraph_id": "P00421" }
  ],
  "entities": [{ "name": "Lothal", "type": "place" }],
  "keywords": [{ "term": "port city" }],
  "learning_objectives": [
    { "objective": "Identify Lothal as a Harappan port city", "bloom_level": "remember" }
  ]
}
```

---

#### Worker 2 — Relationship Intelligence

| | |
|---|---|
| **Purpose** | Answer: *How do concepts connect? When? Where?* Build Knowledge Graph edges. |
| **Input** | Worker 1 output for a section + neighbour section metadata |
| **Output** | Relationships (SPO triples), timeline entries, geography hierarchy, synonyms, cross-topic links |

**Output example:**
```json
{
  "relationships": [
    {
      "subject": "Lothal", "predicate": "belongs_to", "object": "Harappan Civilization",
      "evidence": "Lothal was an important port city of the Harappan Civilization.",
      "source_paragraph_id": "P00421"
    }
  ],
  "timeline": [
    { "concept": "Harappan Civilization", "start": "-2600", "end": "-1900", "precision": "century" }
  ],
  "geography": [
    { "place": "Lothal", "state": "Gujarat", "country": "India", "chain": ["Gujarat", "India"] }
  ]
}
```

---

#### Worker 3 — Question Intelligence

| | |
|---|---|
| **Purpose** | Parse PYQs/MCQs and link each question to concepts and exam patterns. **Separate pipeline from paragraphs.** |
| **Input** | Raw question from question bank (CSV, PDF extract, manual entry) |
| **Output** | Parsed MCQ, concept mappings, pattern type, difficulty, Bloom level, explanation draft, confusions |

**Input example:**
```json
{
  "raw_text": "Lothal is famous for?\n(A) Steel (B) Dockyard (C) Temple (D) Coins",
  "exam": "BPSC", "year": 2022, "correct_hint": "B"
}
```

**Output example:**
```json
{
  "question_id": "Q_bpsc_hist_lothal_042",
  "stem": "Lothal is famous for?",
  "options": [...],
  "correct_option_id": "B",
  "explanation": "Lothal had a Harappan dockyard.",
  "concept_mappings": [{ "concept_id": "CONCEPT_hist10_lothal", "confidence": 0.96 }],
  "pattern": { "type": "direct_fact", "confidence": 0.93 },
  "bloom": { "level": "remember" },
  "difficulty": { "level": "easy", "score": 2 }
}
```

---

#### Worker 4 — Exam Intelligence

| | |
|---|---|
| **Purpose** | Compute exam-facing signals: revision priority, PYQ frequency, topic weightage, trends. Powers Reader intelligence rail. |
| **Input** | Published Knowledge Graph + Question Graph + `exam_profile` (BPSC, UPSC…) |
| **Output** | Per-concept and per-topic intelligence for Student APIs |

**Output example (topic — powers Reader UI):**
```json
{
  "section_id": "SEC_3_2",
  "exam_code": "BPSC",
  "why_it_matters": "Harappan sites are frequent BPSC Prelims MCQs.",
  "key_points": ["Lothal = dockyard", "Gujarat location"],
  "pyq_patterns": [
    {
      "question": "Lothal is famous for?",
      "type": "direct_fact",
      "exam": "BPSC",
      "year": 2022,
      "stage": "PRE",
      "question_id": "Q_bpsc_hist_lothal_042",
      "tip": "Link site → unique feature (dockyard)"
    },
    {
      "question": "Which Harappan site is in Gujarat?",
      "type": "direct_fact",
      "exam": "BPSC",
      "year": 2019,
      "stage": "PRE",
      "question_id": "Q_bpsc_hist_lothal_019",
      "tip": "Map + state recall"
    }
  ],
  "expected_patterns": [
    {
      "pattern": "match_site_to_feature",
      "description": "Match Harappan site to its unique feature (dockyard, granary, citadel)",
      "likelihood": "high",
      "reason": "BPSC repeats site-feature MCQs every 2–3 years; Lothal dockyard is high-frequency"
    },
    {
      "pattern": "map_based",
      "description": "Locate Lothal / Dholavira on map of Gujarat",
      "likelihood": "medium",
      "reason": "Geography linkage common in Prelims when map questions appear"
    },
    {
      "pattern": "chronology",
      "description": "Order Harappan phases or site occupation periods",
      "likelihood": "low",
      "reason": "Less common for this topic but possible in Mains analytical framing"
    }
  ],
  "remember": [{ "label": "Lothal", "hook": "L = Dockyard in Gujarat" }],
  "avoid": ["Do not confuse Lothal dockyard with Dholavira's water management"],
  "revision_priority": 0.87,
  "pyq_years_asked": [2018, 2019, 2022, 2023],
  "pyq_frequency": "high"
}
```

**Field definitions — exam intelligence topic object:**

| Field | Type | Purpose |
|-------|------|---------|
| `pyq_patterns[]` | array | **Past** questions — what BPSC **already asked** |
| `pyq_patterns[].year` | int | **Year the PYQ appeared** (required for published PYQs) |
| `pyq_patterns[].exam` | string | e.g. `BPSC`, `UPSC` |
| `pyq_patterns[].stage` | string | `PRE`, `MAINS_GS1`, etc. |
| `pyq_patterns[].question_id` | string | Link to Question Graph |
| `expected_patterns[]` | array | **Future** patterns — what **may be asked** based on trends |
| `expected_patterns[].likelihood` | enum | `high` / `medium` / `low` |
| `expected_patterns[].reason` | string | Why we predict this pattern (required for review) |
| `pyq_years_asked` | int[] | Denormalized list of all years for quick UI display |
| `pyq_frequency` | enum | `high` / `medium` / `low` / `none` |

**Rule:** `pyq_patterns` must be grounded in real PYQs (W3 / licensed bank). `expected_patterns` are **predictions** — LLM or rule-based — and **always require human review** before publish.

**Execution order:** Orchestration runs W1 → W2 (per book). W3 runs on question bank in parallel. W4 runs after KG+QG published and validation passes.

---

### Layer 5 — Validation Engine ⭐ AI TEAM

| | |
|---|---|
| **Purpose** | **Automated firewall** — every worker output is validated before staging or review. No hallucinated fact passes through. |
| **Input** | Raw JSON from Workers 1–4 + canonical paragraphs from Layer 2 |
| **Output** | `ValidationReport` per item: pass/fail, errors[], warnings[], routing (auto-staging / priority-review / reject) |
| **AI role** | **No LLM.** Deterministic validators + optional embedding dedup. |

```mermaid
flowchart TB
    IN[Worker JSON output]
    L1[L1 JSON parse]
    L2[L2 Schema validation]
    L3[L3 Ontology validation]
    L4[L4 Grounding validation]
    L5[L5 Dedup detection]
    L6[L6 Confidence routing]
    OUT{Result}
    STG[Write to staging]
    REJ[Reject + error artifact]
    PRIO[Priority review queue]

    IN --> L1 --> L2 --> L3 --> L4 --> L5 --> L6 --> OUT
    OUT -->|pass, confidence ≥ 0.85| STG
    OUT -->|pass, confidence < 0.85| PRIO
    OUT -->|fail| REJ
```

**Validation levels:**

| Level | Engine module | Checks |
|-------|---------------|--------|
| **L1 JSON** | `validators/json.py` | Parseable; matches worker JSON Schema |
| **L2 Schema** | `validators/schema.py` | Required fields, enums, ID regex |
| **L3 Ontology** | `validators/ontology.py` | Valid predicates; DAG for prerequisites; MCQ rules (1 correct) |
| **L4 Grounding** | `validators/grounding.py` | `paragraph_id` exists; facts in text; no new dates/names |
| **L5 Dedup** | `validators/dedup.py` | Near-duplicate questions (cosine > 0.92); duplicate concept slugs |
| **L6 Confidence** | `validators/confidence.py` | Route `< 0.70` → reject; `0.70–0.85` → priority review |

**Input example:**
```json
{
  "worker": "content_intelligence",
  "extraction_run_id": "c9bf9e57-…",
  "paragraph_id": "P00421",
  "payload": { "concepts": [...], "atomic_facts": [...] },
  "canonical_paragraphs": { "P00421": "Lothal was an important port city..." }
}
```

**Output example (`ValidationReport`):**
```json
{
  "worker": "content_intelligence",
  "paragraph_id": "P00421",
  "passed": true,
  "errors": [],
  "warnings": [
    { "level": "confidence", "check_id": "L6-001", "message": "Concept confidence 0.82 — priority review" }
  ],
  "routing": "priority_review",
  "metrics": { "concepts_validated": 2, "facts_validated": 3, "duration_ms": 45 }
}
```

**W4-specific validation (exam intelligence):**
| Check | Rule |
|-------|------|
| `V-W4-001` | Every `pyq_patterns[].year` required when `source=pyq` |
| `V-W4-002` | `expected_patterns[].reason` ≥ 20 chars |
| `V-W4-003` | `expected_patterns` must not cite fake PYQ years |
| `V-W4-004` | `pyq_years_asked` must match union of `pyq_patterns[].year` |

---

### Layer 6 — Human Review + Publish

| | |
|---|---|
| **Purpose** | Content Ops approves staging rows — especially `expected_patterns`, low-confidence extractions, and new PYQ mappings. |
| **Input** | Staging rows that passed Validation Engine (or routed to priority review) |
| **Output** | `status = published` rows in `intelligence.*`; incremented `pack_version` |
| **AI role** | None. Human gate per ADR-002. |

**AI Team implements:** Review API + Publish Worker. Content Ops owns approval SLA.

---

### Layer 7 — Knowledge Graph + Question Graph

| | |
|---|---|
| **Purpose** | Store **connected** intelligence — not flat JSON files. |
| **Input** | Approved staging rows from Layer 4 |
| **Output** | Queryable graph in PostgreSQL `intelligence.*` |

**Knowledge Graph (from W1 + W2):**
```
Concept → Facts → Entities → Relationships → Timeline → Geography
```

**Question Graph (from W3):**
```
Question → tests → Concept → grounded_in → Paragraph
         → Pattern, Bloom, Difficulty
```

**AI role:** None at runtime — stores published AI output.

---

### Layer 8 — Intelligence Engine

| | |
|---|---|
| **Purpose** | Merge Knowledge Graph × Question Graph → per-concept exam intelligence. |
| **Input** | Both graphs + exam profile |
| **Output** | `revision_priority`, `pyq_count`, `importance_score`, smart highlight candidates, topic workspace bundle |

**Per concept, compute:**

| Signal | Source |
|--------|--------|
| Mentioned in N books | KG |
| Paragraph / fact count | KG |
| PYQ count & patterns | QG |
| Common confusions | QG + traps |
| Revision priority | W4 formula |
| Related concepts | KG 1-hop edges |

**AI role:** W4 + mostly SQL aggregations. Optional LLM for narrative `why_it_matters` (must be reviewed).

---

### Layer 9 — Student APIs

| | |
|---|---|
| **Purpose** | Serve intelligence to PWA. **Zero LLM.** |
| **Input** | HTTP requests (`topic_id`, `concept_id`, `user_id`) |
| **Output** | JSON from SQL joins on canonical + intelligence tables |

| Endpoint (target) | Returns |
|-------------------|---------|
| `GET /api/courses/.../workspace` | Reading + exam intelligence rail |
| `GET /api/concepts/{id}` | Concept node + facts |
| `GET /api/concepts/{id}/pyqs` | Linked PYQs |
| `GET /api/practice/sessions` | MCQs by topic/concept |
| `GET /api/revision/today` | Top revision concepts |

**AI role:** None.

---

### Layer 10 — Reader UI

| | |
|---|---|
| **Purpose** | Student experience: Topic Learning Workspace, highlights, practice. |
| **Input** | Student APIs JSON |
| **Output** | Rendered UI (no AI) |

**AI role:** None. Displays Layer 8 output — including `pyq_patterns` (with year), `expected_patterns`, highlights, traps.

---

### Layer 11 — Online AI Tutor (future)

| | |
|---|---|
| **Purpose** | Answer student doubts **grounded** in published knowledge. |
| **Input** | Student question + `user_id` |
| **Output** | Answer with citations (`paragraph_id`, `concept_id`, `fact_id`) |

```
Student question → Retriever (KG + paragraphs + facts + PYQs) → LLM → Grounded answer
```

**AI role:** **Only production LLM.** Must not invent facts. Build after Layers 3–8 are stable.

---

## Part 3 — AI input/output master table

| Component | Trigger | Input (from Engineering) | Output |
|-----------|---------|--------------------------|--------|
| **Orchestration** | Book loaded / manual | `book_id`, `version_id`, exam_profile, worker DAG | job status, `extraction_run_id`, worker invocations |
| **W1 Content** | Orchestrator dispatch | `paragraph_id`, `text`, `section_id`, `book_id`, glossary, exam_profile | concepts, atomic_facts, entities, keywords, learning_objectives |
| **W2 Relationship** | W1 section complete | W1 outputs + neighbour sections | relationships, timeline, geography, synonyms, cross_refs |
| **W3 Question** | PYQ import / batch | raw question text, exam, year, answer hint | parsed MCQ, concept_mappings, pattern, bloom, difficulty, explanation |
| **W4 Exam** | KG + QG published | graphs + exam_profile | topic intelligence incl. `pyq_patterns` (with year), `expected_patterns` |
| **Validation Engine** | After each worker | worker JSON + canonical paragraphs | `ValidationReport`, routing, staging or reject |
| **Review + Publish** | Staging ready | staging rows | approved → `intelligence.*` published |

---

## Part 4 — Grounding rules (anti-hallucination)

Every AI output that contains a **fact** must satisfy:

| Rule | Meaning |
|------|---------|
| **G-001** | `source_paragraph_id` exists in canonical DB |
| **G-002** | Highlight/fact terms appear in source paragraph |
| **G-003** | Years in output ⊆ years in source text |
| **G-004** | No new proper nouns not in paragraph |
| **G-005** | MCQ explanation uses only mapped concept facts |
| **G-006** | Human review before `status = published` |

---

## Part 5 — Data stored after AI (key entities)

| Entity | ID pattern | Created by | Used by |
|--------|------------|------------|---------|
| Concept | `CONCEPT_{book}_{slug}` | W1 | Graph, practice, UI |
| Atomic Fact | `FACT_{uuid}` | W1 | MCQ grounding, tutor |
| Entity | `ENT_{slug}` | W1 | Graph, geography |
| Relationship | `REL_{...}` | W2 | Navigator, Mains linkages |
| Question | `Q_{exam}_{slug}_{seq}` | W3 | Practice engine |
| Highlight | `HL_{section}_{slug}` | W1/W4 | AnnotatedText UI |
| Trap | `TRAP_{section}_{slug}` | W1/W3 | "Avoid these traps" rail |
| PYQ Pattern | `PYQ_{exam}_{year}_{seq}` | W3/W4 | "How BPSC tests it" rail |
| Expected Pattern | `EXP_{section}_{slug}` | W4 | "What may be asked" rail |

**Schemas:** PostgreSQL `intelligence_staging.*` → review → `intelligence.*`

---

## Part 6 — What AI team owns vs does not own

### Owns ✅
- AI Orchestration Layer (job DAG, batching, lineage)
- 4 offline workers
- **Validation Engine** (L1–L6 automated validators)
- JSON schemas for structured LLM output
- Staging tables + extraction jobs
- Model pinning, metrics, golden tests
- Prompts (internal, versioned — not the contract)

### Does not own ❌
- PDF ingestion (Layer 1)
- Canonical schema (Layer 2)
- Student APIs (Layer 7)
- Reader UI (Layer 8)
- PYQ licensing (Product/Legal)

---

## Part 7 — Implementation phases

| Phase | Scope | Weeks |
|-------|-------|-------|
| **1** | Orchestration + W1 + W2 + Validation Engine for `hist_class10` CH 1–3 | 8 |
| **2** | W3 on BPSC PYQ sample + Question Graph | 6 |
| **3** | W4 (`pyq_patterns` + `expected_patterns`) + Intelligence Engine + API | 4 |
| **4** | Full book + review UI | 4 |

---

## Part 8 — End-to-end example (one paragraph)

**Canonical input (Layer 2):**
> "In April 1919, Gandhiji organised a satyagraha against the Rowlatt Act."

**W1 output:** Concepts `Rowlatt Act`, `Satyagraha`; Fact `"Rowlatt Act passed 1919"`; Entity `Gandhi`

**W2 output:** `Rowlatt Act` —caused→ `Satyagraha`; Timeline `1919`

**W3 output (PYQ):** "Rowlatt Act allowed detention without trial" → maps to `CONCEPT_rowlatt`

**W4 output:** Topic intelligence — PYQ patterns with years `[2019, 2022]`; expected_patterns `cause_effect`, `direct_fact`; revision_priority 0.91

**Student sees (Layer 10):** Highlight on "Rowlatt Act"; rail shows "Asked in BPSC 2019, 2022"; expected pattern "match Act → provision"

---

## Part 9 — Related documents

| Doc | Location |
|-----|----------|
| Deterministic pipeline (Layer 1–2) | [`../backend/02-ingestion-pipeline.md`](../backend/02-ingestion-pipeline.md) |
| Canonical DB schema | [`../backend/03-canonical-database.md`](../backend/03-canonical-database.md) |
| Student APIs | [`../backend/04-student-apis.md`](../backend/04-student-apis.md) |
| Reader UI | [`../frontend/01-reader-ui.md`](../frontend/01-reader-ui.md) |
| ADR: No AI in ingestion | [`./adr/001-deterministic-ingestion-first.md`](./adr/001-deterministic-ingestion-first.md) |
| ADR: Human review gate | [`./adr/002-offline-extraction-human-review-gate.md`](./adr/002-offline-extraction-human-review-gate.md) |

---

## Appendix — Why 4 workers, not 9 agents

Early design had 9 micro-agents (Concept, Fact, Entity, Relation, Timeline, Geography, Keyword, Learning Objective, Difficulty).

**We use 4 domain workers** because enterprise AI teams (Google, Microsoft, OpenAI, Anthropic) organize by **business capability**, not one agent per field:

| 9 agents problem | 4 workers solution |
|------------------|-------------------|
| 9× orchestration | Clear ownership |
| Fragmented context | Shared `ParagraphContext` |
| Hard to test/version | One golden suite per worker |
| Duplicate LLM calls | Batched structured extraction |

| Original agent | Worker |
|----------------|--------|
| Concept, Fact, Entity, Keyword, Learning Objective | **W1** |
| Relationship, Timeline, Geography | **W2** |
| Question parse, concept map, pattern, Bloom | **W3** |
| Difficulty, revision, weightage, trends | **W4** |
