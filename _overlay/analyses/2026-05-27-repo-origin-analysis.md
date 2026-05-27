# 저장소 생성 과정 분석과 재현 청사진

날짜: 2026-05-27
범위: 이 fork 전용 분석
상태: 더 깊은 분석과 따라 하기 절차 보강

## 핵심 결론

이 저장소를 따라 만들려면 앱 코드를 먼저 보면 안 됩니다. 먼저 복제해야 하는
것은 **AI 에이전트가 반복 실행할 수 있는 개발 운영체제**입니다.

이 저장소는 대략 다음 순서로 만들어진 것으로 보입니다.

1. 사람이 Claude와 제품 아이디어를 대화로 구체화했습니다.
2. 그 대화에서 제품 명세, 유스케이스, 아키텍처, 기술 스택, TDD 규칙을
   문서로 뽑았습니다.
3. 에이전트가 매 iteration마다 읽고 실행할 [AGENTS.md](../../AGENTS.md),
   `GOAL.md`(최초 커밋에는 있었으나 현재 트리에는 없음), [docs/state/](../../docs/state/),
   [scripts/](../../scripts/)를 만들었습니다.
4. 첫 목표는 “35개 유스케이스마다 E2E 테스트가 있고 모두 통과한다”였습니다.
5. 이후에는 [goals/](../../goals/) 아래에 goal 스택을 만들고, 각 goal을
   `.md`, `.gates.sh`, `.next-task.sh` 3개 파일 세트로 관리했습니다.
6. 구현은 사람이 직접 파일을 한 줄씩 쌓은 방식이 아니라, RED/GREEN/REFACTOR
   커밋 루프를 에이전트가 반복하면서 만들어졌습니다.
7. MVP 이후에는 [docs/findings/](../../docs/findings/), [cycles/](../../cycles/),
   [.codex/agents/](../../.codex/agents/), [.claude/agents/](../../.claude/agents/)를
   통해 유지보수 루프까지 확장했습니다.

따라서 “나도 이런 저장소를 만들고 싶다”의 실제 시작점은 Fastify나 Prisma
코드가 아니라, **작은 제품을 대상으로 이 하네스를 축소 복제하는 것**입니다.
처음부터 35개 유스케이스와 1,000개 이상의 커밋을 목표로 하면 따라가기
어렵습니다. 먼저 5~7개 유스케이스로 같은 구조를 재현해야 합니다.

## 관찰한 증거

### 1. 최초 입력은 제품 구상 대화였습니다

[docs/ideation.md](../../docs/ideation.md)는 사용자와 Claude가 나눈 제품 구상
대화를 기록합니다. 초반부는 Cockburn 유스케이스 방법론 설명으로 시작하고,
뒤쪽에서는 Codex `goal` 기능을 사용해 MVP를 반복 구현하는 구조를 논의합니다.

특히 [docs/ideation.md](../../docs/ideation.md) 안에는 다음 흐름이 있습니다.

- Cockburn 유스케이스 방법론 정리
- vspec 제품 아이디어 구체화
- 16개 도메인 엔티티와 35개 유스케이스 도출
- `GOAL.md`, [AGENTS.md](../../AGENTS.md), [docs/](../../docs/),
  [scripts/](../../scripts/)를 어떻게 만들지 설계
- 프롬프트 캐싱을 위해 “자주 읽는 문서”와 “자주 바뀌는 상태 문서”를 분리

즉, 이 저장소는 “코드를 작성하다가 문서를 붙인 것”이 아니라 “에이전트가
코드를 만들 수 있게 문서와 루프를 먼저 만든 것”에 가깝습니다.

### 2. 최초 커밋은 앱이 아니라 하네스였습니다

최초 커밋은 다음입니다.

```text
826f602 setup: bootstrap autonomous-build harness for vspec MVP
```

이 커밋은 74개 파일과 11,419줄을 추가했습니다. 주요 구성은 다음과 같습니다.

- [AGENTS.md](../../AGENTS.md): 에이전트 작업 규칙
- `GOAL.md`: Codex `goal`에 넣을 미션 프롬프트. 현재 트리에는 없지만 최초
  커밋에는 존재
- [docs/00-overview.md](../../docs/00-overview.md)부터
  [docs/09-bootstrap.md](../../docs/09-bootstrap.md)까지의 제품/기술 명세
- [docs/usecases/](../../docs/usecases/): 35개 fully-dressed 유스케이스
- 최초의 `prisma/schema.prisma`: 현재 위치는
  [apps/api/prisma/schema.prisma](../../apps/api/prisma/schema.prisma)
