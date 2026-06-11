---
title: Vooster reproduction prompt guardrail
created_at: 2026-06-11T09:20:41Z
resolved: true
priority: P1
kind: analysis
related:
  - _overlay/analyses/2026-06-11-vooster-origin-method-reconstruction.md
  - docs/ideation.md
  - docs/00-overview.md
  - docs/05-data-model.md
---

# Vooster reproduction prompt guardrail

## TL;DR

When guiding a from-zero Vooster exercise, do not emit the next prompt just
because it sounds useful. First pass it through this gate:

```text
baseline -> purpose -> three product axes -> drift check -> irreversible-risk check
```

The guardrail exists because the previous "compress the entity list" prompt
optimized for a smaller MVP and pushed the exercise away from the shape that the
Vooster origin artifacts suggest.

## Operating Rule

Before writing any next prompt for `vooster-from-zero`, the assistant must do
this short review:

1. Name the current baseline artifact or conversation state.
2. Name the purpose of the next prompt in one sentence.
3. Check the three product axes.
4. Check whether the prompt is redesigning, compressing, or optimizing beyond
   the stated step.
5. Check whether the prompt can introduce hard-to-remove context drift.

If any check fails, do not provide the prompt. Revise it and run the gate again.

## The Three Product Axes

Every domain-modeling prompt must preserve all three axes unless the user
explicitly narrows the task.

### 1. Cockburn Method Axis

The prompt must keep the method artifacts visible:

- `Actor`
- `Goal`
- `UseCase`
- `Scenario`
- `Step`
- `Stakeholder`
- `StakeholderInterest`
- `Revision`

Do not prematurely collapse `Scenario` and `Step` into opaque `UseCase` fields
when the step asks for an agent-operable domain model.

### 2. SaaS Collaboration Axis

The prompt must keep the tenant and collaboration skeleton visible:

- `Workspace`
- `User`
- `Membership`
- `Project`
- `Comment`

Do not treat `Membership` as "advanced permissions" during MVP modeling. Fine
grained roles can be deferred, but basic workspace membership is part of the
minimum SaaS collaboration shape.

### 3. Parallel Agent Workflow Axis

The prompt must keep agent-safe work coordination visible:

- `WorkSession`
- `SpecBranch`
- `MergeRequest`
- `Lock`
- `Revision`

Do not replace these with generic names such as `Session` and `Version` unless
the current step explicitly asks for a fresh simplified product, not a
Vooster-like first model.

## Drift Checks

Reject or revise a prompt if it contains unscoped pressure like:

- "as small as possible"
- "aggressively reduce"
- "collapse anything that can be a field"
- "make a better MVP"
- "simplify the model" without naming which axis must remain intact

These phrases are not always wrong, but they must be paired with explicit
axis-preservation rules.

## Irreversible-Risk Check

Conversation context is sticky. A prompt can be risky even if no file changes
are made.

Before emitting a prompt, ask:

- If this answer goes wrong, can we ignore it cleanly?
- Would this inject misleading terms into the active thread?
- Should this be run in a forked/restored session instead?
- Is the user asking for comparison against Vooster origin artifacts or for a
  new independent design?

If the answer is uncertain, state the risk before giving the prompt.

## Prompt Gate Template

Use this private checklist before sending a next prompt:

```text
Baseline:
- What was the last accepted answer?
- Which Vooster files or prior answers define the comparison target?

Purpose:
- What should the next answer produce?
- Is this step exploration, narrowing, comparison, or implementation?

Axis check:
- Cockburn method axis preserved? yes/no
- SaaS collaboration axis preserved? yes/no
- Parallel agent workflow axis preserved? yes/no

Drift check:
- Does the prompt redesign, compress, or optimize beyond the stated purpose?
- Does it introduce loaded terms that steer the model away from the current
  exercise?

Risk check:
- Is the prompt safe to send in the active thread?
- If not, use a fork/restored session or ask before sending.
```

Only the final prompt should be shown to the user unless the user asks for the
gate details.

## Self-Test Loop

### Test 1: Previous Bad Prompt

Input candidate:

```text
좋다. 방금 도출한 엔티티는 사고 재료로는 좋지만 MVP로는 너무 많고 복잡하다.
Vooster의 첫 MVP를 만든다고 가정하고, 엔티티를 과감하게 줄여줘.
엔티티 수를 가능한 한 줄인다.
별도 엔티티가 아니라 UseCase의 필드로 충분한 것은 필드로 내려라.
```

