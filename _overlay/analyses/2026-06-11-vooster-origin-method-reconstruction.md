---
title: Vooster origin method reconstruction
created_at: 2026-06-11T07:19:56Z
resolved: true
priority: P2
kind: analysis
related:
  - docs/ideation.md
  - docs/00-overview.md
  - docs/05-data-model.md
  - docs/usecases/_index.md
  - AGENTS.md
  - prompts/goal-docs-generate.md
---

# Vooster origin method reconstruction

## TL;DR

Vooster does not appear to have started as "build an app" prompting. The
repository evidence points to a sequence of methodology study, domain modeling,
multi-agent workflow pressure, MVP scoping, use-case extraction, harness design,
and then TDD implementation.

The recoverable method is:

```text
methodology -> domain entities -> agent-concurrency problem -> MVP dogfood
  -> fully dressed use cases -> local agent harness -> red/green/refactor code
```

This file records that reconstruction so a new from-zero clone can imitate the
process without copying the existing codebase too early.

## Evidence Boundary

The repository can prove files, commits, and written prompts. It cannot prove
the original developer's private Claude Code memory, global settings, plugins,
model version, or unrecorded conversation context.

So this is a reconstruction from durable artifacts, not a claim about every
hidden step the developer took.

## Direct Evidence

`docs/ideation.md` says it is a record of a conversation between the user and
Claude for developing the Vooster Spec product idea (`docs/ideation.md:3`).

The first recorded prompt asks for Alistair Cockburn's use-case definition
technique (`docs/ideation.md:7`). The next recorded product prompt asks for
domain entities for a SaaS specialized in managing Cockburn-style use cases,
available through CLI and API (`docs/ideation.md:122`).

The initial product overview defines vspec as a SaaS for managing software
specifications using Cockburn's methodology, designed for humans collaborating
with multiple parallel AI coding agents (`docs/00-overview.md:5`). It explicitly
names 6+ concurrent coding-agent sessions as the pressure that existing tools do
not handle well (`docs/00-overview.md:18`).

The initial data model maps Cockburn concepts into first-class entities:
`UseCase`, `Actor`, `Scenario`, `Step`, `Stakeholder`, `StakeholderInterest`,
`Goal`, and `Revision` (`docs/05-data-model.md:16`). It then adds concurrency
entities such as `WorkSession`, `SpecBranch`, `MergeRequest`, and `Lock`
(`docs/05-data-model.md:20`).

The initial use-case index contains 35 use cases across auth, project setup,
use-case authoring, concurrency, collaboration, output, and AI-agent-specific
flows (`docs/usecases/_index.md:3`). The AI-agent category is explicit:
`UC-033-ai-learn`, `UC-034-ai-fetch-spec`, and
`UC-035-ai-propose-change` (`docs/usecases/_index.md:67`).

The working protocol then converts the product idea into an implementation
discipline. `AGENTS.md` identifies the agent as a Kent Beck-style test-first
engineer (`AGENTS.md:5`), sets a beta-release target around use-case generation,
CLI integration, and web viewer operation (`AGENTS.md:16`), requires one use
case per iteration (`AGENTS.md:31`), and fixes the stack (`AGENTS.md:44`).

The historical initial `GOAL.md` in commit `826f602` required every iteration to
start with `bash scripts/diagnose.sh`, read `AGENTS.md`, `docs/state/next-task.md`,
`docs/state/blockers.md`, and the current use case, then follow a RED, GREEN,
REFACTOR loop verified by `bash scripts/verify-tdd.sh`.

## Reconstructed Starting Order

### 1. Learn the governing method first

The first durable move was not implementation. It was asking how Cockburn-style
use cases work.

The hidden skill here is selecting a method that can become product structure.
Cockburn is not just background reading in this repo. It becomes the language of
the product, the data model, the use-case files, and the agent's completion
conditions.

### 2. Convert the method into domain entities

The next move was to ask what entities a SaaS would need if it specialized in
that method and exposed CLI/API surfaces.

This is the key design jump. The developer did not ask for pages, tables, or
routes first. They asked for the domain nouns and relationships implied by the
method.

### 3. Insert the developer's own pain as product pressure

Vooster's distinctive shape comes from the multi-agent constraint: humans use
many coding-agent sessions at once, so specs must be pinnable, mergeable,
lockable, and safe to read while work is in flight.

That pressure creates the product's differentiators:

- immutable revision snapshots per work session
- spec branches and merge requests
- semantic locks
- impact analysis
- agent-readable CLI/API responses
- local markdown sync

### 4. Define MVP by dogfooding

The MVP was not "has auth and CRUD." The stronger condition was: vspec should
manage its own use cases and development cycle.

