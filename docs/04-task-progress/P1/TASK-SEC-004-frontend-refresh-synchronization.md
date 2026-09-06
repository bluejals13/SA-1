# TASK-SEC-004: Frontend Refresh Synchronization Verification

- **Priority:** `P1`
- **Type:** `Verification`
- **Status:** `TODO`
- **Current Codebase State:** `IMPLEMENTED_BUT_UNVERIFIED`
- **Target Repository:** `26-05adf` (Frontend)
- **Date:** 2026-09-06

---

## 1. Traceability Links

- **Related PRD:** `docs/01-prd/security/PRD-SEC-002-refresh-token.md`
- **Related ADR:** `docs/03-adr/0005-refresh-token-rotation.md` (Section 4.2 Responsibility Separation)
- **Related Architecture:** `docs/02-architecture/authentication-flow.md`
- **Evidence Ledger Target:** `E-004` (Refresh Token Rotation), 프론트엔드 인증 상태 안정성

---

## 2. Problem Statement & Gap Analysis

- **ADR-0005 책임 분리 원칙:**
  - **Frontend 책임 (이 TASK의 범위):** 여러 비동기 요청에서 동시에 401 Unauthorized가 발생하더라도, 서버로 중복 Refresh 요청이 쏟아지지 않도록 Promise 공유(Single Flight) 또는 Mutex를 통해 **갱신 요청을 클라이언트 단에서 직렬화 및 예방**한다. 이는 UX 안정성 및 서버 부하 감소를 위한 최적화이며, 보안의 최종 보장 수단이 아니다.
  - **Backend 책임 (TASK-SEC-003의 범위):** Frontend의 동기화 실패나 예외 상황으로 중복 요청이 유입되더라도, Redis Lua Script 기반 원자적 처리를 통해 단 1건의 Rotation만 허용하여 **보안 무결성을 최종 보장**한다. Frontend 동기화는 Backend 보안 보장의 전제 조건이 아니다.
- **현재 구현 상태 (`IMPLEMENTED_BUT_UNVERIFIED`):**
  - `frontend/src/api/http.ts` (lines 50~113)에 `refreshPromise`를 활용한 Single Flight 및 401 인터셉터가 이미 충실히 작성되어 있음.
  - **단, 프론트엔드 프로젝트(`package.json`)에 테스트 프레임워크(Vitest/Jest) 설정이 전무**하며, 동시 401 발생 시 재발급 API가 정확히 1회만 호출되고 후속 요청이 정상 완료되는지 실증한 테스트 코드 및 실행 결과 Evidence가 없음.

---

## 3. Scope of Work

### 3.1 Implementation Scope
- `frontend`에 경량 단위 테스트 환경(Vitest, JSDOM) 최소 설정 추가 (또는 모의 E2E 스크립트 작성).
- `http.ts` 자체는 코드가 이미 존재하므로, 검증 도중 엣지 케이스(Timeout, 거부 시 큐 정리 등) 결함이 발견된 경우에만 코드 수정.

### 3.2 Verification Scope
1. **Test Strategy:**
   - Vitest 기반 모의 fetch/인터셉터 테스트 스위트 작성 (`http.spec.ts`).
2. **Test Scenarios:**
   - **Single Flight 검증:** 3개의 비동기 `http.get()` 요청이 거의 동시에 401 에러를 수신했을 때, `/api/auth/refresh` 네트워크 호출이 정확히 1회만 실행되는지 확인.
   - **토큰 전파 및 재시도 검증:** 갱신 완료 후 3개의 원래 요청이 신규 Access Token 헤더를 부착하여 모두 성공(200 OK)으로 복구되는지 확인.
   - **갱신 실패 시 처리 검증:** Refresh 호출 자체가 실패(401)할 경우 대기 중인 모든 요청이 일관되게 에러를 수신하고 로그아웃 처리되는지 확인.

---

## 4. Verification Traceability Chain

| Phase | Target / Location | Verification Method | Status |
| :--- | :--- | :--- | :---: |
| **Implementation** | `26-05adf/frontend/src/api/http.ts` | 기존 Single Flight 코드 정적 점검 | `VERIFIED_CODE` |
| **Test** | `26-05adf/frontend/src/api/__tests__/http.spec.ts` | Vitest 기반 동시 401 가로채기 테스트 코드 작성 | `PENDING` |
| **Execution** | `npm run test` (또는 `npx vitest run`) | 프론트엔드 테스트 스위트 실행 | `PENDING` |
| **Evidence** | `Evidence Ledger v1.0.md` 연결 | 테스트 실행 결과 리포트 및 호출 카운트(1회) 로그 | `PENDING` |

---

## 5. Acceptance Criteria

- [ ] 프론트엔드에 실행 가능한 테스트 러너(Vitest 등)가 구성되어야 한다.
- [ ] 다수의 동시 401 발생 시 `/api/auth/refresh` 호출이 정확히 1회만 수행되는 실증 테스트가 통과해야 한다.
- [ ] 갱신 성공 후 중단되었던 원래 요청들이 신규 토큰으로 정상 재수행됨이 검증되어야 한다.
- [ ] 테스트 실행 결과가 Evidence Ledger에 연결되어 증명되어야 한다.