- [scripts/diagnose.sh](../../scripts/diagnose.sh)
- [scripts/next-task.sh](../../scripts/next-task.sh)
- [scripts/completion-check.sh](../../scripts/completion-check.sh)
- [scripts/update-state.sh](../../scripts/update-state.sh)
- [scripts/verify-tdd.sh](../../scripts/verify-tdd.sh)
- [scripts/check-bypass.sh](../../scripts/check-bypass.sh)
- [scripts/dogfood-test.sh](../../scripts/dogfood-test.sh)

사람이 “빈 프로젝트에서 바로 앱 코드를 짰다”면 첫 커밋이 이렇게 생기기
어렵습니다. 첫 커밋은 제품, 방법론, 상태 관리, 검증 스크립트가 한 번에 들어간
**자율 빌드 부트스트랩**입니다.

### 3. Git 히스토리는 사람이 손으로 반복하기 어려운 밀도입니다

현재 `HEAD` 기준 커밋 수는 1,322개입니다. 날짜별 커밋 수는 다음과 같습니다.

```text
19  2026-05-18
624 2026-05-19
186 2026-05-20
203 2026-05-21
64  2026-05-22
17  2026-05-23
42  2026-05-24
1   2026-05-25
105 2026-05-26
61  2026-05-27
```

커밋 제목 prefix도 루프 흔적을 보입니다.

```text
202 red
202 green
47  refactor
73  docs(state)
39  docs(findings)
```

초기 히스토리는 다음처럼 진행됩니다.

```text
setup: bootstrap autonomous-build harness for vspec MVP
setup: initial scaffolding
docs: refresh state after scaffolding
red: UC-001 signup creates account workspace and session
green: UC-001 complete signup happy path
refactor: UC-001 split signup route handlers
red: UC-001 denied authorization clears state
green: UC-001 clear state on denied auth
...
docs: refresh state after UC-001
red: UC-002 login returns existing user workspaces
green: UC-002 return existing user workspaces on login
```

이 패턴은 [guidelines/goal-iteration.md](../../guidelines/goal-iteration.md)의
RED/GREEN/REFACTOR 규칙과 맞습니다. 사람이 수동으로 모든 파일을 작성했다기보다,
에이전트가 정해진 커밋 단위로 반복 실행한 결과로 보는 것이 자연스럽습니다.

### 4. 현재 저장소는 최초 설계보다 한 단계 진화했습니다

최초 설계는 root `src/`, root `tests/`, root `prisma/` 구조였습니다. 현재는
[apps/api/](../../apps/api/), [apps/cli/](../../apps/cli/),
[apps/app/](../../apps/app/), [apps/www/](../../apps/www/),
[packages/contracts/](../../packages/contracts/)를 가진 pnpm workspace입니다.

이 변화는 “처음부터 완벽한 최종 구조를 만든 것”이 아니라 다음처럼 진화했다는
뜻입니다.

- 1단계: 단일 패키지 형태로 MVP 하네스와 기본 앱을 세움
- 2단계: API, CLI, 웹, 랜딩 페이지를 monorepo 앱으로 분리
- 3단계: [packages/contracts/src/](../../packages/contracts/src/)로 API/CLI
  공유 계약을 분리
- 4단계: [goals/](../../goals/)와 [docs/findings/](../../docs/findings/)로
  유지보수 작업을 큐잉
- 5단계: UI/프레젠테이션 작업은
  [docs/claude/delegation.md](../../docs/claude/delegation.md)를 통해
  Claude Code headless에 일부 위임

따라서 이 저장소를 복제할 때 현재 최종 구조를 한 번에 따라 하려고 하면
복잡합니다. 먼저 최초 하네스의 축소판을 만들고, 나중에 monorepo와 계약 패키지로
분리하는 순서가 더 현실적입니다.

## 실제 작동 메커니즘

### 레이어 1. 제품 명세 레이어

이 레이어는 에이전트가 “무엇을 만들어야 하는가”를 알게 합니다.

대표 파일:

- [docs/00-overview.md](../../docs/00-overview.md): 제품 정의와 MVP 성공 조건
- [docs/01-architecture.md](../../docs/01-architecture.md): 포트/어댑터 구조,
  동시성 모델, 브랜치/세션/락 모델
- [docs/02-tech-stack.md](../../docs/02-tech-stack.md): 기술 스택 제한
- [docs/03-cockburn-method.md](../../docs/03-cockburn-method.md): 유스케이스 작성법
- [docs/05-data-model.md](../../docs/05-data-model.md): 도메인 모델
- [docs/06-api-contract.md](../../docs/06-api-contract.md): REST API 계약
- [docs/07-cli-spec.md](../../docs/07-cli-spec.md): CLI 계약
- [docs/08-file-format.md](../../docs/08-file-format.md): 로컬 마크다운 포맷
- [docs/usecases/](../../docs/usecases/): 기능 단위의 acceptance source

