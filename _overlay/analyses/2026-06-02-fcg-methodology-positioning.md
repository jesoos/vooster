# FCG 개발 방식의 위치와 방법론 비교

날짜: 2026-06-02
범위: 이 fork 전용 분석
상태: 저장소 증거와 재현 실험 준비 내용을 바탕으로 한 판단

## 결론

`findings -> cycles -> goals`는 Scrum, Kanban, TDD처럼 이미 널리 통용되는
표준 방법론 이름으로 보기는 어렵습니다. 이 저장소 주인이 Vooster 작업 중
AI looping agent에 맞게 조합하고 이름 붙인 운영 패턴으로 보는 것이 더 타당합니다.

다만 완전히 새롭거나 고립된 발명도 아닙니다. 핵심 부품은 기존 방법론에서 온
것입니다.

- TDD: RED/GREEN/REFACTOR 단위의 작은 검증 루프
- Cockburn use case: 기능을 사용자 목표와 acceptance source로 표현
- CI/CD gate: 완료 조건을 사람이 아니라 스크립트가 판정
- Kanban/backlog: 발견한 문제를 작업 후보로 보존
- ADR/decision log: 의존성, 구조, 정책 변경 이유를 기록
- Agent prompt engineering: 에이전트가 반복해서 읽을 규칙과 상태를 분리

FCG의 차별점은 이 부품들을 사람 팀의 회의/보드 중심이 아니라, **Codex 같은
에이전트가 장시간 혼자 실행할 수 있는 파일 계약**으로 재배열한 데 있습니다.

```text
findings -> cycles -> goals -> next-task -> TDD -> gates -> state -> commit/push
관찰/부채   실행 묶음   영속 완료조건   다음 행동   구현   검증    기억     공유
```

## 원 개발자가 이 프로세스를 알게 된 경로에 대한 판단

저장소 안의 증거만으로 원 개발자의 학습 경로를 확정할 수는 없습니다. 그래도
가능성이 높은 흐름은 다음입니다.

1. 제품 구상 단계에서 [docs/ideation.md](../../docs/ideation.md)에 남은 것처럼
   Claude와 Cockburn use case, MVP 범위, Codex `goal` 루프를 함께 구체화했습니다.
2. [AGENTS.md](../../AGENTS.md)에 Kent Beck식 TDD 원칙을 강하게 고정했습니다.
3. 최초 커밋 `826f602`에서 `GOAL.md`, `docs/state/`, `scripts/diagnose.sh`,
   `scripts/next-task.sh`, `scripts/completion-check.sh`를 한 번에 넣어
   에이전트가 반복 실행할 수 있는 하네스를 만들었습니다.
4. 구현이 진행되면서 단순 `GOAL.md` 기반 루프만으로는 유지보수와 후속 미션
   관리가 어렵다는 문제가 생겼고, 이후 [goals/](../../goals/),
   [cycles/](../../cycles/), [docs/findings/](../../docs/findings/) 구조로
   확장한 것으로 보입니다.
5. 그 확장된 구조가 나중에 `greatSumini/cc-system`의
   `findings-cycles-goals` 키트로 추출됐을 가능성이 높습니다. 자세한 근거는
   [2026-05-27-cc-system-fcg-relation.md](2026-05-27-cc-system-fcg-relation.md)를
   봅니다.

따라서 이 개발자는 "아무것도 없는 상태에서 즉흥적으로 앱을 만든" 것보다는,
AI와 먼저 작업 운영체제를 설계하고, 그 운영체제 위에서 앱을 자라게 한 것으로
보는 편이 맞습니다.

## 기존 방법론과의 비교

### TDD와 비교

TDD는 FCG의 가장 강한 기반입니다. [AGENTS.md](../../AGENTS.md)와
[guidelines/goal-iteration.md](../../guidelines/goal-iteration.md)는 테스트를
설계 도구로 보고, production code 이전에 failing test를 요구합니다.

차이점은 TDD가 보통 개발자의 사고 루프인 반면, FCG는 그 루프를 에이전트가
실행할 수 있도록 파일과 스크립트로 외부화한다는 점입니다. FCG에서 테스트는
개별 기능 검증이고, `gates.sh`는 goal 전체의 완료 판정입니다.

### Scrum과 비교

Scrum은 sprint, planning, review, retrospective 같은 팀 의식을 중심에 둡니다.
FCG의 `cycles/`는 sprint와 비슷하지만 회의 단위가 아닙니다.

