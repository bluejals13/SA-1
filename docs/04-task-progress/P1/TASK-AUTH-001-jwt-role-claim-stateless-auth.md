# TASK-AUTH-001: JWT Role Claim / Stateless Authorization

- **Priority:** `P1`
- **Type:** `Implementation & Verification`
- **Status:** `TODO`
- **Current Codebase State:** `PARTIALLY_IMPLEMENTED`
- **Target Repository:** `26-05adf`
- **Date:** 2026-09-06

---

## 1. Traceability Links

- **Related PRD:** `docs/01-prd/security/PRD-SEC-001-jwt.md`, `docs/01-prd/security/PRD-SEC-003-rbac.md`
- **Related ADR:** `docs/03-adr/0004-jwt.md` (Decision, Section 4.1), `docs/03-adr/0006-rbac.md` (Decision 3)
- **Related Architecture:** `docs/02-architecture/authentication-flow.md`
- **Evidence Ledger Target:** `E-003` (JWT 인증), `E-006` (RBAC 권한 제어)

---

## 2. Problem Statement & Gap Analysis

- **ADR-0004 / ADR-0006 요구사항:**
  - Access Token은 Stateless 검증을 지향하며, 토큰 자체에 사용자 식별자와 최소한의 권한 정보(**Role**)를 Claim으로 포함해야 한다.
  - 보호된 API 엔드포인트 요청 시 서버 상태 저장소(MySQL/Redis)를 조회하지 않고도 JWT의 서명 및 Claim만으로 1차 인가(GrantedAuthority)를 수행할 수 있어야 한다.
- **현재 구현 상태 (`PARTIALLY_IMPLEMENTED`):**
  - `JwtProvider.createAccessToken()` 생성 시 `role` Claim이 누락되어 있음 (`userId`, `username`, `type: access`만 포함).
  - 이로 인해 `JwtAuthenticationFilter`는 매 API 요청마다 `userAuthorityService.getAuthorities(userId)`를 호출하여 MySQL DB(`userRepository.findWithRolesAndPermissionsById`)를 직접 조회하고 있음.
  - 이는 Stateless JWT 설계 결정에 위배되며, 인증 필터에서 불필요한 DB 커넥션 및 I/O 오버헤드를 유발함.

---

## 3. Scope of Work

### 3.1 Implementation Scope
1. **`JwtProvider.java` 수정:**
   - `createAccessToken(Long userId, String username, List<String> roles)` 형태로 시그니처 확장 및 `roles` 클레임 추가.
2. **`AuthService.java` 연동:**
   - 로그인 및 Refresh 시 조회된 사용자의 Role 목록을 Access Token 생성 시 전달.
3. **`JwtAuthenticationFilter.java` & `UserAuthorityService.java` 최적화:**
   - 토큰의 Claim에 Role이 존재하는 경우, DB 조회 없이 `SimpleGrantedAuthority("ROLE_" + role)`를 즉시 구성하도록 리팩토링 (세부 권한 매핑 캐시 또는 토큰 Claim 활용).

### 3.2 Verification Scope
1. **Test Strategy:**
   - 토큰 생성 단위 테스트 및 Spring Security 통합 테스트(쿼리 카운트 검증) 작성.
2. **Test Scenarios:**
   - **Claim 검증:** 생성된 Access Token 디코딩 시 `roles` 또는 `role` Claim이 올바르게 포함되어 있는지 확인.
   - **Zero DB Query 검증:** 유효한 Access Token으로 `/api/users/me` 또는 보호 API 호출 시, DB `SELECT` 쿼리가 발생하지 않고 200 OK로 인가되는지 확인.
   - **권한 격리 검증:** Role Claim에 따라 일반 사용자와 관리자 경로의 접근 권한(200 vs 403)이 정상적으로 분기되는지 확인.

---

## 4. Verification Traceability Chain

| Phase | Target / Location | Verification Method | Status |
| :--- | :--- | :--- | :---: |
| **Implementation** | `26-05adf/backend/src/main/java/com/example/demo/auth/jwt/JwtProvider.java`<br>`26-05adf/backend/src/main/java/com/example/demo/auth/security/JwtAuthenticationFilter.java` | 코드 리팩토링 및 정적 검증 | `PENDING` |
| **Test** | `26-05adf/backend/src/test/java/com/example/demo/auth/jwt/JwtRoleClaimIntegrationTest.java` | JUnit 5 통합 테스트 및 쿼리 계측 작성 | `PENDING` |
| **Execution** | `./gradlew test --tests com.example.demo.auth.jwt.JwtRoleClaimIntegrationTest` | 실제 테스트 실행 및 Hibernate SQL 로그 확인 | `PENDING` |
| **Evidence** | `Evidence Ledger v1.0.md` (E-003, E-006) 연결 | 쿼리 카운트 0건 확인 로그 및 테스트 결과 리포트 | `PENDING` |

---

## 5. Acceptance Criteria

- [ ] Access Token 생성 시 사용자의 Role 정보가 JWT Payload에 포함되어야 한다.
- [ ] 유효한 Access Token을 통한 일반 보호 API 호출 시 MySQL DB 조회가 발생하지 않아야 한다 (Zero DB Query on Authentication).
- [ ] 권한 통제(ROLE_USER, ROLE_ADMIN)가 기존과 동일하게 올바르게 동작해야 한다 (403 Forbidden 검증).
- [ ] 관련 통합 테스트가 작성되어 통과하고 Evidence Ledger에 연결되어야 한다.