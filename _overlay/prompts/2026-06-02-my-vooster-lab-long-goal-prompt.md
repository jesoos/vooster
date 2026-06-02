# my-vooster-lab 장시간 Slash Goal 프롬프트

날짜: 2026-06-02
대상 저장소: [/home/ubuntu/workspace/personal/my-vooster-lab](/home/ubuntu/workspace/personal/my-vooster-lab)

## 사용법

```bash
cd /home/ubuntu/workspace/personal/my-vooster-lab
codex
```

그 다음 아래 블록을 첫 입력으로 붙여 넣습니다.

```text
/goal AGENTS.md, HANDOFF.md, GOAL.md, DECISIONS.md, SOURCE_REFS.md, cycles/260529-02-hours-scale-cli-mvp.md, goals/2-hours-scale-cli-mvp.md를 먼저 읽으세요.

이 저장소는 Vooster식 findings -> cycles -> goals autonomous build harness 재현 실험을 위한 lab repository입니다.
이번 목표는 짧은 smoke goal이 아니라 goals/2-hours-scale-cli-mvp.md 를 완료하는 몇 시간 규모의 FCG 실험입니다.

반드시 다음 방식으로 진행하세요.

1. 먼저 git status --short --branch 와 git config --get core.hooksPath 를 확인하세요.
2. core.hooksPath 가 scripts/hooks 가 아니면 git config core.hooksPath scripts/hooks 만 실행하세요.
3. bash scripts/completion-check.sh 를 실행해 active goal을 갱신하세요.
4. active goal이 goals/2-hours-scale-cli-mvp.md 인지 확인하세요.
5. 매 iteration 시작 시 bash scripts/diagnose.sh 와 bash scripts/next-task.sh 를 실행하세요.
6. scripts/next-task.sh 가 출력한 Goal 2의 한 task만 수행하세요.
7. 제품 동작 변경은 RED -> GREEN -> REFACTOR 순서로 진행하세요.
8. commit은 explicit path만 stage해서 git add <paths> && git commit -m "<message>" 형태로 수행하세요.
9. git add . 또는 git add -A 는 사용하지 마세요.
10. 현재 사용자 지시가 "훅을 실행하지 말라"이면 git commit 또는 git push 에 --no-verify를 사용하세요.
11. bash scripts/completion-check.sh 가 통과하기 전에는 push하지 마세요.
12. completion-check가 통과하고 working tree가 clean이면 git push 하세요.

목표는 빠르게 끝내는 것이 아니라 FCG 루프를 충분히 관찰하는 것입니다.
게이트를 약화하지 말고, Goal 2가 요구하는 여러 RED/GREEN commit, persistence, CLI e2e, state update를 순서대로 완료하세요.
```

## 기록 이유

이 프롬프트는 랩 저장소의
[SLASH_LONG_GOAL_PROMPT.md](/home/ubuntu/workspace/personal/my-vooster-lab/SLASH_LONG_GOAL_PROMPT.md)를
원본 분석 fork에도 보존하기 위한 사본입니다.

랩 저장소는 실험 실행 장소이고, 이 `_overlay/prompts/` 문서는 왜 그런 프롬프트를
쓰는지 추적하기 위한 분석 저장소의 기록입니다.
