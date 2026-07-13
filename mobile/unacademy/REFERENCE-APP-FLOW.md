# Reference App Flow — Business Summary

| Field | Value |
|---|---|
| Purpose | Summarise the supplied reference screens for business discussion |
| Scope | Interaction patterns only — not brand, copy, content, or UI replication |
| Source set | 96 supplied screenshots reduced to 28 representative screens |
| Target product | SarkariExamsAI BPSC mobile app |

## Executive summary

The reference product is built around a clear exam-preparation loop:

```mermaid
flowchart LR
    Login["Phone login and OTP"] --> Home["Daily dashboard"]
    Home --> Subjects["Subject selection"]
    Subjects --> Learn["Units and lessons"]
    Learn --> Checkpoint["Practice checkpoint"]
    Checkpoint --> Setup["Practice setup"]
    Setup --> Prelims["Prelims practice and revision"]
    Home --> Mains["Mains PYQs and daily question"]
    Home --> News["Current affairs"]
    News --> Prelims
    Prelims --> Review["Performance review"]
    Mains --> Review
```

Its strongest idea is not any individual screen. It is the way the app makes a student repeatedly choose the next small action: resume a lesson, attempt a question, revise a weak topic, or write a Mains answer.

SarkariExamsAI should retain this behaviour, while differentiating through canonical NCERT/reference reading, BPSC-specific exam intelligence, source-backed PYQs, and reviewed guidance.

## Representative flow

| Step | Representative screen | What the reference demonstrates | SarkariExamsAI decision |
|---|---|---|---|
| 1 | `01-login-phone.png` | Phone-number login introduces identity before personalized progress. | Use a BPSC-branded OTP flow with privacy consent and no mandatory marketing opt-in. |
| 2 | `02-login-otp.png` | OTP verification completes the login journey. | Securely store session credentials and explain retry/error states. |
| 3 | `03-home-dashboard.png` | A dashboard offers several entry points: daily work, current affairs, practice, and learning continuation. | One primary “continue/next action” plus secondary BPSC utilities. |
| 4 | `04-subject-list.png` | Subjects are grouped into syllabus domains with unit counts. | Use BPSC syllabus groups and canonical subject IDs. |
| 5 | `05-subject-grid.png` | A visual card alternative makes broad subjects easier to scan. | Offer a compact/list accessibility mode; original artwork only. |
| 6 | `06-subject-units.png` | Units make a large syllabus navigable and show completion. | Keep subject → unit → lesson hierarchy based on canonical IDs. |
| 7 | `07-ordered-lesson-path.png` | Lessons are sequenced and a practice sheet appears at the appropriate learning checkpoint. | Link reading sections to BPSC Prelims practice, rather than generic sheets. |
| 8 | `08-lesson-lock-gate.png` | Progression gates encourage sequence. | Use prerequisites only when educationally necessary; never obscure core NCERT access. |
| 9 | `09-prelims-home.png` | Prelims gets a dedicated destination and daily action. | Separate Prelims workspace using `PRE` PYQs, timed practice, and revision. |
| 10 | `10-practice-hub.png` | Practice is split into sectional, current-affairs and full-length paths. | Start with topic/PYQ practice; add full tests only with a valid BPSC blueprint. |
| 11 | `11-practice-configuration.png` | Learners choose subject, topic, question count and bank before starting. | Use BPSC stage, subject, topic, PYQ years and attempt history as filters. |
| 12 | `12-practice-test-modes.png` | Practice and timed-test modes set different learning expectations. | Support learning mode and BPSC test simulation with transparent scoring. |
| 13 | `13-marking-scheme.png` | The exam marking rule is shown before the test begins. | Display the current official BPSC marking rule with source/version metadata. |
| 14 | `14-question-generation.png` | Loading state communicates session preparation. | Say “Preparing practice set”; use deterministic/session-backed selection, not opaque live generation. |
| 15 | `15-mcq-question.png` | A focused question screen supports one response at a time. | Include question source, stage, accessibility support and recoverable progress. |
| 16 | `16-exit-practice-confirmation.png` | An exit safeguard prevents accidental abandonment. | Save a resumable attempt locally/server-side; never trap a learner. |
| 17 | `17-pyq-filter.png` | PYQs are filtered by year, topic and attempt history. | Support BPSC cycle/year, stage, subject, topic, bookmarks and incorrect-only filters. |
| 18 | `18-overall-progress.png` | One screen combines completion, accuracy, MCQs and Mains activity. | Show source-backed progress metrics, separated by Prelims and Mains. |
| 19 | `19-subject-performance.png` | Subject-level strength analysis identifies revision need. | Derive strength from attempts, accuracy, recency and verified topic priority. |
| 20 | `20-progress-comparison.png` | Comparative trends can motivate more practice. | Optional and carefully worded; never shame learners or make unverifiable topper claims. |
| 21 | `21-revision-areas.png` | Revision is based on strength/weakness and high-repeat topics. | Combine learner accuracy with validated BPSC PYQ frequency and revision priority. |
| 22 | `22-mains-pyq-list.png` | Mains questions are searchable/filterable, with marks and subject labels. | BPSC Mains GS-I filters: subject, topic, year, marks, and question pattern. |
| 23 | `23-daily-mains-question.png` | A single daily Mains prompt reduces choice overload. | One BPSC daily question with word limit, cited framework, saved drafts, and evaluation status. |
| 24 | `24-current-affairs-feed.png` | Current affairs are grouped into exam-relevant themes and lead to MCQs. | Source-linked Bihar/India/world feed with reviewed tags, highlights, and practice links. |
| 25 | `25-current-affairs-article.png` | An article expands to highlights and detail. | Preserve citations and mark reviewed explanatory content; avoid unaudited “why it matters” prose. |
| 26 | `26-news-practice.png` | Current-affairs practice can be narrowed by subject and month. | Link reviewed articles to a BPSC practice set with source citations. |
| 27 | `27-shorts-feed.png` | Short media supports discovery and quick revision. | Later feature; any short must be linked to a source/topic, not a substitute for reading. |
| 28 | `28-leaderboard.png` | Ranking can provide motivation. | Optional after learning-loop metrics are healthy; show fair, explainable scoring. |