중요한 점은 [apps/api/src/](../../apps/api/src/) 같은 구현보다 이 명세 레이어가
먼저 있었다는 것입니다. 에이전트는 구현 중 판단이 필요할 때 여기로 돌아갑니다.

### 레이어 2. 에이전트 행동 규칙 레이어

[AGENTS.md](../../AGENTS.md)는 에이전트에게 개발자 역할, TDD 원칙, 기술 스택,
레이아웃, 금지 패턴, 커밋 규칙 위치를 알려줍니다.

핵심은 다음입니다.

- 테스트 먼저
- 한 iteration에 한 use case
- 작은 커밋
- 생산 코드 작성 전 실패 테스트 필요
- 새 dependency는 [docs/decisions/](../../docs/decisions/)에 이유를 먼저 기록
- 프레젠테이션 계층은 필요 시 Claude Code headless로 위임 가능

이 파일은 단순한 README가 아니라 에이전트의 운영 헌법입니다.

### 레이어 3. 상태와 다음 작업 라우팅 레이어

에이전트가 매번 전체 저장소를 다시 추론하지 않도록 상태 파일과 스크립트가
있습니다.

- [docs/state/progress.md](../../docs/state/progress.md): 진행률 매트릭스
- [docs/state/next-task.md](../../docs/state/next-task.md): 다음 작업
- [docs/state/blockers.md](../../docs/state/blockers.md): 막힌 문제
- [docs/state/learnings.md](../../docs/state/learnings.md): 반복 중 학습
- [scripts/update-state.sh](../../scripts/update-state.sh): 상태 문서 재생성
- [scripts/next-task.sh](../../scripts/next-task.sh): active goal에 맞는 다음 작업 출력
- `.state/active-goal`: 현재 실패 중인 goal 경로. git에는 보통 올리지 않는
  런타임 상태

초기 `next-task`는 “scaffolding부터 하라”고 지시했고, 현재는
[goals/](../../goals/)의 active goal에 따라 라우팅합니다.

### 레이어 4. 기계 검증 레이어

에이전트가 테스트를 약하게 만들거나 완료 조건을 우회하지 못하도록 스크립트가
완료 조건을 기계적으로 확인합니다.

초기에는 [scripts/completion-check.sh](../../scripts/completion-check.sh)가
다음 항목을 직접 검사했습니다.

- 모든 [docs/usecases/](../../docs/usecases/)에 E2E 테스트가 있는가
- 모든 테스트가 통과하는가
- [scripts/check-bypass.sh](../../scripts/check-bypass.sh)가 통과하는가
- lint/typecheck/coverage가 통과하는가
- [scripts/dogfood-test.sh](../../scripts/dogfood-test.sh)가 통과하는가

현재는 더 발전해서 [scripts/completion-check.sh](../../scripts/completion-check.sh)가
[goals/](../../goals/) 아래 goal들을 번호순으로 실행하고, 첫 실패 goal을
`.state/active-goal`에 기록합니다.

각 goal은 3개 파일 한 세트입니다.

- `goals/<n>-<name>.md`: 자연어 목표와 완료 조건
- `goals/<n>-<name>.gates.sh`: 완료 조건을 검증하는 스크립트
- `goals/<n>-<name>.next-task.sh`: 다음 작업 힌트

예시는 다음입니다.

- [goals/0-init.md](../../goals/0-init.md)
- [goals/0-init.gates.sh](../../goals/0-init.gates.sh)
- [goals/0-init.next-task.sh](../../goals/0-init.next-task.sh)

이 구조 때문에 “에이전트가 알아서 다음 일을 찾는 것처럼” 보입니다. 실제로는
사람이 만든 라우팅 스크립트가 다음 작업 후보를 좁혀줍니다.

### 레이어 5. TDD와 커밋 피드백 레이어

[guidelines/goal-iteration.md](../../guidelines/goal-iteration.md)는 한 iteration을
다음 단계로 나눕니다.

1. [scripts/diagnose.sh](../../scripts/diagnose.sh)로 상태 확인
2. [docs/state/next-task.md](../../docs/state/next-task.md) 읽기
3. 현재 유스케이스 문서 읽기
4. [docs/state/test-plan.md](../../docs/state/test-plan.md)에 테스트 계획 작성
5. RED 테스트 작성 후 커밋
6. GREEN 구현 후 커밋
7. 필요하면 REFACTOR 후 커밋
8. [scripts/verify-tdd.sh](../../scripts/verify-tdd.sh),
   [scripts/check-bypass.sh](../../scripts/check-bypass.sh) 실행