FCG cycle은 "어떤 finding을 어떤 goal로 승격해 어떤 gate로 닫을 것인가"를
담는 실행 묶음입니다. 사람이 논의한 결과를 보존할 수는 있지만, 핵심 목적은
에이전트가 다음 실행에서 바로 읽고 움직이게 하는 것입니다.

### Kanban과 비교

Kanban은 backlog와 WIP 제한을 통해 흐름을 관리합니다. FCG의
[docs/findings/](../../docs/findings/)는 backlog와 비슷합니다.

차이점은 FCG finding이 단순 할 일 목록이 아니라, 관찰한 문제, 영향, 승격 대상,
완료 증거까지 연결하는 재료라는 점입니다. finding은 곧바로 구현되지 않고,
cycle에서 선택된 뒤 goal로 승격됩니다.

### Use Case Driven Development와 비교

Vooster는 [docs/usecases/](../../docs/usecases/)를 매우 강하게 사용합니다.
유스케이스는 기능 명세이자 테스트 대상이며, [scripts/next-task.sh](../../scripts/next-task.sh)와
[scripts/completion-check.sh](../../scripts/completion-check.sh)가 순회하는
source of truth입니다.

FCG는 use case driven development를 대체하지 않습니다. 오히려 use case를
goal gate 안에 묶어 에이전트의 작업 순서를 제어합니다.

### CI/CD와 비교

CI/CD는 변경 후 검증과 배포를 자동화합니다. FCG의 `*.gates.sh`와
[scripts/completion-check.sh](../../scripts/completion-check.sh)는 CI/CD의
local-first 버전입니다.

차이점은 FCG gate가 배포 직전 품질 확인뿐 아니라, 에이전트의 "완료" 판단 자체를
대신한다는 점입니다. gate가 실패하면 goal은 아직 끝나지 않은 것입니다.

### ADR/Decision Log와 비교

Vooster는 새 의존성을 추가하기 전에 [docs/decisions/](../../docs/decisions/)에
이유를 쓰도록 요구합니다. 이는 ADR과 유사합니다.

FCG에서는 decision log가 독립 문서로 끝나지 않고, goal과 gate의 제약으로 다시
연결됩니다. 의사결정은 에이전트가 이후 반복에서 같은 결정을 번복하지 않게 하는
메모리 장치입니다.

## FCG를 실제로 사용할 때의 최소 조건

새 프로젝트에서 FCG를 쓰려면 앱 코드보다 아래 항목이 먼저 필요합니다.

1. 에이전트 작업 헌법: `AGENTS.md`
2. 제품의 작은 목표: `GOAL.md` 또는 `goals/<n>.md`
3. 다음 작업 라우터: `scripts/next-task.sh`
4. 완료 판정: `scripts/completion-check.sh`와 `goals/*.gates.sh`
5. 상태 문서: `docs/state/progress.md`, `docs/state/next-task.md`,
   `docs/state/blockers.md`, `docs/state/learnings.md`
6. 발견 큐: `docs/findings/`
7. 실행 묶음: `cycles/`
8. 작은 TDD 커밋을 남기는 git 규칙

이 중 `findings`, `cycles`, `goals`만 만들고 gate가 약하면 FCG가 아니라 문서화된
TODO 목록에 가까워집니다. 반대로 gate만 있고 finding/cycle이 없으면 장기
운영에서 왜 그 goal이 생겼는지 추적하기 어렵습니다.

## 실험에서 검증해야 할 가설

아직 저장소만으로 확정하기 어려운 부분은 작은 재현 실험으로 검증해야 합니다.

1. `GOAL.md` 기반 proto-harness만으로도 첫 MVP slice가 가능한가?
2. FCG 3자산을 처음부터 넣으면 오히려 초반 복잡도가 커지지 않는가?
3. `next-task.sh`가 충분히 구체적이면 에이전트가 멈추지 않고 진행하는가?
4. `gates.sh`가 너무 약하면 에이전트가 얕은 구현으로 통과하지 않는가?
5. 자동 commit/push를 허용하려면 어떤 hook과 prompt 경계가 필요한가?

이 가설을 검증하기 위한 현재 실험 저장소는
[/home/ubuntu/workspace/personal/my-vooster-lab](/home/ubuntu/workspace/personal/my-vooster-lab)입니다.
