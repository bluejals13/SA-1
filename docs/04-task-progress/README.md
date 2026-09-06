# 04-task-progress (Task Execution & Traceability)

본 디렉토리는 프로젝트의 **PRD ➔ ADR ➔ TASK ➔ Implementation ➔ Verification ➔ Evidence** 추적 계층을 관리하는 실행 작업 단위 저장소입니다.

---

## 1. 운영 원칙

1. **Priority 기반 분류 (기술 영역별 분류 지양)**
   - 디렉토리는 Backend, Frontend, Infra 등 기술 스택이 아니라 **우선순위(P0, P1, P2)**를 기준으로 분리합니다.
2. **구현과 검증의 분리 (No False Claims)**
   - "코드가 존재한다"는 사실만으로 기능이 완료되었다고 판단하지 않습니다.
   - 모든 TASK는 **Implementation(코드)**과 **Verification(테스트 ➔ 실행 ➔ Evidence)**을 엄격히 구분하여 추적합니다.
   - 이미 구현된 기능에 검증이 없는 경우(`IMPLEMENTED_BUT_UNVERIFIED`), 불필요한 구현 TODO 대신 **검증(Verification) TASK**로 정의합니다.
3. **Evidence Ledger 연결**
   - TASK 문서 내에 실행 로그를 장황하게 복사하지 않고, `Evidence Ledger`의 증거 ID 및 위치와 연결합니다.
   - `TASK` ➔ `Evidence ID` ➔ `Evidence Ledger` ➔ `실제 Test / Execution Log / PR`
4. **`completed/` 이동 기준**
   - 단순 코드 작성 완료가 아닌 다음 6대 조건을 **모두 충족**해야만 `completed/` 디렉토리로 이동할 수 있습니다:
     1. Implementation 완료
     2. Acceptance Criteria 100% 충족
     3. 재현 가능한 테스트 코드 작성 및 통과
     4. 실제 환경(또는 통합 환경)에서의 Execution 완료
     5. Evidence Ledger에 증거 항목 등록 및 상태 🟢 확정
     6. 관련 Git Commit / PR 해시 연결
5. **`backlog.md` 운영**
   - 아직 실행 TASK로 확정되지 않았거나, 측정 결과에 따라 후속으로 진행할 항목만 관리합니다.

---

## 2. 디렉토리 구조

```text
docs/04-task-progress/
├── README.md               # 본 운영 가이드
├── backlog.md              # 실행 미확정 후보 및 조건부 후속 과제
├── P0/                     # 시스템 무결성 및 핵심 보안 필수 과제
│   ├── TASK-SEC-001-refresh-replay-detection.md
│   ├── TASK-SEC-002-token-family-revocation.md
│   └── TASK-SEC-003-concurrent-refresh-atomicity.md
├── P1/                     # 최적화, 정합성 및 보조 검증 과제
│   ├── TASK-SEC-004-frontend-refresh-synchronization.md
│   └── TASK-AUTH-001-jwt-role-claim-stateless-auth.md
├── P2/                     # 편의성 개선 및 장기 과제
├── completed/              # 구현 + 검증 + Evidence 완료된 확정 TASK
└── changelog/              # 과거 Phase 작업 이력 보존 (읽기 전용)
    ├── phase1_backend/     # Phase 1 Backend 최적화 이력
    ├── phase2_frontend/    # Phase 2 Frontend 최적화 이력
    └── phase3_infra/       # Phase 3 인프라 이력
```

---

## 3. TASK 라이프사이클

```text
[backlog.md] 
     │ (요구사항 및 우선순위 확정)
     ▼
[P0 / P1 / P2] 
  ├── Status: TODO
  ├── Status: IN_PROGRESS (Implementation 중)
  ├── Status: VERIFYING (Test 작성 및 Execution 단계)
  │
  │ (Acceptance Criteria + Test + Execution + Evidence Ledger 완료)
  ▼
[completed/]
```

---

## 4. 책임 분리 원칙 (Responsibility Separation)

본 추적 체계에서 Authentication & Concurrency 관련 책임은 다음과 같이 엄격히 분리됩니다:

- **Frontend:** 중복 Refresh 요청 예방 및 Single Flight 직렬화 (클라이언트 최적화)
- **Backend:** 동일 Refresh Token에 대해 단 1회의 Rotation만 성공하도록 보장 및 Replay 탐지 (최종 보안 보장)
- **Redis:** Refresh Token 상태 전이 및 Invalidation의 원자적 처리 보장 (원자성 인프라)