9. [scripts/update-state.sh](../../scripts/update-state.sh) 실행
10. 상태 문서 갱신 커밋

이 저장소의 많은 `red`, `green`, `refactor`, `docs(state)` 커밋은 이 프로토콜의
결과입니다.

### 레이어 6. 후속 유지보수 레이어

MVP 이후에는 단순 유스케이스 구현이 아니라 발견된 문제를 처리하는 체계가
붙었습니다.

- [docs/findings/](../../docs/findings/): 구현과 명세 사이의 gap, 하네스 부채,
  테스트 정직성 문제를 기록
- [cycles/](../../cycles/): 여러 finding을 무인 실행 순서로 묶은 작업 계획
- [guidelines/meta-system-audit.md](../../guidelines/meta-system-audit.md):
  하네스 자체가 과하거나 부정직해지지 않았는지 감사하는 렌즈
- [.codex/agents/harness-engineer.toml](../../.codex/agents/harness-engineer.toml):
  하네스 감사 전용 에이전트 정의
- [.claude/agents/harness-engineer.md](../../.claude/agents/harness-engineer.md):
  Claude용 하네스 감사 에이전트 정의
- [docs/claude/delegation.md](../../docs/claude/delegation.md):
  프레젠테이션 계층 작업을 Claude Code headless에 위임하는 계약

이 레이어가 없으면 “한 번 생성된 코드”는 유지보수되지 않습니다. 이 저장소의
특징은 생성 이후에도 에이전트 루프를 유지보수 프로세스로 확장했다는 점입니다.

## 이전 분석에서 누락됐던 핵심

### 누락 1. 이 저장소는 프롬프트 하나로 만들어진 것이 아닙니다

겉으로 보면 “vibe coding으로 만들었다”처럼 보이지만, 실제로는 프롬프트 하나가
아니라 다음 파일들이 함께 작동했습니다.

- 제품 대화: [docs/ideation.md](../../docs/ideation.md)
- 에이전트 헌법: [AGENTS.md](../../AGENTS.md)
- 최초 미션: `GOAL.md`
- 제품 명세: [docs/00-overview.md](../../docs/00-overview.md),
  [docs/01-architecture.md](../../docs/01-architecture.md),
  [docs/usecases/](../../docs/usecases/)
- 작업 라우터: [scripts/next-task.sh](../../scripts/next-task.sh)
- 완료 판정기: [scripts/completion-check.sh](../../scripts/completion-check.sh)
- 상태 재생성기: [scripts/update-state.sh](../../scripts/update-state.sh)
- TDD 규칙 검사기: [scripts/verify-tdd.sh](../../scripts/verify-tdd.sh)
- 우회 방지기: [scripts/check-bypass.sh](../../scripts/check-bypass.sh)

따라 할 때도 “프롬프트를 잘 쓰기”보다 “에이전트가 읽을 파일 시스템을 만들기”가
먼저입니다.

### 누락 2. next-task가 실제 작업 선택을 좁혀줬습니다

에이전트가 임의로 많은 파일을 뒤지는 것이 아니라
[scripts/next-task.sh](../../scripts/next-task.sh)와 각
`goals/*.next-task.sh`가 작업 후보를 좁혔습니다.

예를 들어 [goals/0-init.next-task.sh](../../goals/0-init.next-task.sh)는 다음
순서로 작업을 고릅니다.

1. `package.json`이 없으면 스캐폴딩
2. `tsconfig.json`이 없으면 TypeScript 설정
3. `vitest.config.ts`가 없으면 테스트 설정
4. [apps/api/prisma/schema.prisma](../../apps/api/prisma/schema.prisma)가 없으면
   Prisma schema 설정
5. 실패 중인 UC 테스트가 있으면 그 테스트 계속 진행
6. 아직 시작하지 않은 다음 UC를 priority order에서 선택
7. 모든 UC가 있으면 gate 실행

이런 라우터가 없으면 에이전트는 매번 “무엇부터 해야 하는지”를 추론해야 하고,
그 과정에서 비용과 오류가 커집니다.

### 누락 3. gate는 단순 테스트 실행이 아니라 명세-검증 계약입니다

[docs/goal-design.md](../../docs/goal-design.md)의 핵심 원칙은
“universal claim이면 gate도 source of truth를 enumerate해야 한다”입니다.

예를 들어 “모든 유스케이스에 테스트가 있어야 한다”는 goal이 있으면 gate는
샘플 하나만 검사하면 안 됩니다. [docs/usecases/](../../docs/usecases/)의
`UC-*.md` 전체를 순회해야 합니다.

