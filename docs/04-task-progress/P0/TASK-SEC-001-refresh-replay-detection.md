# TASK-SEC-001: Refresh Token Replay Detection

- **Priority:** `P0`
- **Type:** `Implementation & Verification`
- **Status:** `TODO`
- **Current Codebase State:** `PARTIALLY_IMPLEMENTED`
- **Target Repository:** `apms-sr`
- **Date:** 2026-09-06

---

## 1. Traceability Links

- **Related PRD:** `docs/01-prd/security/PRD-SEC-002-refresh-token.md`
- **Related ADR:** `docs/03-adr/0005-refresh-token-rotation.md` (Decision 3, Section 6.1, 6.2)
- **Related Architecture:** `docs/02-architecture/authentication-flow.md`
- **Evidence Ledger Target:** `E-004` (Refresh Token Rotation), `E-005` (Token Blacklist / JTI 관리)

---

## 2. Problem Statement & Gap Analysis

- **ADR-0005 요구사항:**
  - Refresh Token Rotation 시 기존 사용된 Refresh Token의 JTI를 즉시 폐기하되, 단순 삭제/덮어쓰기를 하지 않는다.
  - 소진된 JTI와 `invalidatedAt`(폐기 시각)을 Redis에 기록하고, Replay Detection을 위해 설정된 TTL 동안 보존해야 한다.
  - 이미 Invalidated 상태인 JTI가 재요청될 경우 Replay Attack으로 즉각 탐지하고 갱신을 거부해야 한다.
- **현재 구현 상태 (`PARTIALLY_IMPLEMENTED`):**
  - `RefreshTokenRepository.java`는 단일 Redis 키(`auth:refresh:user:{userId}`)에 신규 JTI를 즉시 `SET`으로 덮어씀.
  - 소진된 이전 JTI 및 무효화 이력을 Redis에 별도로 기록/보존하지 않음.
  - 이로 인해 이전 토큰이 재사용되었을 때 '단순 불일치(0 반환)'와 '이미 사용된 토큰 재사용(Replay)'을 명확히 구분 및 추적할 수 없음.

---

## 3. Scope of Work

### 3.1 Implementation Scope

> **책임 범위:** 이 TASK는 JTI Invalidation 기록과 Replay 탐지(신호 발생)까지를 담당한다.  
> Replay 탐지 이후 Family 전체 Revocation은 **TASK-SEC-002**의 책임이다.

1. **Redis Key 설계 확장:**
   - 유효 Refresh Token 키: `auth:refresh:user:{userId}` (현재 유효한 JTI)
   - 소진된 Refresh Token 키: `auth:refresh:invalidated:{jti}` (값: `invalidatedAt` 타임스탬프, TTL: 토큰 잔여 수명 또는 7일)
2. **`RefreshTokenRepository.java` 로직 개선:**
   - 토큰 교체 시 기존 JTI를 `auth:refresh:invalidated:{oldJti}`로 저장하고 TTL을 설정하는 원자적 처리(회전 로직 연동).
   - 특정 JTI가 이미 `invalidated` 상태인지 확인하는 `isInvalidated(String jti)` 메서드 추가.
3. **`AuthService.java` Replay 판별 분기:**
   - 갱신 요청 시 제출된 JTI가 이미 `invalidated` 상태인 경우 `ReplayAttackException`(또는 `BadCredentialsException("REPLAY_ATTACK_DETECTED")`)을 발생시켜 호출 스택으로 신호를 전달한다.
   - **이 TASK의 구현 범위는 Exception 발생까지이며, Exception을 받아 Family 전체를 Revoke하는 처리는 TASK-SEC-002에서 담당한다.**

### 3.2 Verification Scope
1. **Test Strategy:**
   - 단위 테스트(Mock)에 의존하지 않고, 실제 Redis 인스턴스(Testcontainers 또는 로컬 Redis 6379) 환경에서 통합 테스트 수행.
2. **Test Scenarios:**
   - **정상 Rotation 검증:** 토큰 회전 성공 시 신규 토큰 발급 및 기존 JTI가 `auth:refresh:invalidated:{oldJti}`에 등록되는지 확인.
   - **Replay 탐지 검증:** 회전되어 이미 소진된 구버전 JTI로 다시 `/api/auth/refresh` 호출 시 즉각 401 및 Replay 탐지 오류 반환 확인.
   - **TTL 만료 검증:** 보존된 Invalidated JTI 키가 설정된 TTL 이후 Redis에서 자동 소멸하는지 확인.

---

## 4. Verification Traceability Chain

| Phase | Target / Location | Verification Method | Status |
| :--- | :--- | :--- | :---: |
| **Implementation** | `apms-sr/backend/src/main/java/com/example/demo/auth/security/RefreshTokenRepository.java`<br>`apms-sr/backend/src/main/java/com/example/demo/auth/security/AuthService.java` | 코드 리뷰 및 정적 검증 | `PENDING` |
| **Test** | `apms-sr/backend/src/test/java/com/example/demo/auth/security/RefreshTokenReplayIntegrationTest.java` | JUnit 5 통합 테스트 코드 작성 | `PENDING` |
| **Execution** | `./gradlew test --tests com.example.demo.auth.security.RefreshTokenReplayIntegrationTest` | 실제 테스트 실행 및 콘솔 출력 | `PENDING` |
| **Evidence** | `Evidence Ledger v1.0.md` (E-004, E-005) 연결 | 테스트 결과 로그 및 Redis Key 덤프 | `PENDING` |

---

## 5. Acceptance Criteria

- [ ] 기존 Refresh Token 회전 시 소진된 JTI가 `invalidatedAt`과 함께 Redis에 저장되어야 한다.
- [ ] 소진된 JTI의 Redis 키는 설정된 유효 수명(TTL) 동안 보존된 후 자동 삭제되어야 한다.
- [ ] 소진된 JTI를 사용한 갱신 요청은 Replay Attack으로 탐지되어 401 Unauthorized로 거부되어야 한다.
- [ ] 실제 Redis 환경에서 실행 가능한 통합 테스트 코드가 작성되고 100% 통과해야 한다.
- [ ] 테스트 실행 결과가 Evidence Ledger(E-004, E-005)에 링크되어 증거로 확인되어야 한다.