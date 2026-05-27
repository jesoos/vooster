# Vooster 최초 커밋 재분석

날짜: 2026-05-27
대상 커밋: `826f6023368c55e715c39f86b20cbee64cf30afd`
커밋 시각: 2026-05-18T17:47:42+09:00
커밋 제목: `setup: bootstrap autonomous-build harness for vspec MVP`

## 결론

Vooster 최초 커밋은 실행 가능한 앱 스캐폴딩이 아닙니다. 오히려 다음 실행에서
에이전트가 앱 스캐폴딩을 시작하도록 만드는 **제품 명세 + 에이전트 운영 규칙 +
상태 파일 + 검증 스크립트 패킷**입니다.

가장 중요한 재해석은 이것입니다.

- 최초 커밋에는 `package.json`이 없습니다.
- 최초 커밋에는 `src/`, `apps/`, `packages/`가 없습니다.
- 최초 커밋에는 `goals/`, `cycles/`, `docs/findings/`도 없습니다.
- 대신 `GOAL.md`, [AGENTS.md](../../AGENTS.md), `docs/00..09`,
  `docs/usecases/UC-001..UC-035`, `docs/state/*`, `scripts/*`가 있습니다.

즉 이 커밋은 “앱의 첫 코드”가 아니라 “앱을 만들 에이전트 루프의 첫 입력
상태”입니다.

## 커밋 본문이 직접 말하는 목적

커밋 본문은 이 파일 묶음의 목적을 다음처럼 설명합니다.

```text
Files for codex 'goal' loop to build vspec MVP via TDD
```

또한 주요 구성으로 다음을 명시합니다.

- `GOAL.md`, `AGENTS.md`, `README.md`: mission and protocol
- `docs/00..09`: 제품/기술/부트스트랩 명세
- `docs/usecases/UC-001..UC-035`: 35개 fully-dressed Cockburn use cases
- `docs/state/`: agent-managed scratch space
- `prisma/schema.prisma`: 16-entity Postgres schema seed
- `scripts/`: diagnose, next-task, verify-tdd, completion-check 등
- `tests/e2e/_template.test.ts`: E2E pattern

커밋 메시지 자체가 이 커밋을 “앱 구현”이 아니라 “Codex goal loop를 위한
하네스”라고 정의합니다.

## 파일 구성

최초 커밋은 74개 파일, 11,419줄을 추가했습니다.

주요 줄 수:

```text
docs/ideation.md                         4030
prisma/schema.prisma                      407
docs/06-api-contract.md                   386
docs/07-cli-spec.md                       345
docs/05-data-model.md                     332
docs/09-bootstrap.md                      257
AGENTS.md                                 251
docs/08-file-format.md                    212
docs/01-architecture.md                   165
docs/04-tdd-protocol.md                   159
docs/03-cockburn-method.md                154
scripts/next-task.sh                      131
docs/02-tech-stack.md                     122
scripts/completion-check.sh               104
docs/00-overview.md                       100
GOAL.md                                    98
```

이 분포는 구현 코드보다 문서와 하네스가 중심임을 보여줍니다.

## 최초 커밋에 있었던 것

### 1. 제품 구상 원문

`docs/ideation.md`는 4,030줄입니다. 단순 요약이 아니라 사용자와 Claude가 나눈
제품 구상 대화를 저장한 것으로 보입니다.

이 파일은 다음 산출물의 원천 역할을 합니다.

- Cockburn 유스케이스 방법론 정리
- 제품 문제 정의
- 도메인 엔티티
- MVP 범위
- Codex `goal` 루프 설계
- `GOAL.md`, `AGENTS.md`, `docs/*`, `scripts/*` 구성 아이디어

### 2. Codex goal 입력 파일

최초 커밋의 `GOAL.md`는 Codex `goal`에 넣을 미션 프롬프트입니다.

핵심 완료 조건:

1. 모든 [docs/usecases/](../../docs/usecases/)의 `UC-*.md`에 대응하는 passing E2E
   테스트가 있어야 함
2. lint, type, coverage, no-bypass, mutation sample 등 품질 gate 통과
3. [scripts/dogfood-test.sh](../../scripts/dogfood-test.sh) 통과
4. [scripts/completion-check.sh](../../scripts/completion-check.sh) exit 0

중요한 운영 지시:

- 매 iteration의 첫 명령은 `bash scripts/diagnose.sh`
- 읽기 순서는 `AGENTS.md` → `docs/state/next-task.md` →
  `docs/state/blockers.md` → 현재 유스케이스
