# TASK-SEC-003: Concurrent Refresh Atomicity

- **Priority:** `P0`
- **Type:** `Verification`
- **Status:** `TODO`
- **Current Codebase State:** `IMPLEMENTED_BUT_UNVERIFIED`
- **Target Repository:** `26-05adf`
- **Date:** 2026-09-06

---

## 1. Traceability Links

- **Related PRD:** `docs/01-prd/security/PRD-SEC-002-refresh-token.md`
- **Related ADR:** `docs/03-adr/0003-redis.md`, `docs/03-adr/0005-refresh-token-rotation.md` (Decision 3, Section 6.3)
- **Related Architecture:** `docs/02-architecture/authentication-flow.md`, `docs/02-architecture/failure-topology.md`
- **Evidence Ledger Target:** `E-004` (Refresh Token Rotation), `E-008` (Redis Lua 기반 원자적 교체 검증)

---

## 2. Problem Statement & Gap Analysis

- **ADR-0005 요구사항:**
  - 동일한 Refresh Token을 사용하여 동시에 다수의 Refresh 요청이 유입되는 경우, Redis 상태 변경의 원자성(Lua Script)을 통해 **반드시 단 1개의 요청만 Rotation에 성공**해야 한다.
  - 나머지 경쟁 요청은 이미 사용된 토큰으로 판단되어 즉시 거부(401)되어야 하며, Redis에 유효 토큰 상태가 중복 생성되어서는 안 된다.
- **현재 구현 상태 (`IMPLEMENTED_BUT_UNVERIFIED`):**
  - `RefreshTokenRepository.java`에 Lua Script(`ROTATE_SCRIPT`)가 Compare-and-Set 방식으로 구현되어 있음.
  - 그러나 기존 테스트(`RefreshTokenRepositoryTest.java`)는 `redisTemplate.execute(...)`를 Mockito로 mocking하여 가상 결과(1L/0L)만 확인한 단위 테스트임.
  - 실제 Redis 인스턴스에 다중 스레드로 동시 요청을 발생시켜 Race Condition 방지 및 원자성을 실증한 테스트 코드 및 실행 결과 Evidence가 리포지토리에 전무함 (`PR-1A1` 문서는 실제 코드가 없는 허위 CLAIM이었음).

---

## 3. Scope of Work

### 3.1 Implementation Scope
- 현재 구현된 Lua 스크립트의 파라미터 규격(`KEYS[1]`, `ARGV[1] oldJti`, `ARGV[2] newJti`, `ARGV[3] ttlMs`) 검토 및 필요시 보완.
- 추가적인 대규모 코드 작성보다는, 검증 과정에서 동시성 정합성 결함이 발견될 경우에 한해 코드 수정 진행.

### 3.2 Verification Scope
1. **Test Strategy:**
   - JUnit 5 기반의 다중 스레드 동시성 통합 테스트 작성.
   - `ExecutorService`, `CountDownLatch`(동시 출발선 보장), `CompletableFuture`를 활용하여 동일 Refresh Token으로 동시 10건의 갱신 요청 발생.
   - k6 부하 테스트는 본 TASK의 필수 AC에서 제외하며, 원자성 기능 실증에 집중 (성능 측정은 Backlog `B-002`로 관리).
2. **Test Scenarios:**
   - **단일 성공 보장:** 10건의 동시 요청 중 정확히 1건만 HTTP 200 (또는 rotate 성공 `true`)을 수신하는지 확인.
   - **경쟁 요청 거부:** 나머지 9건의 요청은 모두 실패(HTTP 401 / rotate 실패 `false`)하는지 확인.
   - **상태 정합성 검증:** 동시 요청 종료 후 Redis의 해당 사용자 키(`auth:refresh:user:{userId}`)에 저장된 토큰이 중복되지 않고, 오직 성공한 1건의 신규 JTI만 존재하는지 확인.

---

## 4. Verification Traceability Chain

| Phase | Target / Location | Verification Method | Status |
| :--- | :--- | :--- | :---: |
| **Implementation** | `26-05adf/backend/src/main/java/com/example/demo/auth/security/RefreshTokenRepository.java` | 기존 인라인 Lua 스크립트 코드 정적 점검 | `VERIFIED_CODE` |
| **Test** | `26-05adf/backend/src/test/java/com/example/demo/auth/security/ConcurrentRefreshIntegrationTest.java` | JUnit 5 다중 스레드 동시성 테스트 코드 신규 작성 | `PENDING` |
| **Execution** | `./gradlew test --tests com.example.demo.auth.security.ConcurrentRefreshIntegrationTest` | 실제 테스트 실행 및 동시 요청 결과 계측 | `PENDING` |
| **Evidence** | `Evidence Ledger v1.0.md` (E-008, E-004) 연결 | 동시 요청 성공 1건 / 실패 N-1건 측정 로그 및 Redis 상태 확인 | `PENDING` |

---

## 5. Acceptance Criteria

- [ ] JUnit 5 기반의 실제 동시성 통합 테스트 코드가 작성되어야 한다 (`ConcurrentRefreshIntegrationTest.java`).
- [ ] 동일 토큰에 대한 10개 동시 요청 환경에서 정확히 1개의 요청만 성공하고 나머지 9개는 거부되어야 한다.
- [ ] 동시 요청 종료 후 Redis 내 유효 Refresh Token이 정확히 1건으로 유지되어 중복 발급이 없어야 한다.
- [ ] 실행 결과 및 로그가 Evidence Ledger(E-008)에 연결되어 🟢 확정 상태로 증명되어야 한다.