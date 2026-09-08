# TASK-SEC-002: Token Family Revocation

- **Priority:** `P0`
- **Type:** `Implementation & Verification`
- **Status:** `TODO`
- **Current Codebase State:** `NOT_IMPLEMENTED`
- **Target Repository:** `apms-sr`
- **Date:** 2026-09-06

---

## 1. Traceability Links

- **Related PRD:** `docs/01-prd/security/PRD-SEC-002-refresh-token.md`
- **Related ADR:** `docs/03-adr/0005-refresh-token-rotation.md` (Decision 3, Section 6.2)
- **Related Architecture:** `docs/02-architecture/authentication-flow.md`
- **Evidence Ledger Target:** `E-004` (Refresh Token Rotation), `E-007` (인증 관련 상태/토큰 관리)

---

## 2. Problem Statement & Gap Analysis

- **ADR-0005 요구사항:**
  - 동일 Token Family에서 Replay가 탐지된 경우, 해당 Family의 인증 상태를 **Revoked 처리**하여 이후 동일 Family의 모든 Refresh Token을 이용한 갱신을 원천 거부해야 한다.
  - Family가 Revoked 상태가 된 이후에는 동일 Family의 **최신(현재 유효하다고 간주되었던) Refresh Token을 이용한 갱신도 함께 거부**되어야 한다.
- **현재 구현 상태 (`NOT_IMPLEMENTED`):**
  - 현재 시스템에는 Token Family 개념이 없으며, Replay 또는 키 불일치 발생 시 단순히 해당 요청에 대해 `BadCredentialsException`만 던지고 종료됨.
  - 사용자의 활성 토큰(`auth:refresh:user:{userId}`)을 무효화하지 않음.
  - **결과적 취약점:** 공격자가 먼저 토큰을 가로채 갱신한 경우, 뒤늦게 정상 사용자가 이전 토큰으로 갱신을 시도해도 공격자의 세션은 안전하게 유지되는 보안 맹점이 존재함.

---

## 3. Scope of Work

### 3.1 Implementation Scope

> **책임 범위:** 이 TASK는 Token Family 모델링, Family 상태 관리, Replay 신호 수신 후 Family 전체 Revocation을 담당한다.  
> JTI Invalidation 기록(`auth:refresh:invalidated:{jti}`) 및 Replay 탐지(Exception 발생)는 **TASK-SEC-001**의 책임이다.

1. **Token Family 모델링:**
   - Refresh Token 발행 시 Token Family 식별자(`familyId`)를 JWT Claim 또는 Redis 상태 키에 포함하여 동일 로그인 세션을 하나의 Family로 연계.
   - Family 상태(`ACTIVE` / `REVOKED`)를 Redis에 별도 키(`auth:family:{familyId}:status`)로 관리.
2. **`RefreshTokenRepository.java` Family Revocation 기능 구현:**
   - `revokeFamily(String familyId)` 메서드 구현: Family 상태를 `REVOKED`로 등록하고, 해당 Family의 최신 유효 Refresh Token 키(`auth:refresh:user:{userId}`)를 즉시 삭제.
   - `isFamilyRevoked(String familyId)` 메서드 추가: 갱신 요청 진입 시 Family 상태 선행 검사.
3. **`AuthService.java` Replay 신호 수신 후 Family Revoke 연동:**
   - TASK-SEC-001의 `ReplayAttackException`을 catch하여 즉시 `revokeFamily()`를 호출하고 전체 Family 세션을 무효화.
   - **이 TASK는 "Replay 탐지" 자체를 수행하지 않으며, SEC-001로부터 Exception을 수신한 후 Family 차원의 Revocation을 실행하는 것이 핵심 책임이다.**
   - 감사 로그(`AuditAction.SECURITY_VIOLATION` 또는 `REPLAY_ATTACK`) 기록 연동.

### 3.2 Verification Scope
1. **Test Strategy:**
   - JUnit 5 기반의 Replay-and-Revoke 종단 시나리오 통합 테스트 작성.
2. **Test Scenarios:**
   - **시나리오 A (정상 사용자 선행 갱신):**  
     사용자가 RT1 ➔ RT2로 정상 갱신 완료한 뒤, 탈취자(공격자)가 구버전 RT1으로 갱신 요청 시도 ➔ Replay 감지 ➔ Token Family 전체 Revoked ➔ 정상 사용자의 RT2로 후속 갱신 시도 시에도 거절(401) 확인.
   - **시나리오 B (공격자 선행 갱신 탈취):**  
     탈취자가 RT1을 가로채 먼저 RT2_Attacker로 갱신 성공한 상태에서, 정상 사용자가 RT1으로 갱신 시도 ➔ Replay 감지 ➔ Token Family 전체 Revoked ➔ 공격자가 소유한 RT2_Attacker 역시 즉시 무효화되어 추가 갱신 차단 확인.

---

## 4. Verification Traceability Chain

| Phase | Target / Location | Verification Method | Status |
| :--- | :--- | :--- | :---: |
| **Implementation** | `apms-sr/backend/src/main/java/com/example/demo/auth/security/RefreshTokenRepository.java`<br>`apms-sr/backend/src/main/java/com/example/demo/auth/security/AuthService.java` | 코드 리뷰 및 정적 검증 | `PENDING` |
| **Test** | `apms-sr/backend/src/test/java/com/example/demo/auth/security/TokenFamilyRevocationIntegrationTest.java` | JUnit 5 시나리오 테스트 코드 작성 | `PENDING` |
| **Execution** | `./gradlew test --tests com.example.demo.auth.security.TokenFamilyRevocationIntegrationTest` | 실제 테스트 실행 및 로그 검증 | `PENDING` |
| **Evidence** | `Evidence Ledger v1.0.md` (E-004, E-007) 연결 | 테스트 실행 리포트 및 Family Revoked 로그 | `PENDING` |

---

## 5. Acceptance Criteria

- [ ] Refresh Token 발행 시 `familyId`가 생성되고 Family 상태(`ACTIVE`)가 Redis에 기록되어야 한다.
- [ ] TASK-SEC-001의 `ReplayAttackException` 수신 시 해당 Family 상태가 즉시 `REVOKED`로 전환되고, 최신 유효 Refresh Token 키(`auth:refresh:user:{userId}`)가 Redis에서 삭제되어야 한다.
- [ ] Family가 `REVOKED` 상태가 된 이후에는 동일 Family의 어떠한 Refresh Token(최신 포함)으로도 추가 갱신이 불가능해야 한다 (401 반환).
- [ ] 탈취자 선행 갱신(시나리오 B) 및 정상 사용자 선행 갱신(시나리오 A) 2개 시나리오에 대한 통합 테스트 코드가 작성되고 100% 통과해야 한다.
- [ ] 실행 결과 및 콘솔 로그가 Evidence Ledger(E-004, E-007)에 연결되어 증빙되어야 한다.