- RED/GREEN/REFACTOR마다 별도 커밋
- 3 TDD cycle 동안 진전이 없으면 blocker 기록 후 다른 task로 이동

### 3. 에이전트 작업 헌법

최초 [AGENTS.md](../../AGENTS.md)는 251줄입니다. 현재 파일과는 이후 변경이
있지만, 최초부터 다음 성격을 갖고 있었습니다.

- Kent Beck식 TDD 정체성 부여
- 작은 커밋
- 한 iteration에 한 use case
- 생산 코드 전 실패 테스트
- 기술 스택 고정
- 파일/함수 크기 제한
- 테스트 우회 금지

즉 최초 커밋은 에이전트를 단순 코드 생성기가 아니라 “정해진 개발 프로토콜을
따르는 작업자”로 설정합니다.

### 4. 제품과 구현 계약 문서

`docs/00..09`는 에이전트가 앱을 만들 때 참조할 계약입니다.

- `docs/00-overview.md`: 제품 정의와 성공 조건
- `docs/01-architecture.md`: hexagonal architecture, revisions, sessions,
  branches, locks, impact analysis
- `docs/02-tech-stack.md`: TypeScript, Node, Vitest, Fastify, Prisma, oclif
- `docs/03-cockburn-method.md`: 유스케이스 작성법
- `docs/04-tdd-protocol.md`: TDD와 검증 규칙
- `docs/05-data-model.md`: 16개 도메인 엔티티
- `docs/06-api-contract.md`: REST API 계약
- `docs/07-cli-spec.md`: CLI 계약
- `docs/08-file-format.md`: 로컬 markdown/spec 포맷
- `docs/09-bootstrap.md`: 첫 iteration에서 만들 scaffold 계약

이 문서들이 먼저 있었기 때문에 에이전트가 이후 구현 판단을 할 수 있었습니다.

### 5. 유스케이스 코퍼스

최초 커밋은 35개 fully-dressed use case를 모두 포함합니다.

```text
docs/usecases/UC-001-signup.md
...
docs/usecases/UC-035-ai-propose-change.md
```

이것은 구현 backlog이자 acceptance source입니다. [scripts/next-task.sh](../../scripts/next-task.sh)와
[scripts/completion-check.sh](../../scripts/completion-check.sh)는 이 문서들을
순회하며 다음 작업과 완료 조건을 정합니다.

### 6. 상태 파일

최초 커밋은 [docs/state/](../../docs/state/)를 포함합니다.

- `blockers.md`
- `learnings.md`
- `next-task.md`
- `progress.md`
- `test-plan.md`

초기 `next-task.md`는 스캐폴딩을 지시했습니다.

```text
TASK: Initialize project scaffolding.
```

초기 `progress.md`는 35개 유스케이스가 모두 `NOT STARTED`인 상태를 담았습니다.
이것은 다음 실행이 어디서 시작해야 하는지 명확히 해줍니다.

### 7. 하네스 스크립트

최초 커밋의 스크립트는 현재 FCG식 goal stack 이전의 단순한 하네스입니다.

대표 역할:

- `scripts/diagnose.sh`: git 상태, scaffolding 상태, 테스트 상태, UC 진행률,
  blockers, 다음 작업 출력
- `scripts/next-task.sh`: package.json/tsconfig/vitest/prisma 순서로 bootstrap을
  지시하고, 이후 실패 중인 UC 또는 다음 UC를 선택
- `scripts/completion-check.sh`: 6개 gate로 완료 조건 판정
- `scripts/update-state.sh`: progress와 next-task 재생성
- `scripts/verify-tdd.sh`: RED/GREEN/REFACTOR 커밋 패턴 검증
- `scripts/check-bypass.sh`: `.skip`, `.todo`, tautological assertion 등 우회 방지
- `scripts/dogfood-test.sh`: self-dogfooding 검증

이 시점의 `completion-check.sh`는 아직 현재처럼 [goals/](../../goals/)를
번호순으로 병렬 실행하지 않습니다. 직접 6개 gate를 검사합니다.

## 최초 커밋에 없었던 것

다음은 `826f602` tree에 없었습니다.

```text
absent goals
absent cycles
absent docs/findings
absent guidelines
absent .claude
absent .codex
absent apps
absent packages
absent pnpm-workspace.yaml
absent package.json
```

의미:

- 현재의 FCG 구조는 최초 커밋 이후에 생긴 진화 결과입니다.
- 현재 monorepo 구조도 최초 커밋 이후에 생긴 진화 결과입니다.
- 최초 커밋은 “바로 실행 가능한 Node 프로젝트”가 아닙니다.
- 최초 커밋은 “다음 커밋에서 Node 프로젝트를 만들게 할 agent bootstrap
  package”입니다.

## 최초 루프가 실제로 선택한 첫 작업

최초 `scripts/next-task.sh`는 `package.json`이 없으면 다음 작업을 출력합니다.

```text
TASK: Initialize project scaffolding.
  - npm init -y
  - Install dev deps: typescript tsx vitest @types/node prisma @prisma/client
  - Install runtime deps: fastify zod pino @oclif/core gray-matter
  - Read: docs/02-tech-stack.md (do not deviate)
  - Commit: "setup: initial scaffolding"
```

실제 다음 커밋도 다음과 같습니다.

```text
a1e14f3 2026-05-18T23:42:59+09:00 setup: initial scaffolding
```

따라서 최초 커밋은 의도적으로 `package.json`을 포함하지 않고, 에이전트가 첫
iteration에서 scaffolding을 하도록 남겨둔 것으로 해석할 수 있습니다.

## 최초 completion-check의 의미

최초 `completion-check.sh`는 다음 6개 gate를 직접 검사했습니다.

1. structural: 모든 `docs/usecases/UC-*.md`에 `tests/e2e/<UC-ID>.test.ts`가 있는가
2. functional: 전체 vitest 통과
3. integrity: [scripts/check-bypass.sh](../../scripts/check-bypass.sh) 통과
4. lint/type: `tsc --noEmit`, `eslint .`
5. coverage: `vitest run --coverage`
6. dogfooding: [scripts/dogfood-test.sh](../../scripts/dogfood-test.sh) 통과

현재의 `goals/*.gates.sh` 구조와 비교하면 더 단순합니다. 하지만 핵심 원칙은
이미 있습니다.

- 완료 조건은 사람이 감으로 판정하지 않고 스크립트가 판정
- `docs/usecases/`를 source of truth로 순회
- 테스트 우회 패턴을 별도로 차단
- self-dogfooding을 완료 조건에 포함

## FCG 관점에서 본 최초 커밋

최초 커밋은 FCG가 아닙니다.

없는 것:

- findings 자산
- cycles 자산
- goals 3종 세트
- active goal pointer
- per-goal cache
- `check-gate-rigor.sh`
- `_meta` goal

하지만 FCG의 원형은 있습니다.

- `docs/state/next-task.md`: 다음 작업 힌트
- `scripts/next-task.sh`: 작업 라우터
- `scripts/completion-check.sh`: gate 판정
- `docs/state/blockers.md`: 막힌 작업 기록
- `GOAL.md`: 단일 mission file

따라서 최초 커밋은 **pre-FCG proto-harness**로 보는 것이 정확합니다. 이후
Vooster가 성장하면서 단일 `GOAL.md`와 단일 `completion-check.sh` 구조가
`goals/`, `cycles/`, `docs/findings/`로 분화된 것으로 보입니다.

## 재현 실험에 대한 수정된 권장

새 재현 실험은 현재 FCG 키트를 바로 복사하기보다 최초 커밋 모델을 먼저 따라
하는 편이 좋습니다.

1. `docs/ideation.md` 작성
2. `AGENTS.md` 작성
3. `GOAL.md` 작성
4. `docs/00..09`의 축소판 작성
5. `docs/usecases/` 5~7개 작성
6. `docs/state/*` 초기화
7. `scripts/diagnose.sh`, `scripts/next-task.sh`, `scripts/completion-check.sh`
   작성
8. `package.json`은 의도적으로 만들지 않고, `next-task.sh`가 첫 작업으로
   scaffolding을 출력하게 함
9. Codex goal 또는 새 Codex 세션이 첫 `setup: initial scaffolding` 커밋을 만들게 함

이 방식이 Vooster 최초 커밋의 실제 구조와 더 잘 맞습니다.

## 남는 질문

저장소만으로는 다음을 확정할 수 없습니다.

1. 최초 11,419줄짜리 커밋이 단일 에이전트 실행으로 생성됐는가?
2. 사람이 `docs/ideation.md` 이후 어느 정도 파일을 직접 편집했는가?
3. `GOAL.md`를 실제 Codex `goal`에 넣은 정확한 명령은 무엇인가?
4. `cc-system`의 2026-04 자율주행 하네스 계열이 개념적으로 영향을 줬는가?
5. 최초 커밋 이전에 실패한 초안이나 삭제된 브랜치가 있었는가?