That dogfood criterion forced the product to become useful to the developer's
actual workflow, not only to a hypothetical buyer.

### 5. Extract use cases before coding

The initial repo had a use-case inventory before normal implementation began.
This turns the backlog into behavior contracts. It also gives coding agents a
stable unit of work: one use case per iteration.

### 6. Build a repository-local harness

The harness is ordinary files plus scripts, but the composition is unusual:

- `AGENTS.md` defines the agent identity and rules.
- `docs/state/*` keeps mutable work state outside human memory.
- `scripts/diagnose.sh` tells the agent what is true now.
- `scripts/next-task.sh` chooses the next action.
- `scripts/verify-tdd.sh` checks the red/green/refactor discipline.
- `scripts/completion-check.sh` decides whether the mission is done.
- goal documents later become repeatable work contracts.

The important move is that the developer externalized their workflow into the
repo before expecting agents to operate reliably.

### 7. Implement with small TDD commits

The early commit history follows explicit `red:`, `green:`, and `refactor:`
messages. That suggests implementation began only after the product language,
use cases, and harness were already strong enough to constrain the agent.

## What To Imitate In `vooster-from-zero`

Do not start by copying source files.

Start by imitating the sequence:

1. Ask for the governing method.
2. Ask how that method becomes product/domain entities.
3. Add the real workflow pressure: multiple coding agents, local files, CLI/API,
   pinned context, and safe parallelism.
4. Define MVP by dogfooding.
5. Generate use cases.
6. Generate the repo-local agent protocol and verification loop.
7. Only then scaffold code.

This preserves the valuable part of the original process: the product grows from
a method and a workflow problem, not from a UI mockup or CRUD schema.

## First Prompt For The New Clone

Use this as the first instruction in `/home/ubuntu/workspace/personal/vooster-from-zero`:

```text
나는 지금부터 Vooster를 바닥부터 다시 재현하려고 한다.
기존 저장소의 코드를 복사하지 않고, 개발자가 처음 접근했을 법한 순서만 따라가고 싶다.

바로 코딩하지 마라.
먼저 제품의 기반이 될 방법론과 도메인 사고를 세워야 한다.

출발점은 Alistair Cockburn의 유스케이스 작성법이다.
이 방법론을 다음 관점에서 정리해줘.

1. 유스케이스를 기능 목록이 아니라 액터의 목표와 이해관계자의 이익으로 보는 이유
2. scope, level, primary actor, stakeholders/interests, preconditions, guarantees,
   main success scenario, extensions의 의미
3. brief, casual, fully dressed 형식의 차이
4. SaaS 제품의 데이터 모델로 옮길 때 반드시 보존해야 하는 개념
5. 코딩 에이전트가 이 방법론을 사용해 개발할 때 특히 중요한 규칙

응답은 단순 설명으로 끝내지 말고, 다음 단계에서 도메인 엔티티를 뽑을 수 있도록
구조화해줘.
```

## Second Prompt To Use After That

After receiving the method summary, continue with:

```text
좋다. 이제 이 Cockburn 방식으로 유스케이스를 관리하는 SaaS를 만들고 싶다.
이 SaaS는 web UI뿐 아니라 CLI와 REST API로도 사용할 수 있어야 한다.

또한 핵심 사용자는 한 명의 개발자/PM이 여러 개의 코딩 에이전트를 병렬로 실행하는
환경이다. 최소 6개의 에이전트 세션이 동시에 서로 다른 작업을 수행할 수 있다.

이 조건에서 필요한 도메인 엔티티를 도출해줘.

주의할 점:

- 화면이나 라우트부터 설계하지 말고 도메인 엔티티와 관계부터 설계한다.
- Cockburn의 원래 개념을 어떤 엔티티로 보존할지 설명한다.
- 병렬 AI 에이전트 때문에 추가로 필요한 세션, 스냅샷, 브랜치, 락, 충돌, 영향 분석
  개념을 포함한다.
- MVP에 넣을 것과 나중으로 미룰 것을 분리한다.
- 마지막에는 다음 단계에서 유스케이스 목록을 뽑기 위한 질문을 제안한다.
```

## Guardrail

If the agent starts coding before method, entity, MVP, and use-case documents
exist, it is no longer reproducing the Vooster origin method. Stop and redirect
it back to product-method reconstruction.

For any follow-up prompt in the `vooster-from-zero` exercise, first run the
prompt through `_overlay/prompts/2026-06-11-vooster-reproduction-prompt-guardrail.md`.
That gate exists to prevent context drift such as over-compressing the domain
model, forgetting the SaaS membership axis, or over-correcting into exact-copy
answer matching.
