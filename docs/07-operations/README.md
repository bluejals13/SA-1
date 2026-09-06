# 07-operations (Operations, Runbooks & Infrastructure Assets)

본 디렉토리는 시스템의 **실제 배포/운영 절차(Runbook), 런타임 환경 구성, 인프라 대시보드 자산, 헬스체크 및 장애 복구 매뉴얼**을 관리하는 공간입니다.

> **운영 핵심 원칙:**  
> "나중에 운영으로 확장할 곳이며, 실제 작업할 때 필요한 문서를 추가합니다."  
> 내용 없는 빈 플레이스홀더 파일 생성을 지양하며, 정형화된 운영 절차가 수립될 때 온디맨드로 문서를 작성합니다.

---

## 1. 문서 계층 간 책임 경계 (Separation of Concerns)

`07-operations`는 타 문서 계층과 다음과 같이 역할을 분리합니다:

| 문서 계층 | 주요 관심사 | 다루는 내용 | 07-operations와의 경계 |
| :--- | :---: | :--- | :--- |
| **`01-prd`** | WHAT | 기능/비기능 요구사항, 보안 스펙 | 시스템 목표 스펙이며, 실제 인프라 기동법을 다루지 않음 |
| **`02-architecture`** | WHERE/HOW | 시스템 컴포넌트, 데이터/인증 흐름 | 전체 설계 청사진이며, 런타임 운영 매뉴얼을 다루지 않음 |
| **`03-adr`** | WHY | 기술적 의사결정 이유, 트레이드오프 | 선정 배경을 다루며, 운영 자산(대시보드 등)을 다루지 않음 |
| **`04-task-progress`** | EXECUTION | 실행 TASK 추적, 검증 결과 Evidence | 작업 진행 상태를 추적하며, 지속적인 운영 Runbook을 다루지 않음 |
| **`05-knowledge`** | LEARNING | 장애/버그 트러블슈팅, 기술적 교훈 | **과거 장애 원인 분석 및 배운 점**이며, 정형화된 대응 절차를 다루지 않음 |
| **`06-ai-engineering`** | AI-FLOW | AI 협업 프로세스, Failure Cases | AI와의 협업 프로세스를 다루며, 인프라 운영을 다루지 않음 |
| **`07-operations`** | **RUN-OPS** | **서비스 배포/운영, Runbook, 인프라 대시보드 자산, 장애 대응 절차** | **프로덕션/로컬 런타임 환경 운영 및 인프라 자산의 단일 저장소** |
| **`architecture/` (루트)** | ONBOARDING | 온보딩용 아키텍처 개요 및 퀵스타트 | `02_Quick_Start.md`는 개발자 초기 셋업용 단일 소스로 유지 |

---

## 2. 개발 환경 셋업 및 퀵스타트 단일 공급원 (SSOT)

로컬 개발 환경 설정, `.env` 포트 설정, Docker Compose 기동 및 서비스 검증 가이드는 단일 진실 공급원(SSOT) 원칙에 따라 아래 문서를 공식 참조합니다:

* **공식 퀵스타트 가이드:** [`architecture/02_Quick_Start.md`](file:///C:/Users/bluej/Desktop/my2/SA-1/architecture/02_Quick_Start.md)

---

## 3. 인프라 자산 (Infrastructure Assets)

* **Grafana 대시보드:**
  * [`grafana/fullstack-infra-dashboard.json`](file:///C:/Users/bluej/Desktop/my2/SA-1/docs/07-operations/grafana/fullstack-infra-dashboard.json)
  * UID: `fullstack_infra`
  * 대상: Prometheus Self-Monitor, JVM 메트릭, 컨테이너 리소스, 노드 상태 모니터링

---

## 4. 표준 운영 Runbook 작성 템플릿

향후 백업/복구, 롤백, 헬스체크, 장애 대응 등 정형화된 운영 절차를 작성할 때는 아래 규격을 따릅니다:

```markdown
# [Runbook] {운영 절차 / 장애 대응 제목}

- **최종 수정일:** YYYY-MM-DD
- **대상 환경:** Local / Staging / Production
- **담당 역할:** DevOps / Backend / Infra Lead

### 1. 목적 및 트리거 조건 (Objective & Trigger)
* 본 매뉴얼을 실행해야 하는 상황 또는 모니터링 알람 조건

### 2. 사전 확인 사항 (Prerequisites)
* 접근 권한, 필요 CLI 도구, 백업 여부 확인

### 3. 단계별 실행 절차 (Step-by-Step Procedure)
1. 명령 1 및 예상 결과
2. 명령 2 및 예상 결과

### 4. 사후 검증 (Post-verification)
* 작업 정상 완료 여부를 확인할 수 있는 헬스체크 명령어 또는 모니터링 지표

### 5. 실패 시 롤백 방안 (Rollback Plan)
* 절차 진행 중 문제 발생 시 원상태 복구 절차
```
