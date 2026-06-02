# 재현 실험 랩과 Slash Goal 운용 정리

날짜: 2026-06-02
범위: `my-vooster-lab` 재현 실험, `/goal` 실행 방식, 자동 commit/push 구조

## 결론

현재 재현 실험은 원본 fork 내부의 하위 디렉터리가 아니라 별도 저장소인
[/home/ubuntu/workspace/personal/my-vooster-lab](/home/ubuntu/workspace/personal/my-vooster-lab)에서
진행하는 것이 맞습니다.

이유는 다음과 같습니다.

- Vooster fork의 upstream sync와 충돌하지 않습니다.
- nested git repository가 되지 않습니다.
- GitHub 원격을 별도로 연결할 수 있습니다.
- Codex 대화 컨텍스트는 파일로 고정해 새 세션에 넘길 수 있습니다.
- 실패하면 원본 분석 저장소를 건드리지 않고 폐기할 수 있습니다.

이 랩의 목적은 Vooster의 최종 앱 구조를 복사하는 것이 아니라,
`findings -> cycles -> goals` 하네스가 실제로 새 프로젝트에서 작동하는지
확인하는 것입니다.

## 랩 저장소의 현재 역할

`my-vooster-lab`는 작은 TypeScript CLI를 대상으로 FCG를 실험합니다.

현재 랩에서 새 Codex가 먼저 읽어야 하는 주요 파일은 다음입니다.

- [HANDOFF.md](/home/ubuntu/workspace/personal/my-vooster-lab/HANDOFF.md)
- [GOAL.md](/home/ubuntu/workspace/personal/my-vooster-lab/GOAL.md)
- [AGENTS.md](/home/ubuntu/workspace/personal/my-vooster-lab/AGENTS.md)
- [SOURCE_REFS.md](/home/ubuntu/workspace/personal/my-vooster-lab/SOURCE_REFS.md)
- [SLASH_GOAL_PROMPT.md](/home/ubuntu/workspace/personal/my-vooster-lab/SLASH_GOAL_PROMPT.md)
- [SLASH_LONG_GOAL_PROMPT.md](/home/ubuntu/workspace/personal/my-vooster-lab/SLASH_LONG_GOAL_PROMPT.md)

랩의 핵심 FCG 파일은 다음입니다.

- [docs/findings/2026-05-29T1015-fcg-layer-missing.md](/home/ubuntu/workspace/personal/my-vooster-lab/docs/findings/2026-05-29T1015-fcg-layer-missing.md)
- [docs/findings/2026-05-29T1025-short-goal-too-small.md](/home/ubuntu/workspace/personal/my-vooster-lab/docs/findings/2026-05-29T1025-short-goal-too-small.md)
- [cycles/260529-01-fcg-bootstrap.md](/home/ubuntu/workspace/personal/my-vooster-lab/cycles/260529-01-fcg-bootstrap.md)
- [cycles/260529-02-hours-scale-cli-mvp.md](/home/ubuntu/workspace/personal/my-vooster-lab/cycles/260529-02-hours-scale-cli-mvp.md)
- [goals/0-fcg-lab-bootstrap.md](/home/ubuntu/workspace/personal/my-vooster-lab/goals/0-fcg-lab-bootstrap.md)
- [goals/1-uc001-cli-loop.md](/home/ubuntu/workspace/personal/my-vooster-lab/goals/1-uc001-cli-loop.md)
- [goals/2-hours-scale-cli-mvp.md](/home/ubuntu/workspace/personal/my-vooster-lab/goals/2-hours-scale-cli-mvp.md)

## `/goal`은 왜 자동 commit/push처럼 보이는가

`/goal` 자체가 git commit과 push를 자동으로 해 주는 마법 기능이라고 보는 것은
부정확합니다.

실제로는 아래 조건이 함께 맞물립니다.

1. goal prompt가 에이전트에게 commit/push를 명시적으로 지시합니다.
2. [AGENTS.md](../../AGENTS.md)나 랩의 `AGENTS.md`가 작은 TDD 커밋 규칙을
   고정합니다.
3. `scripts/next-task.sh`가 다음 작업을 좁혀 줍니다.
4. `scripts/completion-check.sh`와 `goals/*.gates.sh`가 완료 판정을 대신합니다.
5. git hook 또는 수동 검증 명령이 commit/push 직전 검증 경계가 됩니다.
6. Codex 세션이 shell 명령 실행 권한을 갖고 있으면, 에이전트가 `git commit`,
   `git push`를 실제로 실행합니다.

따라서 자동 commit/push의 본질은 "도구 기능"보다 "프롬프트와 하네스가
에이전트에게 어디까지 해도 되는지 명시한 것"에 가깝습니다.

현재 사용자 지시처럼 hook 실행을 금지한 경우에는 `git commit --no-verify`,
`git push --no-verify`를 사용해야 합니다. 이때도 gate 자체를 약화하면 안 됩니다.
hook을 생략하더라도 필요한 검증 명령은 별도로 실행해야 합니다.