이 원칙이 없으면 에이전트는 테스트 하나만 만들고도 “통과했다”고 착각하거나,
우회 패턴을 넣을 수 있습니다.

### 누락 4. 상태 파일은 로그가 아니라 에이전트의 메모리 압축 장치입니다

[docs/state/](../../docs/state/)는 사람이 보는 진행 문서이기도 하지만, 더 중요한
역할은 다음 iteration의 컨텍스트를 줄이는 것입니다.

- [docs/state/progress.md](../../docs/state/progress.md): 전체 진행률을 압축
- [docs/state/next-task.md](../../docs/state/next-task.md): 지금 해야 할 일을 압축
- [docs/state/blockers.md](../../docs/state/blockers.md): 반복 실패를 외부화
- [docs/state/learnings.md](../../docs/state/learnings.md): 다음 판단에 필요한
  짧은 학습만 누적

사람이 따라 할 때 이 파일을 생략하면 에이전트가 매번 전체 저장소를 다시
해석해야 합니다.

### 누락 5. dogfood가 “정말 쓸 수 있는가”를 검증했습니다

단위 테스트와 API 테스트만 있으면 제품이 자기 목적을 달성하는지 알기 어렵습니다.
[scripts/dogfood-test.sh](../../scripts/dogfood-test.sh)는 vspec이 자기 자신의
유스케이스를 관리할 수 있는지 확인하는 self-dogfooding gate입니다.

이런 end-to-end 실제 사용 시나리오가 없으면 에이전트가 내부적으로는 통과하지만
사용자 관점에서는 쓸 수 없는 결과물을 만들 위험이 큽니다.

## 따라 만들기: 현실적인 재현 순서

아래 순서는 이 저장소의 최종 상태를 그대로 복사하는 것이 아니라, 같은 방식으로
작은 프로젝트를 시작하는 방법입니다.

### 0단계. 범위를 줄이세요

처음부터 이 저장소처럼 35개 유스케이스를 쓰면 실패할 가능성이 큽니다.
처음 재현할 때는 다음 정도가 적절합니다.

- 도메인 엔티티 4~6개
- 유스케이스 5~7개
- API 5~10개
- CLI 명령 3~5개
- dogfood 시나리오 1개

목표는 큰 제품을 한 번에 만드는 것이 아니라 **하네스가 실제로 루프를 돌 수
있는지 증명하는 것**입니다.

### 1단계. 원시 구상 대화를 남기세요

새 저장소에 `docs/ideation.md`를 만들고, 제품 아이디어를 AI와 대화한 내용을
그대로 남깁니다. 이 저장소의 예시는 [docs/ideation.md](../../docs/ideation.md)입니다.

여기서 반드시 뽑아야 할 결과물은 다음입니다.

- 제품이 해결하는 문제
- 주요 사용자와 이해관계자
- MVP에서 반드시 되는 일
- MVP에서 하지 않을 일
- 도메인 엔티티 목록
- 유스케이스 목록
- 성공 조건

이 단계에서 코드를 만들지 않습니다.

### 2단계. 제품 명세 문서를 먼저 만드세요

다음 파일들을 축소판으로 만듭니다.

- `docs/00-overview.md`: 제품 정의, 왜 필요한지, MVP 성공 조건
- `docs/01-architecture.md`: 레이어 구조와 큰 설계 결정
- `docs/02-tech-stack.md`: 사용할 기술과 금지 기술
- `docs/03-method.md`: 요구사항 작성 방식
- `docs/04-tdd-protocol.md`: 테스트/커밋 규칙
- `docs/05-data-model.md`: 엔티티와 관계
- `docs/06-api-contract.md`: API 계약
- `docs/07-cli-spec.md`: CLI 계약이 있다면 작성
- `docs/usecases/UC-001-*.md`부터 5~7개

이 저장소에서는 해당 역할을 [docs/00-overview.md](../../docs/00-overview.md),
[docs/01-architecture.md](../../docs/01-architecture.md),
[docs/02-tech-stack.md](../../docs/02-tech-stack.md),
[docs/usecases/](../../docs/usecases/)가 맡습니다.

### 3단계. 에이전트 헌법을 만드세요

루트에 `AGENTS.md`를 만듭니다. 이 파일에는 다음이 들어가야 합니다.

- 에이전트의 역할
- 작은 단계 원칙
- TDD 규칙
- 기술 스택
- 저장소 레이아웃
- 금지 패턴
- 새 의존성 추가 규칙
- 커밋 규칙
- 막혔을 때 처리 방법

