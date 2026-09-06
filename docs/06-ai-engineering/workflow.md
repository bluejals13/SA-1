# [Conventions] AI 에이전트 표준 프롬프트 명령어 (AI Agent Commands)

- **Version:** 1.0.0
- **Purpose:** AI 에이전트에게 내릴 반복적인 지시(프롬프트)를 표준화하여 작업 속도와 문서 일관성을 극대화한다.

---

## 🚀 핵심 명령어 모음 (복사/붙여넣기용)

### [명령어 1] 다음 Task 진행 및 문서화 (Task & Log)
> "@task_progress.md를 확인하고 다음 번호의 Task를 진행해 줘. 코드를 수정/제안한 뒤, 빌드 및 테스트가 통과하면 `docs/rules.md` 원칙에 따라 `26-05adf-guideline/changelogs/` 경로에 작업 로그(markdown) 초안을 작성해 줘."

### [명령어 2] 시니어 코드 리뷰 (Code Review)
> "현재 터미널에서 `git status`와 `git diff`를 실행해서 내가 수정한 커밋 전 코드 내역을 확인해 줘. `03_Backend_Conventions.md` 규약을 잘 지켰는지 시니어 개발자 관점에서 리뷰하고, 완벽하면 커밋용 메시지를 제안해 줘."

### [명령어 3] 공식 문서 동기화 (Docs Sync)
> "새로운 기능이나 아키텍처 변경이 생겼어. `docs/rules.md`의 '단일 진실 공급원' 원칙에 따라 `docs/01_Architecture_and_Ports.md`, `02_Quick_Start.md`, `03_Backend_Conventions.md` 중 관련된 내용을 업데이트하고, 가이드라인 저장소와 동기화해 줘."

### [명령어 4] 트러블슈팅 장애 기록 (Troubleshooting)
> "방금 발생한 에러와 해결 과정을 `docs/troubleshooting/` 폴더에 문서로 남겨 줘. 반드시 `Symptoms -> Impact -> Detection -> Root Cause -> Resolution -> Prevention` 6단계 순서 규격을 지켜서 작성해 줘."