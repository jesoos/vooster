# cc-system과 FCG 관계 분석

날짜: 2026-05-27
범위: `greatSumini/cc-system`과 Vooster autonomous build harness의 선후 관계,
`findings → cycles → goals` 개념 출처 분석

## 결론

`findings → cycles → goals`, 줄여서 FCG는 알려진 업계 표준 용어라기보다
Vooster/cc-system 주인이 만든 AI looping agent용 하네스 패턴으로 보는 것이
타당합니다.

단, 구성 요소 자체는 새롭지 않습니다.

- `findings`: 이슈, 부채 큐, 감사 finding
- `cycles`: iteration, sprint, work cycle
- `goals`: 목표, acceptance criteria, CI gate
- `gates.sh`: 기계 검증/CI gate

새로운 부분은 이 세 자산을 다음처럼 AI 에이전트 운영 모델로 묶은 것입니다.

```text
findings -> cycles -> goals
부채/통찰    무인 실행 프롬프트    영속 gate invariant
```

공개 저장소 증거 기준으로는 **Vooster가 먼저 이 패턴을 실험했고,
`cc-system/findings-cycles-goals/`가 나중에 그 핵심을 스택 중립 이식 키트로
추출한 것**에 가깝습니다.

## 확인한 자료

검토한 저장소:

- Vooster fork: [../../](../../)
- `cc-system`: `https://github.com/greatSumini/cc-system`

로컬 검토용 clone:

```text
/tmp/cc-system-inspect
```

중요 커밋:

```text
Vooster 826f602 2026-05-18T17:47:42+09:00
setup: bootstrap autonomous-build harness for vspec MVP

cc-system 17179bd 2026-05-26T02:52:54+09:00
feat(prompt): findings-cycles-goals 빌드 하네스 이식 키트 + install 프롬프트 추가 (#2)

cc-system 1f8375b 2026-04-30T20:48:57+09:00
discord-message 스킬 추가
```

## 근거

### 1. cc-system의 FCG 키트는 Vooster 최초 커밋보다 늦습니다

Vooster 최초 하네스 커밋은 2026-05-18입니다.

```text
826f6023368c55e715c39f86b20cbee64cf30afd
2026-05-18T17:47:42+09:00
setup: bootstrap autonomous-build harness for vspec MVP
```

`cc-system/findings-cycles-goals/`는 2026-05-26에 추가됐습니다.

```text
17179bd0fff9ef3d5cec51322b11411f48051760
2026-05-26T02:52:54+09:00
feat(prompt): findings-cycles-goals 빌드 하네스 이식 키트 + install 프롬프트 추가 (#2)
```

따라서 Vooster 최초 커밋이 현재 공개된 FCG 키트를 가져와 스캐폴딩됐다는
가설은 시간상 맞지 않습니다.

### 2. cc-system 문서가 Vooster에서 추출했다고 설명합니다