이 저장소의 기준 파일은 [AGENTS.md](../../AGENTS.md)입니다.

중요한 점은 이 파일이 너무 일반적이면 안 된다는 것입니다. “좋은 코드를 써라”가
아니라 “어떤 파일을 어디에 두고, 어떤 명령을 실행하고, 어떤 커밋 단위로
나눌지”까지 써야 합니다.

### 4단계. 최초 `GOAL.md`를 만드세요

현재 트리에는 없지만 최초 커밋에는 `GOAL.md`가 있었습니다. 축소판은 다음 구조면
충분합니다.

```markdown
# Mission: Build <product> MVP via TDD

## Goal

1. Every use case in docs/usecases/UC-*.md has a passing E2E test.
2. All quality gates pass.
3. The product can dogfood one representative workflow.
4. scripts/completion-check.sh exits 0.

## Mandatory First Step

Always run:

    bash scripts/diagnose.sh

## Loop

1. Read AGENTS.md.
2. Read docs/state/next-task.md.
3. Read the current use case.
4. Write a failing test.
5. Make it pass with the smallest code change.
6. Run verification scripts.
7. Update state.
8. Repeat until completion-check passes.
```

이 파일은 길 필요가 없습니다. 긴 설명은 [AGENTS.md](../../AGENTS.md)와
[docs/](../../docs/)로 보내고, `GOAL.md`는 에이전트 루프의 입구로 유지하는
편이 좋습니다.

### 5단계. 상태 파일을 만드세요

다음을 만듭니다.

- `docs/state/progress.md`
- `docs/state/next-task.md`
- `docs/state/blockers.md`
- `docs/state/learnings.md`
- `docs/state/test-plan.md`

이 저장소 예시는 [docs/state/](../../docs/state/)입니다.

처음 `next-task.md`는 “스캐폴딩을 하라” 정도로 시작하면 됩니다. 나중에는
[scripts/update-state.sh](../../scripts/update-state.sh)가 자동으로 갱신하게
만듭니다.

### 6단계. 최소 하네스 스크립트를 만드세요

처음부터 현재 [scripts/](../../scripts/) 전체를 따라 할 필요는 없습니다.
최소 세트는 다음입니다.

- `scripts/diagnose.sh`: 현재 상태 출력
- `scripts/next-task.sh`: 다음 작업 출력
- `scripts/completion-check.sh`: 완료 조건 전체 검증
- `scripts/update-state.sh`: progress와 next-task 갱신
- `scripts/verify-tdd.sh`: RED/GREEN 커밋 패턴 점검
- `scripts/check-bypass.sh`: `.skip`, `todo`, tautological assertion 등 우회 방지
- `scripts/dogfood-test.sh`: 대표 실제 사용 시나리오 검증

이 저장소의 대응 파일:

- [scripts/diagnose.sh](../../scripts/diagnose.sh)
- [scripts/next-task.sh](../../scripts/next-task.sh)
- [scripts/completion-check.sh](../../scripts/completion-check.sh)
- [scripts/update-state.sh](../../scripts/update-state.sh)
- [scripts/verify-tdd.sh](../../scripts/verify-tdd.sh)
- [scripts/check-bypass.sh](../../scripts/check-bypass.sh)
- [scripts/dogfood-test.sh](../../scripts/dogfood-test.sh)

가장 먼저 만들 스크립트는 `diagnose.sh`와 `completion-check.sh`입니다.
`diagnose.sh`는 에이전트가 현재 상태를 추론하지 않도록 해주고,
`completion-check.sh`는 “끝났는지”를 사람 대신 판정합니다.

### 7단계. 첫 goal은 단순하게 유지하세요

처음 goal은 [goals/0-init.md](../../goals/0-init.md)처럼 “MVP를 TDD로 만든다”로
충분합니다. 단, 처음부터 goal stack을 만들지 않아도 됩니다. 다음 중 하나를
고르세요.

- 더 단순한 방식: `GOAL.md` + `scripts/completion-check.sh`만으로 시작
- 이 저장소 방식: `goals/0-init.md`,
  `goals/0-init.gates.sh`, `goals/0-init.next-task.sh` 3종 세트로 시작

장기적으로는 두 번째 방식이 좋습니다. 이후 새로운 목표를
`goals/1-runnable`, `goals/2-shippable`처럼 쌓을 수 있기 때문입니다.

### 8단계. Codex goal을 돌리기 전에 사람이 한 번 dry-run 하세요

에이전트에게 넘기기 전에 사람이 직접 다음을 해보세요.

```text
bash scripts/diagnose.sh
bash scripts/next-task.sh
bash scripts/completion-check.sh
```

