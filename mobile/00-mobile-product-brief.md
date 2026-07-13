# 00 — Expo Mobile Product Brief

| Field | Value |
|---|---|
| Status | Planning |
| Product | SarkariExamsAI mobile |
| Platform | Expo / React Native, iOS and Android |
| Primary exam | BPSC Prelims + BPSC Mains GS-I |
| Scope | Separate mobile client; shared knowledge platform |

## Product decision

Build an original BPSC preparation app around a simple daily loop:

```mermaid
flowchart LR
    Plan["See today's plan"] --> Learn["Learn one topic"]
    Learn --> Practice["Attempt linked practice"]
    Practice --> Review["Review mistakes"]
    Review --> Revise["Schedule revision"]
    Revise --> Plan
```

The reference screenshots show useful mobile patterns: short, ordered lessons; clear subject/unit grouping; separate Prelims and Mains destinations; daily questions; current-affairs feeds; visible progress; and compact performance feedback. SarkariExamsAI should use these patterns, but it must not reproduce another product’s brand, copy, content, imagery, paywall flow, or exact visual layout.

## User and problem

**Primary user:** a serious BPSC aspirant who studies predominantly on a phone, has limited time, and needs a reliable path from reading to revision.

| Student problem | Mobile response |
|---|---|
| Does not know what to study today | One daily plan with one primary next action |
| Loses track of the syllabus | Subject → unit → ordered lesson path |
| Reads but cannot convert it into marks | Topic-linked Prelims practice and Mains prompts |
| Treats Prelims and Mains as unrelated | Dedicated modes that share the same BPSC concepts |
| Forgets completed material | Revision queue based on completion and practice performance |
| Needs current-affairs context | Curated source-linked feed with BPSC tags |

## Value proposition

SarkariExamsAI is not a video catalogue or generic chatbot. Its mobile differentiator is:

> Canonical, source-backed study material connected to BPSC PYQs, stage-specific practice, and a next-best study action.

The existing product principle remains: **One Goal → One Screen → One Decision.**

## Mobile information architecture

The primary navigation has five destinations:

| Destination | Student goal | MVP |
|---|---|---|
| Home | Resume learning and see today’s plan | Yes |
| Learn | Navigate subjects, units, lessons, and reading | Yes |
| Prelims / Practice | Take MCQs, PYQs, and revise weak areas | Yes |
| Mains | Browse/attempt BPSC Mains questions and view feedback | Yes, initially limited |
| News | Read curated current affairs and make it practice-ready | Later MVP increment |

## MVP boundaries

### Include

- BPSC exam selection and profile
- Course catalog, units, lessons, canonical reading, completion state
- Topic-linked BPSC Prelims MCQ/PYQ sessions
- BPSC Mains GS-I question bank and structured answer submission
- Revision areas based on completed lessons and answered questions
- Offline reading cache for already opened lessons

### Exclude from the first Expo release

- Live classes, creator/video marketplace, or copied coaching-course model
- Real-time generic chat
- Unreviewed AI explanations, tips, or current-affairs claims
- Social leaderboard as a primary loop
- Subscription payments until entitlement and content access rules are approved

## Business and quality outcomes

| Metric | Initial target | Reason |
|---|---:|---|
| First lesson opened after install | under 3 minutes | Validates onboarding and catalog clarity |
| Weekly learner completes a lesson | 40%+ | Tests guided learning loop |
| Practice attempted after lesson | 30%+ | Validates read → practice connection |
| Accuracy lift after related reading | +15% | Validates knowledge intelligence value |
| D7 retention | 30%+ | Tests daily planning and revision habit |
| Mains answer submitted by eligible users | tracked by cohort | Tests Mains workflow before scaling evaluation |

## Content and monetisation guardrails

- NCERT and licensed/reference content must retain provenance and source-level access rules.
- PYQs must be traceable to their official/licensed source.
- Tutor notes, model answers, and evaluations require review and provenance before publication.
- Premium access must not make the free reading path misleading or block a student from material already promised as available.
- Any paid unlock, trial, or target-setting message must be original SarkariExamsAI copy and comply with app-store policies.

## Success criteria for implementation handoff

The Expo team can start implementation when:

1. A BPSC learner can move from Home to an ordered lesson to practice without dead ends.
2. Every student-facing fact and PYQ continues to reference the knowledge platform’s canonical/exam-intelligence source.
3. Prelims and Mains remain separate experiences but share topic and progress identifiers.
4. The backend contract in [02-mobile-backend-contract.md](./02-mobile-backend-contract.md) has an owner and delivery phase for every blocking endpoint.