## `/goal` 세션은 매번 새로 여는 것이 좋은가

짧은 확인 작업은 같은 세션에서 이어도 됩니다. 하지만 몇 시간 규모 goal은 새
Codex 세션에서 시작하는 편이 낫습니다.

이유는 다음입니다.

- FCG는 chat memory가 아니라 disk state를 기준으로 재개되도록 설계되어 있습니다.
- 긴 goal은 컨텍스트가 커져 이전 대화의 잡음이 다음 판단에 섞이기 쉽습니다.
- 새 세션은 [HANDOFF.md](/home/ubuntu/workspace/personal/my-vooster-lab/HANDOFF.md),
  [SLASH_LONG_GOAL_PROMPT.md](/home/ubuntu/workspace/personal/my-vooster-lab/SLASH_LONG_GOAL_PROMPT.md),
  `scripts/diagnose.sh`, `scripts/next-task.sh`만으로 재개 가능해야 합니다.
- 중단 후 재시작할 때 `.state/active-goal`, `completion-check.sh`,
  `next-task.sh`가 기준점이 됩니다.

즉 대화 컨텍스트를 보존하려고 같은 세션에 오래 매달리기보다, 중요한 내용을
파일로 기록하고 새 세션이 그 파일을 읽게 하는 방식이 FCG에 더 맞습니다.

## 장시간 예시 goal을 만든 이유

사용자가 짧은 `/goal` 실험이 7분 48초 만에 끝났다고 지적했습니다. 그 실험은
하네스가 시작되는지는 보여 줬지만, FCG의 핵심인 장시간 루프를 검증하기에는
부족했습니다.

그래서 랩에는 [goals/2-hours-scale-cli-mvp.md](/home/ubuntu/workspace/personal/my-vooster-lab/goals/2-hours-scale-cli-mvp.md)를
추가했습니다.

이 goal은 다음을 요구합니다.

- `lab-task create`, `next`, `complete`, `list`, `show`, `doctor` CLI
- JSON 파일 기반 durable storage
- domain, store, command, CLI entrypoint 분리
- 최소 8개 테스트 파일
- UC-002, UC-003, persistence, CLI e2e의 RED/GREEN 커밋 증거
- `docs/state/` 업데이트
- finding -> cycle -> goal 승격 흔적

핵심은 "빨리 끝나는 예시"가 아니라, 에이전트가 여러 TDD cycle과 gate를 거치며
몇 시간 동안 안정적으로 진행할 수 있는지를 관찰하는 것입니다.

## 새 Codex에서 실행하는 방법

랩에서 긴 실험을 실행하려면 다음처럼 시작합니다.

```bash
cd /home/ubuntu/workspace/personal/my-vooster-lab
codex
```

그 다음 [SLASH_LONG_GOAL_PROMPT.md](/home/ubuntu/workspace/personal/my-vooster-lab/SLASH_LONG_GOAL_PROMPT.md)의
코드블록을 첫 입력으로 붙여 넣습니다.

그 프롬프트의 핵심 지시는 다음입니다.

1. `git status --short --branch`와 `git config --get core.hooksPath` 확인
2. `core.hooksPath`가 `scripts/hooks`가 아니면 설정
3. `bash scripts/completion-check.sh`로 active goal 갱신
4. active goal이 `goals/2-hours-scale-cli-mvp.md`인지 확인
5. 매 iteration 시작 시 `bash scripts/diagnose.sh`,
   `bash scripts/next-task.sh` 실행
6. `next-task.sh`가 출력한 한 task만 수행
7. 제품 변경은 RED -> GREEN -> REFACTOR 순서로 진행
8. explicit path만 stage해서 commit
9. hook 금지 지시가 있으면 `--no-verify` 사용
10. `completion-check.sh` 통과 전에는 push하지 않음

## 현재 랩에서 추가로 확인할 것

랩 저장소는 FCG 실험을 위한 최소 구조가 들어가 있지만, goal 실행자는 매번
다음을 확인해야 합니다.

```bash
git status --short --branch
git config --get core.hooksPath
bash scripts/completion-check.sh
bash scripts/next-task.sh
```

특히 long goal은 의도적으로 gate가 실패하는 상태에서 시작합니다. 이것은 실패가
아니라 다음 작업을 산출하기 위한 정상 상태입니다.

## 원본 Vooster와 연결되는 파일

랩 실험이 참조하는 원본 분석 문서는 다음입니다.

- [2026-05-27-repo-origin-analysis.md](2026-05-27-repo-origin-analysis.md)
- [2026-05-27-cc-system-fcg-relation.md](2026-05-27-cc-system-fcg-relation.md)
- [2026-05-27-vooster-initial-commit-reanalysis.md](2026-05-27-vooster-initial-commit-reanalysis.md)
- [../questions/2026-05-27-original-developer-questions.md](../questions/2026-05-27-original-developer-questions.md)

이 문서들은 랩이 원본 앱 코드를 복사하지 않고, 하네스 구조와 실행 원리를
복제하도록 기준을 제공합니다.