## Serial user journey

```mermaid
flowchart TD
    S01["01 Login phone"] --> S02["02 Verify OTP"]
    S02 --> S03["03 Home dashboard"]
    S03 --> S04["04 Subject list"]
    S03 --> S05["05 Subject grid (alternate)"]
    S04 --> S06["06 Unit catalog"]
    S05 --> S06
    S06 --> S07["07 Ordered lesson path"]
    S07 --> S08["08 Lesson gate (optional)"]
    S07 --> S09["09 Prelims hub"]
    S09 --> S10["10 Practice hub"]
    S10 --> S11["11 Configure session"]
    S11 --> S12["12 Choose practice or test"]
    S12 --> S13["13 Confirm marking scheme"]
    S13 --> S14["14 Prepare question set"]
    S14 --> S15["15 Attempt MCQs"]
    S15 --> S16["16 Save or exit safely"]
    S11 --> S17["17 PYQ filter"]
    S15 --> S18["18 Overall progress"]
    S18 --> S19["19 Subject strength"]
    S19 --> S20["20 Progress comparison"]
    S20 --> S21["21 Revision areas"]
    S03 --> S22["22 Mains PYQs"]
    S22 --> S23["23 Daily Mains question"]
    S03 --> S24["24 Current affairs feed"]
    S24 --> S25["25 Article detail"]
    S25 --> S26["26 Current-affairs practice"]
    S03 --> S27["27 Shorts"]
    S03 --> S28["28 Leaderboard"]
```

## Screens intentionally removed from the representative pack

The source set contains multiple near-duplicate views of:

- Unit listings across different History/Art & Culture sections
- The same lesson path at different scroll positions
- Mains PYQ filters for type, subject, and year
- Leaderboards split by MCQ, daily Mains question, and daily Prelims question
- Article detail screens split between highlights and long-form content
- Login variants, loading variants, and progress views that expose the same interaction state

The retained 28 screens now cover the complete mobile journey in serial order: login → subject selection → learning → practice setup → attempts → progress/revision → Mains/current affairs.

## What to replicate vs not replicate

| Retain as a product pattern | Do not copy |
|---|---|
| Ordered syllabus path and visible progress | Reference brand, colors, iconography, wording, or screenshots |
| Separate Prelims and Mains journeys | Generic course/video-first information architecture |
| Practice at learning checkpoints | Paywall/lock pattern without approved entitlement policy |
| Revision based on performance | Unverifiable rankings or engagement mechanics as the core loop |
| Current affairs → explanation → practice path | Unreviewed AI-written facts or model answers |
| Filters for large PYQ collections | Third-party content, images, or publisher material |

## Product implication for SarkariExamsAI

```mermaid
flowchart TD
    Canonical["Canonical NCERT and licensed sources"] --> Topic["Topic reader"]
    Intelligence["Published BPSC intelligence"] --> Topic
    Topic --> Prelims["Linked Prelims practice"]
    Topic --> Mains["Linked Mains prompts"]
    Prelims --> Revision["Revision queue"]
    Mains --> Revision
    CurrentAffairs["Reviewed current affairs"] --> Prelims
    CurrentAffairs --> Mains
```

The reference design is strongest as a model for **mobile habit formation**. SarkariExamsAI’s defensible advantage must be **truthful, source-backed exam intelligence**, not visual similarity.

## Source PDF

The supplied reference file is stored as [`Economy-NCERT_v8.pdf`](./Economy-NCERT_v8.pdf).

It is reference material only and must not be republished as SarkariExamsAI content without confirming its licensing and permitted use.