이 세 명령만으로 다음 질문에 답할 수 있어야 합니다.

- 지금 무엇이 안 됐는가?
- 다음에 무엇을 해야 하는가?
- 전체 완료 조건은 무엇 때문에 실패하는가?

이 질문에 답하지 못하면 아직 에이전트에게 넘기면 안 됩니다.

### 9단계. 첫 iteration은 스캐폴딩만 맡기세요

처음부터 “전체 MVP를 완성해줘”라고 하지 말고, 첫 iteration이 다음 정도만 하게
하세요.

- package manager 설정
- TypeScript 설정
- Vitest 설정
- smoke test 작성
- 첫 `setup` 커밋
- [scripts/update-state.sh](../../scripts/update-state.sh) 실행
- 상태 문서 갱신 커밋

이 저장소도 최초 하네스 커밋 다음에 `setup: initial scaffolding`이 나옵니다.

### 10단계. 유스케이스 하나씩 RED/GREEN으로 진행하세요

유스케이스 하나마다 다음 흐름을 강제하세요.

1. 현재 UC 문서 읽기
2. [docs/state/test-plan.md](../../docs/state/test-plan.md)에 테스트 계획 작성
3. 첫 실패 테스트 작성
4. `red: UC-XXX ...` 커밋
5. 최소 구현
6. `green: UC-XXX ...` 커밋
7. 필요하면 정리
8. `refactor: UC-XXX ...` 커밋
9. [scripts/update-state.sh](../../scripts/update-state.sh)
10. `docs(state): refresh state after UC-XXX` 커밋

이 단계가 반복되면서 앱 코드가 생깁니다. 앱 코드는 하네스의 입력과 검증에
반응한 결과물입니다.

### 11단계. MVP 이후에 findings와 cycles를 붙이세요

처음부터 [docs/findings/](../../docs/findings/)와 [cycles/](../../cycles/)까지
만들 필요는 없습니다. MVP 루프가 돌아가기 시작한 뒤에 붙이세요.

붙이는 순서는 다음이 좋습니다.

1. 구현과 명세가 어긋난 부분을 `docs/findings/YYYY-MM-DD-*.md`로 기록
2. 여러 finding을 묶어 `cycles/YYYYMMDD-*.md`로 작업 큐 생성
3. 각 cycle에 실행 순서, guard, rollback 조건, 종료 조건 작성
4. 매 work-unit 후 commit/push 규칙 지정
5. 메타 감사 규칙 추가

이 저장소의 예시는
[cycles/260527-01-findings-meta-audit-sweep.md](../../cycles/260527-01-findings-meta-audit-sweep.md)입니다.

## 최소 복제 체크리스트

아래 항목이 없으면 “AI가 만든 코드베이스”처럼 보일 수는 있어도, 이 저장소와
같은 방식으로 반복 성장하기 어렵습니다.

### 반드시 있어야 하는 파일

- `AGENTS.md`
- `GOAL.md` 또는 `goals/0-init.md`
- `docs/00-overview.md`
- `docs/01-architecture.md`
- `docs/02-tech-stack.md`
- `docs/05-data-model.md`
- `docs/06-api-contract.md`
- `docs/usecases/UC-001-*.md`부터 최소 5개
- `docs/state/progress.md`
- `docs/state/next-task.md`
- `docs/state/blockers.md`
- `docs/state/test-plan.md`
- `scripts/diagnose.sh`
- `scripts/next-task.sh`
- `scripts/completion-check.sh`
- `scripts/update-state.sh`
- `scripts/check-bypass.sh`
- `scripts/verify-tdd.sh`

### 반드시 있어야 하는 규칙

- 생산 코드 전 실패 테스트
- 한 use case씩 진행
- RED/GREEN/REFACTOR 커밋 분리
- 모든 완료 조건은 script로 판정
- 상태 문서는 스크립트가 갱신
- 막히면 blocker에 기록하고 다음 작업으로 이동
- 테스트 우회 패턴 금지
- 새 의존성은 결정 문서 작성 후 추가

### 처음에는 없어도 되는 것

- [apps/app/](../../apps/app/) 같은 웹 UI
- [apps/www/](../../apps/www/) 같은 랜딩 페이지
- [packages/contracts/](../../packages/contracts/) 같은 공유 계약 패키지
- [docs/claude/delegation.md](../../docs/claude/delegation.md) 같은 위임 시스템
- [.codex/agents/](../../.codex/agents/)와 [.claude/agents/](../../.claude/agents/)
  같은 특수 에이전트 정의
- 복잡한 gate cache
- 30개 이상의 goal stack
- 1,000개 이상의 커밋 히스토리

