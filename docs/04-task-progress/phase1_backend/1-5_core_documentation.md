# Phase 1.5: Core Documentation Reorganization & Alignment Log

- **Date:** 2026-08-26
- **Status:** COMPLETED
- **Author:** Antigravity Agent

---

## 1. 개요 (Overview)
프로젝트 전반의 파편화된 기술 문서를 통합하고 단일 진실 공급원(Single Source of Truth) 원칙에 따라 표준 문서 체계를 확립함.

---

## 2. 주요 작업 내용 (Tasks Completed)

### 1) 핵심 3대 가이드 문서 정립 (`docs/` 최상단)
* `01_Architecture_and_Ports.md`: 전체 컨테이너 토폴로지, 단일 진입점(Nginx), 포트 매핑 및 트래픽 흐름 명세.
* `02_Quick_Start.md`: Java 17, Node 20, Docker Compose 기반 원클릭 실행 및 로컬 개발 환경 가이드.
* `03_Backend_Conventions.md`: Record DTO, ApiResponse, GlobalExceptionHandler, JPA Fetch Join 최적화 표준 규약.

### 2) 문서 5대 원칙 및 도메인 문서 병합/재배치
* `rules.md`: Reference, ADR, Troubleshooting, Roadmap, Duplication 5대 원칙 정립.
* `reference/security.md`: `auth-flow`, `authentication`, `authorization`, `rbac` 통합.
* `testing/security-tests.md`: H2 Fixture, 보안 테스트 매트릭스, 결과 리포트 통합.
* `performance/k6-load-test.md`: k6 환경, 부하/스트레스 시나리오, 결과 수치(2,000+ RPS, 0% Error) 통합.
* `troubleshooting/01-redis-failure.md`: Redis 인스턴스 장애 영향도 및 복구 절차 사건 기록.

### 3) 파편화 파일 정리 및 가이드라인 저장소 동기화
* 레거시 및 중복 문서 삭제 완료.
* `26-05adf-guideline/` 내 `architecture/`, `conventions/` 디렉터리 동기화 완료.