Gate result: FAIL.

Reason:

- Drift check fails: the prompt strongly optimizes for compression.
- Cockburn method axis is at risk: `Scenario`, `Step`, and `Goal` can collapse
  into `UseCase`.
- SaaS collaboration axis is not protected: `Membership` can be deferred.
- Parallel agent workflow axis is at risk: `SpecBranch`, `MergeRequest`, and
  `Revision` can collapse into generic versioning.

Revision required before sending.

### Test 2: Corrected Domain Model Prompt

Input candidate:

```text
좋다. 방금 도출한 엔티티 목록은 사고 재료로 삼겠다.

이제 Vooster의 첫 MVP 도메인 모델을 정리하자.
목표는 더 작은 모델로 억지로 압축하는 것이 아니라,
Cockburn 개념, SaaS 협업/권한 구조, 병렬 코딩 에이전트 워크플로우를
명시적으로 다룰 수 있는 충분히 선명한 초기 모델을 만드는 것이다.

엔티티 수를 최소화하려고 하지 마라.
에이전트가 특정 유스케이스, 시나리오, 스텝, 리비전, 세션, 브랜치, 락을
직접 다뤄야 한다면 별도 엔티티로 유지해라.

특히 Workspace, User, Membership, Project는 SaaS 기본 골격으로 검토해라.
```

Gate result: PASS.

Reason:

- Cockburn method axis is protected.
- SaaS collaboration axis is protected, including `Membership`.
- Parallel agent workflow axis is protected.
- The prompt narrows the model without asking for aggressive compression.

### Test 3: Over-Correction Toward Original Copying

Input candidate:

```text
원본 Vooster와 최대한 똑같은 엔티티 목록을 만들어라.
원본과 다르면 실패다.
```

Gate result: FAIL.

Reason:

- The user clarified that the exercise should proceed as a new from-zero
  product, not use the word "restore" or force exact copying.
- This would turn the exercise into answer matching instead of disciplined
  product design.

Revision required before sending.

### Test 4: Next Safe Prompt Shape

Input candidate:

```text
좋다. 방금 정리한 첫 MVP 엔티티 목록을 기준으로, 각 엔티티의 핵심 필드와 관계를
정리하자.

목표는 구현을 시작하는 것이 아니라, 이후 유스케이스와 데이터 모델 문서로 옮길 수
있는 수준의 도메인 구조를 세우는 것이다.

반드시 세 축을 모두 유지해라.

1. Cockburn 방법론 artifact
2. SaaS 협업/권한/테넌시 구조
3. 병렬 코딩 에이전트 워크플로우
```

Gate result: PASS.

Reason:

- It advances one step from entity list to fields/relationships.
- It explicitly preserves all three axes.
- It avoids implementation and avoids exact-copy language.

### Test 5: Membership Omission

Input candidate:

```text
첫 MVP 엔티티 목록을 정리하자.
Cockburn 개념과 병렬 코딩 에이전트 워크플로우를 명시적으로 다뤄라.
특히 Goal, Scenario, Step, Revision, WorkSession, SpecBranch, MergeRequest,
Lock, Comment를 검토해라.
```

Gate result: FAIL.

Reason:

- The prompt names the Cockburn and parallel-agent axes but omits the SaaS
  collaboration axis.
- `Workspace`, `User`, `Membership`, and `Project` are not protected.
- This repeats the failure mode where `Membership` is treated as optional
  "advanced permissions" instead of MVP SaaS structure.

Revision required before sending.

## Completion Signal

The guardrail is ready when it catches:

1. The previous compression prompt.
2. A prompt that over-corrects into exact original copying.
3. A prompt that forgets the SaaS membership axis.

It must pass:

1. A domain-model prompt that preserves all three axes.
2. A next-step field/relationship prompt that preserves all three axes.

## Verification Run

2026-06-11:

1. Initial draft caught the compression prompt and exact-copy prompt, but its
   completion signal mentioned `Membership` omission without a dedicated test.
2. The guardrail was revised to add Test 5, `Membership Omission`.
3. Mechanical check confirmed three `FAIL` cases and two `PASS` cases.
4. `git diff --check` reported no whitespace errors.

Current status: ready to use for the next `vooster-from-zero` prompt.
