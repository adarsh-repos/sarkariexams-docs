# 01 — Mobile MVP Production Plan

| Field | Value |
|---|---|
| Status | Draft for cross-functional approval |
| Product | SarkariExamsAI mobile |
| Platform | Separate Expo / React Native client for iOS and Android |
| Primary release | BPSC Prelims learning loop; Mains submission is conditional |
| Primary audience | Product, mobile, backend, assessment, content operations, QA, and leadership |
| Source of truth | [00 Product Brief](./00-mobile-product-brief.md), [02 Delivery Plan](./02-delivery-plan.md), [03 Backend Contract](./03-mobile-backend-contract.md), [04 UI Architecture](./04-expo-ui-architecture.md) |

## 1. Production decision

The first production release must prove one valuable, repeatable learner loop:

```mermaid
flowchart LR
    Login["Authenticate"] --> Home["See one next action"]
    Home --> Learn["Read one canonical topic"]
    Learn --> Practice["Attempt linked Prelims MCQs"]
    Practice --> Review["See result and mistakes"]
    Review --> Revise["Receive revision action"]
    Revise --> Home
```

This is a BPSC preparation product, not a video catalogue, generic chatbot, or visual clone of a reference app. Every student-facing fact, PYQ, explanation, and recommendation must come from published, source-backed content.

### Release definition

| Release tier | Included capability | Decision |
|---|---|---|
| **Production core** | Foundation, authenticated Home, Learn, topic completion, topic-linked Prelims practice, results, minimal revision | Required for launch |
| **Conditional production scope** | Mains GS-I PYQ discovery plus offline-safe draft and submission | Include only if Assessment APIs and content meet the readiness gate |
| **Fast follow** | Mains evaluation, reviewed current affairs, published intelligence rail | Release after core-loop quality is proven |
| **Deferred** | Payments, notifications, targets, short-form media, social leaderboards, full-length mocks, generic chat | Do not block production core |

## 2. Goals, users, and non-goals

### Product goal

Enable a serious BPSC aspirant who primarily studies on a phone to move reliably from canonical reading to stage-specific practice and revision in one session.

### Target learner

| Attribute | Product implication |
|---|---|
| Limited study time | One primary action and a resumable next step |
| Phone-first study | Native mobile reading, touch-first controls, dynamic type, offline recovery |
| Preparing for both stages | Separate `PRE` and `MAINS_GS1` workflows using shared topic identifiers |
| Needs trustworthy guidance | Published sources, citations, content versions, and reviewed intelligence only |

### Measurable outcomes

| Outcome | Initial target | Decision supported |
|---|---:|---|
| Activation | First lesson opened within 3 minutes of install | Onboarding and catalog clarity |
| Learn-loop adoption | 40%+ of weekly learners complete a lesson | Whether the lesson experience is useful |
| Read-to-practice conversion | 30%+ attempt linked practice after a lesson | Whether practice checkpoints create value |
| Learning impact | +15% accuracy lift after related reading | Whether knowledge intelligence improves performance |
| Retention | 30%+ D7 retention | Whether next action and revision form a habit |
| Mains adoption | Track eligible-user submissions | Whether to expand Mains workflow |

### Non-goals for the first production release

- Live classes, creator marketplace, or a copied coaching-course model.
- Generic real-time AI chat or unreviewed AI explanations.
- Payments or a paywall before entitlement and free-preview rules are approved.
- Leaderboards, shorts, or comparative “topper” claims as a core motivation loop.
- Current-affairs publishing before provenance and editorial review are operational.
- Republishing third-party reference PDFs, screenshots, branding, copy, or visual assets.

## 3. Release-critical functional requirements

The reference journey in [Reference/REFERENCE-APP-FLOW.md](./Reference/REFERENCE-APP-FLOW.md) informs interaction patterns only. It does not authorize visual or content replication.

The **Reference screenshots** column attaches the relevant internal reference images for each flow. A “No visual reference” entry means the requirement is a production need not represented by the captured reference journey; it still requires original SarkariExamsAI design.

### 3.1 Identity and launch

