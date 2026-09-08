


```txt

# SEC-JWT-001 Access Token 발급
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/auth/jwt/JwtProvider.java
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/auth/security/AuthService.java

# SEC-JWT-002 Token 식별
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/auth/jwt/JwtProvider.java

# SEC-JWT-003 무결성/진위 검증
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/auth/jwt/JwtProvider.java
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/auth/security/JwtAuthenticationFilter.java

# SEC-JWT-004 만료 검증
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/auth/jwt/JwtProvider.java

# SEC-JWT-005 Token Type 검증
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/auth/security/JwtAuthenticationFilter.java

# SEC-JWT-006 사용자 식별
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/auth/jwt/JwtProvider.java
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/auth/security/JwtAuthenticationFilter.java

# SEC-JWT-007 인증 정보 구성
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/auth/security/JwtAuthenticationFilter.java

# SEC-JWT-008 현재 권한 정보 연계
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/auth/security/UserAuthorityService.java
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/auth/security/JwtAuthenticationFilter.java

# SEC-JWT-009 유효하지 않은 Token 거부
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/auth/jwt/JwtProvider.java
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/auth/security/JwtAuthenticationFilter.java

# SEC-JWT-010 인증 실패 격리
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/auth/security/SecurityConfig.java
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/auth/security/JwtAuthenticationFilter.java


```



# PRD-SEC-001 — JWT Authentication Token

# 1. 목적

시스템은 인증 토큰의 진위, 무결성 및 유효기간을 검증하여
위조되거나 변조되었거나 유효하지 않은 토큰이 인증 상태로
사용되지 않도록 해야 한다.

# 2. 범위

- Access Token 발급
- Access Token 식별
- Token 무결성 및 진위 검증
- Token 만료 검증
- Token 유형 검증
- 인증 요청에서 Access Token 검증
- 검증된 사용자에 대한 인증 정보 구성

Refresh Token의 Rotation 및 Replay 방어는
PRD-SEC-002-refresh-token에서 정의한다.

Token 무효화 및 로그아웃 이후 Token 사용 방지는
PRD-SEC-004-token-invalidation에서 정의한다.

# 3. Security Requirements

## SEC-JWT-001 — Access Token 발급

시스템은 인증에 성공한 사용자에게 인증 상태를 유지하기 위한
Access Token을 발급할 수 있어야 한다.

## SEC-JWT-002 — Token 식별

각 Access Token은 개별 토큰을 식별할 수 있는 식별 정보를
가져야 한다.

## SEC-JWT-003 — Token 무결성 검증

시스템은 인증 요청에 사용된 Token의 무결성과 진위를
검증해야 한다.

검증에 실패한 Token은 인증된 요청으로 처리해서는 안 된다.

## SEC-JWT-004 — Token 만료 검증

시스템은 Token의 유효기간을 검증해야 한다.

만료된 Token은 유효한 인증 상태를 생성하는 데 사용할 수 없어야 한다.

## SEC-JWT-005 — Token 유형 검증

시스템은 인증 요청에 사용된 Token이 해당 요청에 허용된
인증 Token 유형인지 검증해야 한다.

Refresh Token 등 다른 목적의 Token을 Access Token 대신
인증 요청에 사용할 수 없어야 한다.

## SEC-JWT-006 — 사용자 식별

유효한 Access Token으로부터 인증 대상 사용자를 식별할 수
있어야 한다.

## SEC-JWT-007 — 인증 정보 구성

시스템은 검증된 Access Token의 사용자 식별 정보를 기반으로
인증 상태를 구성해야 한다.

## SEC-JWT-008 — 권한 정보 연계

인증 상태를 구성할 때 시스템은 해당 사용자의 현재 권한 정보를
적용해야 한다.

Token 자체에 포함된 정보만을 신뢰하여 현재 권한 상태를
확정해서는 안 된다.

## SEC-JWT-009 — 유효하지 않은 Token 거부

다음과 같은 Token은 인증된 요청으로 처리해서는 안 된다.

- 변조된 Token
- 서명 검증에 실패한 Token
- 만료된 Token
- 지원하지 않는 형식의 Token
- 허용되지 않은 Token 유형
- 필수 식별 정보가 유효하지 않은 Token

## SEC-JWT-010 — 인증 실패 격리

Token 검증에 실패한 요청은 인증된 Security Context를
생성해서는 안 된다.

# 4. Implementation Boundary

본 문서는 다음 구현 방법을 요구하지 않는다.

- 특정 JWT 라이브러리
- 특정 서명 알고리즘
- 특정 Framework
- 특정 Token 저장소
- 특정 Cache/Database
- 특정 Filter 구현 방식

구체적인 기술 선택은 Architecture 및 ADR에서 정의한다.

# 5. Traceability
```txt
SEC-JWT-001
 → PRD-FUNC-001 FR-AUTH-002

SEC-JWT-003
 → PRD-FUNC-001 FR-AUTH-003, FR-AUTH-004

SEC-JWT-004
 → PRD-FUNC-001 FR-AUTH-004

SEC-JWT-005
 → PRD-FUNC-001 FR-AUTH-004

SEC-JWT-006
 → PRD-FUNC-001 FR-AUTH-004

SEC-JWT-008
 → PRD-FUNC-001 FR-AUTH-004
 → PRD-SEC-003-rbac

SEC-JWT-009
 → PRD-FUNC-001 FR-AUTH-003, FR-AUTH-004

SEC-JWT-010
 → PRD-FUNC-001 FR-AUTH-003, FR-AUTH-004
```
