# [Log] Phase 1-2: JWT Token 검증 및 Redis Blacklist 연동 최적화

- **Date:** 2026-08-26
- **Domain:** Auth, Security, JWT, Redis Blacklist
- **Author:** AI Agent (Antigravity) & bluejck10113

## 1. 목적 (Purpose)
* **JWT 파싱 & 서명 검증 오버헤드 절감:** 매 요청 시 발생하던 `Jwts.parserBuilder().build()` 객체 중복 생성 및 필터 내 이중 파싱(validateToken + parseClaims)을 단일화하여 CPU 및 메모리 오버헤드 최소화.
* **보안 취약점 보완 (Token Type 검증):** Authorization 헤더를 통한 요청 시 `type: access` 클레임 검증을 추가하여 Refresh Token을 통한 API 접근 우회 차단.
* **Redis 장애 내구성 및 Blacklist 연동 안정성 강화:** `RedisSystemException`뿐 아니라 네트워크 타임아웃, 커넥션 풀 고갈 등 Spring Data의 `DataAccessException` 전체를 일관되게 핸들링하여 서비스 가용성(503 Service Unavailable 및 로깅) 보장.
* **코드 품질 및 불필요 I/O 제거:** 콘솔 표준 출력(`System.out.println`)을 제거하고 SLF4J 로깅으로 전환하여 I/O 블로킹 방지 및 설정 정리.

## 2. 변경된 파일 (Modified Files)
* `backend/src/main/java/com/example/demo/auth/jwt/JwtProvider.java`
* `backend/src/main/java/com/example/demo/auth/security/TokenBlacklistService.java`
* `backend/src/main/java/com/example/demo/auth/security/JwtAuthenticationFilter.java`
* `backend/src/main/java/com/example/demo/auth/security/UserAuthorityService.java`
* `backend/src/main/java/com/example/demo/auth/security/SecurityConfig.java`
* `backend/src/test/java/com/example/demo/auth/security/TokenBlacklistServiceTest.java` (신규)
* `backend/src/test/java/com/example/demo/auth/security/JwtAuthenticationFilterTest.java` (신규)

## 3. 핵심 변경 사항 (Key Changes)

### 1) `JwtProvider.java`
* **`JwtParser` 사전 빌드 및 캐싱:** `@PostConstruct init()` 시점에 `JwtParser`를 1회 빌드해 필드로 캐싱하여 재사용.
* **예외 세분화 및 로깅 개선:** `ExpiredJwtException`, `MalformedJwtException`, `UnsupportedJwtException`, `IllegalArgumentException` 등 JWT 파싱 실패 원인별 경고 로그 분류.

### 2) `TokenBlacklistService.java`
* **광범위한 Redis 예외 처리:** `DataAccessException`을 포괄적으로 catch하여 Redis 연결/타임아웃 장애 시 일관된 `RedisUnavailableException` 발생 및 에러 로그 기록.
* **JTI Null/Blank 방어:** 잘못된 키에 대한 Redis 요청 사전 방지.

### 3) `JwtAuthenticationFilter.java`
* **이중 파싱 제거 (단일 검증 흐름):** 기존 `validateToken` 1회 + `parseClaims` 1회 수행하던 2중 서명 검증을 `parseClaims` 단일 호출 및 예외 캐치 구조로 최적화.
* **Token Type 검증 (`type == "access"`):** Access Token 전용 검증을 통해 Refresh Token 도용 접근 원천 차단.
* **검증 순서 최적화:** `Claims 파싱` ➔ `Token Type 검증` ➔ `Blacklist 조회` ➔ `Subject 파싱 및 권한 조회` 순으로 변경하여 블랙리스트 토큰의 불필요한 DB/객체 연산 조기 차단.

### 4) `UserAuthorityService.java` & `SecurityConfig.java`
* 디버깅용 `System.out.println` 콘솔 출력을 `log.debug` 및 `log.warn`으로 전환하여 I/O 성능 개선.
* `SecurityConfig`의 미사용 의존성 필드 정리 및 표준 예외 로깅 적용.

## 4. 테스트 및 검증 결과 (Verification)
* **단위 테스트 추가:** `TokenBlacklistServiceTest`, `JwtAuthenticationFilterTest` 작성 완료.
* **기존 및 신규 테스트 전체 통과:** `./gradlew test` 실행 결과 전체 테스트 통과 (Build SUCCESSFUL).