`cc-system`의 [findings-cycles-goals README](https://github.com/greatSumini/cc-system/blob/main/findings-cycles-goals/README.md)는
이 키트를 `vibemafiaclub/vooster`가 쓰는 autonomous build harness의 핵심을
스택 중립적으로 추출한 이식 키트라고 설명합니다.

`cc-system`의 [README](https://github.com/greatSumini/cc-system/blob/main/README.md)도
`findings-cycles-goals/`를 Vooster의 findings→cycles→goals 하네스를 스택
중립으로 추출한 이식 키트라고 소개합니다.

문서 설명도 `cc-system -> Vooster`가 아니라 `Vooster -> cc-system kit` 방향을
가리킵니다.

### 3. FCG 이전 cc-system과 Vooster 최초 커밋의 파일 구조는 다릅니다

`cc-system`에서 FCG가 추가되기 전 커밋(`1f8375b`)과 Vooster 최초 커밋
(`826f602`)을 비교하면 exact path overlap은 두 파일뿐이었습니다.

```text
.gitignore
README.md
```

basename overlap도 동일하게 `.gitignore`, `README.md`뿐이었습니다.

반대로 Vooster 최초 커밋에는 다음이 있었지만, FCG 이전 `cc-system`에는 같은
구조가 없었습니다.

- `AGENTS.md`
- `GOAL.md`
- `docs/usecases/`
- `docs/state/`
- `scripts/diagnose.sh`
- `scripts/next-task.sh`
- `scripts/completion-check.sh`
- `scripts/update-state.sh`
- `scripts/verify-tdd.sh`

### 4. FCG 이전 cc-system의 하네스는 다른 계열입니다

2026-04-24 이후 `cc-system`에는 자율주행 하네스 계열이 있었습니다. 하지만 그
구조는 Vooster 최초 커밋과 다릅니다.

`cc-system`의 기존 하네스:

- `.claude/skills/{persuasion-review,ideation,plan-and-build,commit}/`
- `.claude/agents/tech-critic-lead.md`
- `scripts/run-server.py`
- `scripts/run-phases.py`
- `scripts/gen-docs-diff.py`
- `prompts/task-create.md`
- `iterations/`
- `tasks/`

Vooster 최초 커밋:

- `GOAL.md`
- `AGENTS.md`
- `docs/00..09`
- `docs/usecases/UC-001..UC-035`
- `docs/state/*`
- `scripts/diagnose.sh`
- `scripts/next-task.sh`
- `scripts/completion-check.sh`
- `scripts/verify-tdd.sh`

두 계열 모두 “자율 에이전트가 반복 실행한다”는 철학은 공유할 수 있지만, 파일
계약과 실행 루프는 다릅니다.

### 5. Vooster 최초 커밋에는 cc-system 명시 참조가 없습니다

Vooster 최초 커밋의 핵심 파일에서 다음 키워드를 찾았지만 명시 참조는
나오지 않았습니다.

```text
cc-system
greatSumini
run-server
run-phases
persuasion
plan-and-build
HARNESS_HEADLESS
iterations/
tasks/
```

이것은 “절대 영향이 없었다”는 증거는 아닙니다. 다만 저장소 내부 증거만으로
볼 때, Vooster 최초 커밋이 `cc-system` 파일을 직접 이식했다는 흔적은 없습니다.

## 판단

### 낮은 가능성

Vooster 최초 커밋이 현재 공개된 `cc-system/findings-cycles-goals/` 키트를 참고해
스캐폴딩됐다.

이유:

- FCG 키트 공개 커밋이 Vooster 최초 커밋보다 늦습니다.
- `cc-system` 문서가 Vooster에서 추출했다고 설명합니다.
- FCG 이전 `cc-system`과 Vooster 최초 커밋의 파일 구조가 거의 겹치지 않습니다.

### 중간 가능성

원 개발자가 `cc-system`의 이전 Claude Code 자율주행 하네스에서 개념적 영향을
받았다.

이유:

- `cc-system`에는 Vooster보다 앞선 2026-04-24부터 자율주행 하네스 계열이
  있었습니다.
- 하지만 그 하네스는 `run-server.py`, `run-phases.py`, `.claude/skills` 중심이고,
  Vooster 최초 커밋의 Codex `GOAL.md` + TDD/usecase gate 구조와는 다릅니다.
- 이 가능성은 저장소만으로 확인할 수 없고 원 개발자 확인이 필요합니다.

### 높은 가능성

Vooster에서 실험된 goal/gate/finding/cycle 구조가 나중에 `cc-system`의 FCG
키트로 일반화됐다.

이유:

- 시간 순서가 맞습니다.
- `cc-system` 문서가 그렇게 설명합니다.
- `cc-system/findings-cycles-goals/`의 문서와 스크립트는 Vooster 현재 하네스의
  프로젝트 고유 요소를 제거하고 스택 중립적으로 다듬은 형태입니다.

## 이 분석이 재현 실험에 주는 의미

재현 실험에서는 두 가지를 구분해야 합니다.

1. **Vooster 최초 커밋 방식**
   - `GOAL.md`, `AGENTS.md`, `docs/usecases/`, `docs/state/`, 최소 scripts로 시작
   - 아직 `goals/`, `cycles/`, `docs/findings/` 없음
   - 작은 제품의 MVP를 처음부터 만들 때 적합

2. **cc-system FCG 키트 방식**
   - `findings`, `cycles`, `goals` 3-자산 모델
   - 이미 어떤 프로젝트가 있고, 그 위에 영속 gate 하네스를 이식할 때 적합
   - 유지보수와 장기 미션 스택 관리에 더 적합

처음 재현 실험은 Vooster 최초 커밋 방식으로 시작하고, 최소 MVP 하네스가 돈 뒤에
FCG 모델을 붙이는 편이 더 자연스럽습니다.

## 원 개발자에게 확인할 질문

이 부분은 저장소만으로는 확정할 수 없습니다.

1. Vooster 최초 커밋 전에 `cc-system`의 2026-04 하네스 계열을 알고 있었나요?
2. `GOAL.md` + `AGENTS.md` + `docs/state` + `scripts/*` 구조는 어떤 프롬프트에서
   나왔나요?
3. `findings → cycles → goals`라는 이름은 Vooster 작업 중 처음 붙인 것인가요?
4. `cc-system/findings-cycles-goals/`는 Vooster에서 직접 추출한 것인가요?
5. Vooster 최초 하네스를 다시 만든다면 FCG 키트부터 설치하겠나요, 아니면
   최초 방식처럼 `GOAL.md` 기반으로 시작하겠나요?