| ID | Requirement | Priority | Acceptance criteria | Dependency | Reference screenshots |
|---|---|---|---|---|---|
| FR-01 | The app restores an existing session on launch and otherwise guides the learner through approved authentication. | Must | No learner progress is shown without the correct user context. Expired sessions lead to a recoverable sign-in state. | Identity: `GET /api/me`, refresh, logout | <img src="./Reference/functional-requirement-screens/01-login-phone.png" alt="01 Phone login" width="72"> <img src="./Reference/functional-requirement-screens/02-login-otp.png" alt="02 OTP verification" width="72"> |
| FR-02 | Credentials are stored only with Expo SecureStore. | Must | No access token, refresh token, or raw Mains answer is emitted in analytics or standard logs. | Mobile platform | <img src="./Reference/functional-requirement-screens/01-login-phone.png" alt="01 Phone login" width="72"> <img src="./Reference/functional-requirement-screens/02-login-otp.png" alt="02 OTP verification" width="72"> |
| FR-03 | The learner can select BPSC/profile preferences required to personalise Home. | Must | Profile selection produces a stable learner and exam context. | Identity and Student API | <img src="./Reference/functional-requirement-screens/03-home-dashboard.png" alt="03 Home with BPSC selector" width="72"> |

**Product decision required:** confirm phone OTP provider and whether anonymous reading is allowed. If guest reading is approved, progress merging and conflict resolution must be specified before implementation.

### 3.2 Home and next action

| ID | Requirement | Priority | Acceptance criteria | Dependency | Reference screenshots |
|---|---|---|---|---|---|
| FR-04 | Home displays one prominent, resumable next action. | Must | A returning learner can continue reading or practice without navigating a catalog first. | `GET /api/home` (proposed) | <img src="./Reference/functional-requirement-screens/03-home-dashboard.png" alt="03 Home dashboard" width="72"> |
| FR-05 | Home presents only valid secondary actions: Learn and Prelims; Mains appears when enabled for the release. | Must | No unavailable destination leads to a dead end. | Mobile configuration | <img src="./Reference/functional-requirement-screens/03-home-dashboard.png" alt="03 Home navigation" width="72"> |
| FR-06 | Home handles first-use, empty-plan, stale-cache, error, and signed-out states. | Must | Each state provides a specific recovery action rather than an empty screen. | Mobile and Student API | No visual reference — original empty/error states required |

### 3.3 Learn: catalog, reader, and completion

| ID | Requirement | Priority | Acceptance criteria | Dependency | Reference screenshots |
|---|---|---|---|---|---|
| FR-07 | Learners navigate canonical hierarchy: subject → unit → ordered lesson → topic reader. | Must | Topic order and labels match stable `book_id`, `chapter_id`, and `topic_id` values. | Existing course endpoints | <img src="./Reference/functional-requirement-screens/04-subject-list.png" alt="04 Subject list" width="58"> <img src="./Reference/functional-requirement-screens/05-subject-grid.png" alt="05 Subject grid" width="58"> <img src="./Reference/functional-requirement-screens/06-subject-units.png" alt="06 Unit catalog" width="58"> <img src="./Reference/functional-requirement-screens/07-ordered-lesson-path.png" alt="07 Lesson path" width="58"> |
| FR-08 | The reader renders canonical content, reviewed highlights, figures, and source context; it is not a raw PDF viewer. | Must | Each response carries `id`, `content_version`, and `updated_at`; unpublished internal IDs are absent. | Workspace API and content platform | <img src="./Reference/functional-requirement-screens/07-ordered-lesson-path.png" alt="07 Learning path checkpoint" width="72"> |
| FR-09 | Learners can mark a topic complete and receive the next canonical topic. | Must | Completion is idempotent, monotonic, and safely retried after an interruption. | Completion API | <img src="./Reference/functional-requirement-screens/07-ordered-lesson-path.png" alt="07 Ordered lesson progress" width="72"> |
| FR-10 | Recently opened lessons remain readable offline with a visible cache timestamp. | Must | Cached reading is versioned and safely revalidated when connectivity returns. | Local cache and ETag/version support | No visual reference — original offline state required |
| FR-11 | Prerequisite gates are used only when pedagogically necessary. | Should | A gate explains the requirement and never obscures promised core NCERT reading. | Product/content rule | <img src="./Reference/functional-requirement-screens/08-lesson-lock-gate.png" alt="08 Lesson gate" width="72"> |

### 3.4 Prelims practice and revision

