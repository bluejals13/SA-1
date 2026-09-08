

```txt

# FR-AUTH-001 — 사용자 인증
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/auth/security/AuthService.java
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/auth/security/AuthController.java
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/iam/user/service/UserService.java
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/iam/user/repository/UserRepository.java
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/iam/user/domain/User.java

# FR-AUTH-002 — 인증 성공
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/auth/jwt/JwtProvider.java
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/auth/security/AuthService.java

# FR-AUTH-003 — 인증 실패
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/auth/security/AuthService.java
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/auth/security/JwtAuthenticationFilter.java

# FR-AUTH-004 — 인증된 요청
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/auth/security/SecurityConfig.java
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/auth/security/UserAuthorityService.java
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/auth/security/JwtAuthenticationFilter.java

# FR-AUTH-005 — 인증 상태 갱신
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/auth/jwt/JwtProvider.java
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/auth/security/AuthService.java
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/auth/security/AuthController.java
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/auth/security/RefreshTokenRepository.java

# FR-AUTH-006 — 로그아웃
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/auth/security/AuthService.java
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/auth/security/AuthController.java
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/auth/security/RefreshTokenRepository.java
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/auth/security/TokenBlacklistService.java

```

# PRD-FUNC-001 — Authentication

# 1. 개요
## 1.1 목적

시스템 사용자가 유효한 인증 정보를 이용하여 인증 상태를 획득하고, 인증이 필요한 기능을 안전하게 이용할 수 있도록 인증 기능의 요구사항을 정의한다.

본 문서는 인증 기능이 무엇을 보장해야 하는지를 정의하며, 특정 구현 기술이나 인프라 선택을 요구사항으로 규정하지 않는다.

## 1.2 범위

본 요구사항은 다음 인증 기능을 대상으로 한다.

* 사용자 인증 요청
* 인증 성공 및 실패 처리
* 인증 상태 획득
* 인증된 요청 처리
* 인증 상태 갱신
* 로그아웃 및 인증 상태 종료

JWT의 구체적인 구조와 검증 정책, Refresh Token Rotation 및 Replay 방어, Token Invalidation 등의 세부 보안 요구사항은 각각 PRD-SEC-\* 문서에서 정의한다.

# 2. Functional Requirements
## FR-AUTH-001 — 사용자 인증

시스템은 사용자가 유효한 인증 정보를 제출했을 때 해당 사용자의 인증 가능 여부를 검증해야 한다.

인증에 사용할 수 없는 계정 또는 유효하지 않은 인증 정보로 요청한 경우 인증을 성공시켜서는 안 된다.

## FR-AUTH-002 — 인증 성공

인증에 성공한 사용자는 인증이 필요한 시스템 기능을 이용할 수 있는 인증 상태를 획득해야 한다.

인증 성공에 필요한 인증 정보는 시스템이 정의한 인증 정책을 만족해야 한다.

## FR-AUTH-003 — 인증 실패

시스템은 인증에 실패한 요청을 인증 성공 상태로 처리해서는 안 된다.

인증 실패 시 클라이언트가 인증 실패를 인지할 수 있는 일관된 결과를 제공해야 한다.

## FR-AUTH-004 — 인증된 요청

시스템은 인증이 필요한 요청에 대해 요청자의 인증 상태를 검증해야 한다.

유효한 인증 상태를 확인할 수 없는 요청은 인증이 필요한 기능에 접근할 수 없어야 한다.

## FR-AUTH-005 — 인증 상태 갱신

시스템은 기존 인증 상태를 기반으로 인증 상태를 갱신할 수 있는 기능을 제공해야 한다.

갱신 과정에서는 현재 인증 정책 및 보안 정책을 만족하지 않는 인증 상태를 다시 유효한 인증 상태로 전환해서는 안 된다.

Refresh Token의 구체적인 정책은 PRD-SEC-002-refresh-token에서 정의한다.

## FR-AUTH-006 — 로그아웃

사용자는 자신의 인증 상태를 종료할 수 있어야 한다.

로그아웃이 정상적으로 처리된 이후 시스템은 해당 사용자의 이전 인증 상태를 계속해서 유효한 상태로 취급해서는 안 된다.

인증 상태 무효화에 대한 구체적인 정책은 PRD-SEC-004-token-invalidation에서 정의한다.

## FR-AUTH-007 — 인증 상태의 독립성

한 사용자의 인증 상태 변경이 다른 사용자의 인증 상태에 영향을 주어서는 안 된다.

# 3. Functional Boundaries

본 문서에서는 다음 구현 세부사항을 요구하지 않는다.

특정 토큰 형식
특정 암호화 또는 서명 알고리즘
특정 저장소 또는 캐시 기술
특정 데이터베이스 기술
특정 웹 프레임워크
특정 인프라 구성

이러한 구현 및 설계 결정은 이후 Architecture 및 ADR 문서에서 정의한다.

# 4. Security Traceability
| Requirement | Related Security PRD |
|---|---|
| FR-AUTH-002 | PRD-SEC-001-jwt |
| FR-AUTH-004 | PRD-SEC-001-jwt, PRD-SEC-003-rbac |
| FR-AUTH-005 | PRD-SEC-002-refresh-token |
| FR-AUTH-006 | PRD-SEC-004-token-invalidation |

# 5. Acceptance Criteria Traceability
| Requirement | Acceptance Criteria |
|---|---|
| FR-AUTH-001 | AC-001-authentication |
| FR-AUTH-002 | AC-001-authentication |
| FR-AUTH-003 | AC-001-authentication |
| FR-AUTH-004 | AC-001-authentication |
| FR-AUTH-005 | AC-002-refresh-token |
| FR-AUTH-006 | AC-002-refresh-token |
| FR-AUTH-007 | AC-001-authentication |

Acceptance Criteria는 실제 API 동작과 테스트 가능성을 기준으로 세부 조건을 정의한다.

# 6. Implementation Evidence

본 PRD 자체는 구현 방법을 규정하지 않는다.

구현 및 검증 단계에서 각 요구사항과 실제 구현, 테스트 및 실행 증거를 연결한다.
```txt
PRD-FUNC-001
↓
ADR
↓
TASK
↓
Code
↓
Test
↓
Execution
↓
Evidence
```


이를 통해 인증 기능의 요구사항부터 실제 구현 및 검증 결과까지 추적 가능하도록 한다.

