# Backlog (미확정 과제 및 조건부 후속 과제)

본 문서는 아직 실행 TASK로 확정하지 않았거나, 측정 결과 및 조건부 트리거가 발생할 때 진행할 후보 항목을 관리합니다.

---

## 1. 조건부 재검토 항목 (Conditional Reconsideration)

### B-001: Grace Period 기반 Refresh Token Rotation 도입 검토
- **관련 ADR:** ADR-0005 (Section 7.1)
- **보류 사유:**
  - Strict RTR을 우선 적용하여 보안성을 극대화하기로 결정함.
  - 다양한 클라이언트 환경에서 프론트엔드 동기화 실패로 인한 정상 사용자의 잦은 강제 로그아웃이 **실제 측정 또는 런타임 로그로 확인되는 경우에만** 재검토(Option C)하기로 합의됨.
  - 현재 단계에서는 과도한 사전 복잡성(Over-engineering)으로 판단하여 Backlog로 유지.
- **트리거 조건:** 운영 환경 모니터링 시 동시 요청으로 인한 401 로그아웃 발생률이 허용치 초과 시.

---

## 2. 성능 및 부하 측정 후속 과제 (Performance & Load Testing)

### B-002: k6 기반 대규모 동시 Refresh 부하 테스트
- **관련 ADR:** ADR-0003, ADR-0005
- **보류 사유:**
  - `TASK-SEC-003`에서 JUnit 기반의 실제 동시성 통합 테스트로 Redis Lua 원자성 실증을 우선 완료함.
  - k6 기반 대규모 VUs(Virtual Users) 동시 요청 부하 테스트는 기능적 원자성 검증이 완료된 후, Redis Throughput/Latency 벤치마크가 필요한 시점에 별도 성능 검증 TASK로 분리하여 진행.
- **트리거 조건:** 백엔드 원자성 기능 검증 완료 후, Nginx/Backend 성능 튜닝 단계 진입 시.

---

## 3. 인프라 확장 후보 (Infra Scalability)

### B-003: Redis Sentinel / Cluster HA 구성
- **관련 ADR:** ADR-0003 (Section 3)
- **보류 사유:**
  - 현재 시스템 규모 및 리소스 환경(단일 Docker Compose)에서는 Single Redis Instance가 최적임.
  - Sentinel/Cluster는 인프라 복잡도를 크게 증가시키므로 단일 장애점(SPOF) 대응이 필수적인 상용 배포 단계로 연기.
- **트리거 조건:** 프로덕션 다중 노드 배포 아키텍처 확정 시.