| ID | Requirement | Priority | Acceptance criteria | Dependency | Reference screenshots |
|---|---|---|---|---|---|
| FR-12 | A learner can start topic-linked BPSC MCQ/PYQ practice from a completed or active learning path. | Must | Session creation records stage `PRE`, selected topic, question source, and deterministic session ID. | Assessment session API | <img src="./Reference/functional-requirement-screens/09-prelims-home.png" alt="09 Prelims workspace" width="58"> <img src="./Reference/functional-requirement-screens/10-practice-hub.png" alt="10 Practice hub" width="58"> <img src="./Reference/functional-requirement-screens/11-practice-configuration.png" alt="11 Practice configuration" width="58"> |
| FR-13 | Practice configuration supports subject, topic, question count, PYQ window, and attempt-history filters. | Must | Empty filters explain that no questions match; they do not silently substitute unrelated items. | Assessment catalog | <img src="./Reference/functional-requirement-screens/11-practice-configuration.png" alt="11 Practice filters" width="72"> <img src="./Reference/functional-requirement-screens/17-pyq-filter.png" alt="17 PYQ filters" width="72"> |
| FR-14 | Practice mode provides immediate explanation; test simulation uses the currently approved BPSC marking scheme. | Must | Marking rules carry source/version metadata and are displayed before a timed test. | Assessment/content | <img src="./Reference/functional-requirement-screens/12-practice-test-modes.png" alt="12 Practice and test modes" width="72"> <img src="./Reference/functional-requirement-screens/13-marking-scheme.png" alt="13 Marking scheme" width="72"> |
| FR-15 | One-question-at-a-time attempts preserve selected answer and session state through app restart or network loss. | Must | Submitted answers cannot be duplicated; post-submit answers are immutable. | Device queue and answer API | <img src="./Reference/functional-requirement-screens/14-question-generation.png" alt="14 Question preparation" width="58"> <img src="./Reference/functional-requirement-screens/15-mcq-question.png" alt="15 MCQ attempt" width="58"> <img src="./Reference/functional-requirement-screens/16-exit-practice-confirmation.png" alt="16 Exit confirmation" width="58"> |
| FR-16 | Results show accuracy, mistakes, and at least one next revision action. | Must | Recommendations are explainable from attempt evidence and validated PYQ/topic priority. | Results and revision APIs | <img src="./Reference/functional-requirement-screens/18-overall-progress.png" alt="18 Overall progress" width="58"> <img src="./Reference/functional-requirement-screens/19-subject-performance.png" alt="19 Subject performance" width="58"> <img src="./Reference/functional-requirement-screens/21-revision-areas.png" alt="21 Revision areas" width="58"> |
| FR-17 | Overall and subject-level progress distinguish Prelims from Mains data. | Should | No aggregate accuracy merges stages without an explicit learner choice. | Progress service | <img src="./Reference/functional-requirement-screens/18-overall-progress.png" alt="18 Overall progress" width="58"> <img src="./Reference/functional-requirement-screens/19-subject-performance.png" alt="19 Subject progress" width="58"> <img src="./Reference/functional-requirement-screens/20-progress-comparison.png" alt="20 Progress comparison" width="58"> |

### 3.5 Conditional Mains submission

| ID | Requirement | Priority | Acceptance criteria | Dependency | Reference screenshots |
|---|---|---|---|---|---|
| FR-18 | Learners can browse BPSC Mains GS-I questions by subject, topic, year, marks, and pattern. | Conditional | All questions show `MAINS_GS1` stage and source/provenance. | Assessment/content | <img src="./Reference/functional-requirement-screens/22-mains-pyq-list.png" alt="22 Mains PYQ list" width="72"> |
| FR-19 | Learners can write, save, resume, and submit a structured answer. | Conditional | Offline drafts are clearly marked as local; newer remote drafts create a conflict prompt before overwrite. | Mains draft/submit APIs | <img src="./Reference/functional-requirement-screens/23-daily-mains-question.png" alt="23 Daily Mains question" width="72"> |
| FR-20 | Evaluation shows status only until the evaluation policy, rubric, source citations, and review model are approved. | Fast follow | The app never presents unreviewed feedback as authoritative. | Evaluation service | No visual reference — original evaluation/status states required |

### 3.6 Deferred News and engagement

