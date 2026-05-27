# \_overlay

이 디렉터리는 upstream 저장소 위에 얹는 이 fork 전용 자료를 보관합니다.

upstream 소유 파일과 독립적으로 유지하세요. upstream의 문서, 목표, 스크립트,
앱 코드에서 [_overlay/](./)로 역참조를 만들지 않으면 upstream sync 때 충돌
가능성을 줄일 수 있습니다.

## 내용

- [analyses/](analyses/): 저장소, 프로세스, 아키텍처, 유지보수 분석.
  - [2026-05-27-repo-origin-analysis.md](analyses/2026-05-27-repo-origin-analysis.md):
    저장소 생성 과정, AI 에이전트 기반 빌드 흐름, 재현 청사진 분석.
  - [2026-05-27-cc-system-fcg-relation.md](analyses/2026-05-27-cc-system-fcg-relation.md):
    `greatSumini/cc-system`과 Vooster 하네스의 선후 관계 및 FCG 개념 출처 분석.
  - [2026-05-27-vooster-initial-commit-reanalysis.md](analyses/2026-05-27-vooster-initial-commit-reanalysis.md):
    Vooster 최초 커밋 `826f602`의 구성과 의미 재분석.
- [questions/](questions/): 저장소 증거만으로 확인하기 어려운 내용을 원 개발자에게
  확인하기 위한 질문 목록.
  - [2026-05-27-original-developer-questions.md](questions/2026-05-27-original-developer-questions.md):
    생성 과정, 외부 자료, 실행 환경, 의사결정 배경 확인 질문.
- `notes/`: git에 남길 가치가 있는 작업 노트.
- `artifacts/`: 보존할 가치가 있는 생성물 또는 수집 자료.
- `prompts/`: 이 fork에서만 쓰는 프롬프트와 운영 레시피.
- `patches/`: 이 fork에만 해당하는 패치 노트 또는 패치 파일.
- `sync-log/`: upstream sync와 충돌 해결 기록.

필요할 때 하위 디렉터리를 추가하세요. 추적할 내용이 없는 빈 디렉터리는
의도적으로 만들지 않습니다.