이것들은 MVP 하네스가 돈 뒤에 추가된 확장 레이어입니다.

## 이 저장소를 공부하는 순서

따라 만들 목적이라면 다음 순서로 읽는 것이 좋습니다.

1. [docs/ideation.md](../../docs/ideation.md): 어떻게 제품과 하네스가 구상됐는지
2. [AGENTS.md](../../AGENTS.md): 에이전트에게 어떤 개발자 역할을 부여했는지
3. [docs/00-overview.md](../../docs/00-overview.md): 제품 목표
4. [docs/usecases/_index.md](../../docs/usecases/_index.md): 유스케이스 범위
5. [docs/usecases/UC-001-signup.md](../../docs/usecases/UC-001-signup.md):
   fully-dressed 유스케이스 형식
6. [scripts/diagnose.sh](../../scripts/diagnose.sh): 상태 진단 방식
7. [scripts/next-task.sh](../../scripts/next-task.sh): 다음 작업 라우팅 방식
8. [scripts/completion-check.sh](../../scripts/completion-check.sh): 완료 판정 방식
9. [goals/0-init.md](../../goals/0-init.md),
   [goals/0-init.gates.sh](../../goals/0-init.gates.sh),
   [goals/0-init.next-task.sh](../../goals/0-init.next-task.sh): goal 3종 세트
10. [guidelines/goal-iteration.md](../../guidelines/goal-iteration.md):
    실제 iteration 운영 방식
11. [docs/goal-design.md](../../docs/goal-design.md): goal stack과 gate 설계
12. [cycles/260527-01-findings-meta-audit-sweep.md](../../cycles/260527-01-findings-meta-audit-sweep.md):
    MVP 이후 무인 유지보수 방식

앱 코드는 그 다음에 보는 편이 좋습니다. 먼저 [apps/api/src/](../../apps/api/src/)를
보면 “왜 이렇게 나뉘었는지”가 잘 안 보입니다. 위 문서를 먼저 읽으면 앱 코드가
명세와 하네스의 산출물로 보입니다.

## 따라 할 때의 실전 전략

### 좋은 시작점

새 프로젝트를 만든다면 하루 목표를 이렇게 잡는 것이 현실적입니다.

1. `docs/ideation.md`에 제품 구상 기록
2. `docs/00-overview.md`와 `docs/usecases/` 5개 작성
3. `AGENTS.md` 작성
4. `GOAL.md` 작성
5. `scripts/diagnose.sh`, `scripts/next-task.sh`,
   `scripts/completion-check.sh` 작성
6. 사람이 직접 세 스크립트 실행
7. Codex goal에 `GOAL.md`를 넣고 첫 스캐폴딩만 시도

이 단계까지 성공하면 그 다음에 기술 스택과 기능을 늘리면 됩니다.

### 피해야 할 시작점

다음 방식은 실패 가능성이 큽니다.

- “이 저장소처럼 만들어줘”라고 한 번에 요청
- 앱 코드부터 복사
- [apps/api/](../../apps/api/)와 [apps/cli/](../../apps/cli/)를 먼저 분석
- 35개 유스케이스를 처음부터 작성
- completion script 없이 에이전트에게 “알아서 계속해”라고 지시
- 상태 파일 없이 매번 새 대화에서 이어가기
- 테스트 우회 검사 없이 통과 여부를 에이전트 판단에 맡기기

이 저장소의 핵심은 에이전트의 자율성이 아니라, 자율성을 좁은 레일 위에 올린
것입니다.

## 이 분석의 한계

이 분석은 로컬 저장소의 파일, git 히스토리, 최초 커밋, 현재 하네스 문서,
스크립트 구조를 근거로 합니다. 어떤 모델이 어떤 파일을 생성했는지까지는
증명하지 못합니다.

저장소 밖에 있었을 수 있는 원본 프롬프트, 작업 로그, 도구 설정, 외부 노트,
실패한 실험, 사람의 수동 판단은 이 분석만으로 복원할 수 없습니다. 이 공백은
[원 개발자 확인 질문 목록](../questions/2026-05-27-original-developer-questions.md)으로
따로 정리했습니다.

하지만 다음 사실은 충분히 강하게 말할 수 있습니다.

- 이 저장소는 사람이 앱 코드를 직접 전부 작성한 흔적보다, 에이전트 루프를
  설계하고 실행한 흔적이 훨씬 큽니다.
- 따라 해야 할 대상은 “코드 스타일”이 아니라 “명세 → next-task → TDD →
  gate → state update → commit” 루프입니다.
- 처음부터 현재 최종 구조를 따라 하지 말고, 최소 하네스의 축소판을 먼저
  만들어야 합니다.