Current affairs, shorts, rankings, subscriptions, and notifications are not release blockers. When introduced, each current-affairs claim must be reviewed, source-linked, BPSC-tagged, and connected to an eligible practice set.

## 4. Content and non-functional requirements

### Content minimum bar

| Area | Production minimum |
|---|---|
| Learn catalog | A curated BPSC-relevant subject/unit/topic path with stable IDs and published reading |
| Prelims practice | Topic-linked questions with BPSC stage, answer key, explanation provenance, and source traceability |
| Revision | Enough tagged question/attempt data to return a meaningful next action; otherwise state that more attempts are needed |
| Mains, if enabled | A reviewed GS-I question bank with stage, marks, year/pattern labels, word guidance, and draft/submit policy |
| Rights and provenance | Content Ops confirms permitted use; external PDFs in `Reference/` remain internal licensing/review material unless explicitly cleared |

### Reliability and offline

| Scenario | Production requirement |
|---|---|
| Opened lesson offline | Render a versioned cache and show its offline timestamp |
| Completion offline | Queue an idempotent mutation and reconcile without double completion |
| MCQ interruption | Persist selected option and session state; prevent duplicate submission |
| Mains draft interruption | Preserve encrypted/local draft; show unsynced state and resolve revisions safely |
| Stale content | Revalidate with ETag/version data before treating cache as current |
| API failure | Provide a screen-specific retry/fallback and a request ID for support |

### Accessibility and compatibility

- iOS and Android navigation must work without web-view dependence.
- Support light/dark mode, dynamic text, screen readers, external keyboard focus, and 44 × 44 point touch targets.
- Use a Devanagari-capable fallback when Hindi is enabled.
- Do not use color alone to communicate correct/incorrect answers, weakness, or submission status.

### Content, privacy, and compliance

- Include source/citation and content version when an explanation, model answer, tip, or factual claim appears.
- Do not expose raw ingestion blocks, model prompts, unpublished intelligence, raw answer text, tokens, or personal data in analytics.
- Enforce authorization server-side; a mobile UI lock is never an entitlement control.
- Maintain original product language, brand, iconography, and layouts.
- Complete content-rights and app-store copy review before external beta.

## 5. Measurement and instrumentation

### Core funnel

| Event | Required properties | Use |
|---|---|---|
| `app_opened` | app version, authenticated state | Diagnose activation and session restore |
| `home_next_action_seen` | action type, content version | Measure Home relevance |
| `lesson_opened` | book/chapter/topic IDs, source version | Measure activation and content reach |
| `lesson_completed` | topic ID, completion source, queued/synced state | Measure Learn loop and sync health |
| `practice_started` | stage, topic ID, mode, question count | Measure read-to-practice conversion |
| `practice_submitted` | session ID, stage, score band, retry count | Measure assessment health without raw responses |
| `revision_action_seen` | reason code, topic ID | Measure recommendation coverage |
| `mains_draft_saved` / `mains_submitted` | question ID, sync state | Measure conditional Mains adoption |

The product team reviews activation, lesson completion, practice conversion, accuracy lift, and D7 targets after an agreed observation period. These metrics are decision inputs, not guaranteed outcomes; delivery gates and release thresholds are maintained in the [Delivery Plan](./02-delivery-plan.md).

## 6. Document boundaries and references

| Topic | Owning document |
|---|---|
| Functional scope, requirements, acceptance criteria, content/non-functional requirements, and metrics | This document |
| Endpoint readiness, response contracts, API ownership, and sync protocol | [Mobile Backend Contract](./03-mobile-backend-contract.md) |
| Milestones, dependency gates, RACI, go/no-go, rollout, and post-launch sequencing | [Delivery Plan](./02-delivery-plan.md) |
| Architecture decisions and client implementation boundaries | [Expo UI Architecture](./04-expo-ui-architecture.md) |

- [Mobile Product Brief](./00-mobile-product-brief.md)
- [Expo UI Architecture](./04-expo-ui-architecture.md)
- [Mobile Backend Contract](./03-mobile-backend-contract.md)
- [Delivery Plan](./02-delivery-plan.md)
- [Reference App Flow](./Reference/REFERENCE-APP-FLOW.md)
- [Student APIs](../backend/04-student-apis.md)
- [AI Platform Guide](../ai/AI-PLATFORM-GUIDE.md)
