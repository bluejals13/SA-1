# Authentication Flow

## 1. 목적
본 문서는 SA-1 Backend의 Authentication 처리 흐름을 정의한다.
component.txt에서 정의한 Authentication Component를 기준으로 Login, 인증된 요청, Token Refresh, Logout의 처리 흐름을 설명한다.
본 문서는 현재 26-05adf의 Authentication 구조를 기준으로 작성한다.

## 2. Authentication Flow Overview
Authentication의 주요 흐름은 다음과 같다.

**Login**
```text
Client
  ↓
AuthController
  ↓
AuthService
  ↓
User Authentication
  ↓
JwtProvider
  ↓
Access Token / Refresh Token
  ↓
Client
```

**인증된 API 요청은 다음과 같다.**
```text
Client
  ↓
JwtAuthenticationFilter
  ↓
JwtProvider
  ↓
Authentication
  ↓
UserAuthorityService
  ↓
SecurityContext
  ↓
Controller
  ↓
Service
```

**Refresh Token 요청은 다음과 같다.**
```text
Client
  ↓
AuthController
  ↓
AuthService
  ↓
RefreshTokenRepository
  ↓
Redis
  ↓
JwtProvider
  ↓
New Token
```

**Logout은 다음과 같다.**
```text
Client
  ↓
AuthController
  ↓
AuthService
  ↓
TokenBlacklistService
  ↓
Redis
```

## 3. Login Flow

### 3.1 Login Request
사용자가 인증 정보를 이용하여 Login을 요청한다.
```text
Client
   │
   │ Login Request
   ▼
Nginx
   │
   │ HTTP
   ▼
AuthController
```
AuthController는 외부 요청을 수신하고 Authentication 처리를 AuthService에 전달한다.

### 3.2 Authentication Processing
```text
AuthController
      │
      ▼
 AuthService
      │
      ▼
User Authentication
```
AuthService는 전달된 인증 정보와 사용자 상태를 기반으로 인증 가능 여부를 판단한다.
인증에 실패한 경우 Token을 발급하지 않는다.
```text
Invalid Authentication
        │
        ▼
Authentication Failure
        │
        └── No Token Issued
```

### 3.3 Successful Authentication
인증에 성공한 경우 시스템은 인증 상태를 획득할 수 있도록 Token을 생성한다.
```text
AuthService
    │
    ▼
JwtProvider
    │
    ├── Access Token
    │
    └── Refresh Token
```
Access Token은 인증된 요청에 사용되며 Refresh Token은 인증 상태 갱신에 사용된다.

## 4. Access Token Flow
Access Token을 이용한 보호 API 요청은 다음과 같다.
```text
Client
  │
  │ Authorization: Bearer <Access Token>
  ▼
Nginx
  │
  ▼
Backend
  │
  ▼
JwtAuthenticationFilter
```

### 4.1 JWT Extraction
JwtAuthenticationFilter는 HTTP Request에서 인증 Token을 처리한다.
```text
HTTP Request
      │
      ▼
JwtAuthenticationFilter
      │
      ▼
Extract JWT
```
유효한 JWT를 확인할 수 없는 경우 보호된 Resource에 접근할 수 없다.

### 4.2 JWT Validation
```text
JwtAuthenticationFilter
          │
          ▼
     JwtProvider
          │
          ▼
     JWT Validation
```
JWT 검증 결과에 따라 요청 처리가 분기된다.
```text
                 JWT
                  │
          ┌───────┴───────┐
          │               │
       Valid           Invalid
          │               │
          ▼               ▼
 Authentication      Authentication
   Processing           Failure
```
JWT의 구체적인 검증 정책은 PRD-SEC-001-jwt에서 정의한다.

## 5. Authentication Context
JWT 검증이 성공하면 인증된 사용자의 Identity를 Security Context에서 사용할 수 있는 상태로 구성한다.
```text
JwtProvider
     │
     ▼
User Identity
     │
     ▼
JwtAuthenticationFilter
     │
     ▼
SecurityContext
```
이후 Application Controller는 인증된 요청으로 처리할 수 있다.

## 6. Authorization Connection
Authentication 이후 Authorization이 필요한 경우 UserAuthorityService를 통해 사용자의 권한 정보를 구성한다.
```text
Authenticated User
        │
        ▼
UserAuthorityService
        │
        ├── User
        ├── Role
        └── Permission
        │
        ▼
Authorization
```
전체 요청 흐름은 다음과 같다.
```text
Client
  ↓
Nginx
  ↓
JwtAuthenticationFilter
  ↓
JwtProvider
  ↓
UserAuthorityService
  ↓
SecurityContext
  ↓
Authorization
  ↓
Protected Controller
```

## 7. Protected API Flow
인증과 인가가 필요한 API 요청은 다음 순서로 처리된다.
```text
Client
   │
   │ Access Token
   ▼
Nginx
   │
   ▼
JwtAuthenticationFilter
   │
   ▼
JwtProvider
   │
   ▼
Authentication
   │
   ▼
UserAuthorityService
   │
   ▼
Authorization
   │
   ├── Allowed ──────→ Controller
   │                       │
   │                       ▼
   │                    Service
   │
   └── Denied ───────→ Access Denied
```
인증(Authentication)과 인가(Authorization)는 서로 다른 책임으로 취급한다.

* **Authentication**: 누구인가?
* **Authorization**: 무엇을 할 수 있는가?

## 8. Refresh Token Flow
Access Token이 만료되었거나 새로운 인증 상태가 필요한 경우 Client는 Refresh Token을 이용하여 갱신을 요청한다.
```text
Client
  │
  │ Refresh Token
  ▼
Nginx
  │
  ▼
AuthController
  │
  ▼
AuthService
```

### 8.1 Refresh Token State Validation
AuthService는 Refresh Token 관련 상태를 확인한다.
```text
AuthService
     │
     ▼
RefreshTokenRepository
     │
     ▼
   Redis
```
Redis에 저장된 상태와 요청된 Refresh Token을 기준으로 갱신 가능 여부를 판단한다.

### 8.2 Refresh Success
갱신 가능한 Refresh Token인 경우 새로운 인증 상태를 생성한다.
```text
RefreshToken
     │
     ▼
Validation
     │
     ▼
JwtProvider
     │
     ├── New Access Token
     │
     └── New Refresh Token
```
Refresh Token Rotation을 사용하는 경우 기존 Refresh Token은 더 이상 동일한 인증 상태로 계속 사용되지 않도록 처리한다.
구체적인 Rotation 및 Replay 방어 정책은 PRD-SEC-002-refresh-token에서 정의한다.

### 8.3 Refresh Failure
Refresh Token이 유효하지 않거나 갱신 정책을 만족하지 않는 경우 새로운 인증 상태를 발급하지 않는다.
```text
Refresh Token
      │
      ▼
Validation
      │
      ├── Valid ─────→ New Token
      │
      └── Invalid ───→ Authentication Failure
```

## 9. Refresh Token Replay Flow
동일 Refresh Token의 재사용이 탐지되는 경우 정상적인 Token Refresh로 처리해서는 안 된다.
```text
Refresh Token
      │
      ▼
Redis State
      │
      ▼
Replay Detection
      │
      ▼
   Reject
```
Replay 방어의 구체적인 정책은 PRD-SEC-002-refresh-token에서 관리한다.

## 10. Logout Flow
사용자가 Logout을 요청한다.
```text
Client
  │
  │ Logout
  ▼
Nginx
  │
  ▼
AuthController
  │
  ▼
AuthService
```
AuthService는 인증 상태 종료에 필요한 처리를 수행한다.
```text
AuthService
     │
     ├── Refresh Token State
     │
     └── TokenBlacklistService
                    │
                    ▼
                  Redis
```

## 11. Token Invalidation Flow
Token Invalidation이 필요한 경우 무효화된 Token 상태를 관리한다.
```text
Logout / Invalidation
          │
          ▼
TokenBlacklistService
          │
          ▼
        Redis
```
이후 동일한 Token이 다시 사용되더라도 시스템은 이를 유효한 인증 상태로 취급하지 않아야 한다.
구체적인 Invalidation 정책은 관련 Security PRD 및 ADR에서 정의한다.

## 12. Authentication Failure Flow
Authentication 실패는 다음과 같이 처리된다.
```text
Request
   │
   ▼
Authentication Check
   │
   ├── Success
   │      │
   │      ▼
   │   Continue
   │
   └── Failure
          │
          ▼
   Authentication Failure
```
인증 실패 요청은 인증이 필요한 Resource에 접근할 수 없어야 한다.

## 13. Component Interaction Summary
Authentication Component 간 핵심 관계는 다음과 같다.
```text
                         ┌───────────────┐
                         │ AuthController│
                         └───────┬───────┘
                                 │
                                 ▼
                         ┌───────────────┐
                         │  AuthService  │
                         └───┬────┬───┬──┘
                             │    │   │
              ┌──────────────┘    │   └──────────────┐
              ▼                   ▼                  ▼
       ┌─────────────┐   ┌────────────────┐  ┌────────────────────┐
       │ JwtProvider │   │ RefreshToken   │  │ TokenBlacklist     │
       │             │   │ Repository     │  │ Service            │
       └──────┬──────┘   └───────┬────────┘  └──────────┬─────────┘
              │                  │                      │
              │                  ▼                      ▼
              │                Redis                  Redis
              │
              ▼
       Token Generation
```
인증된 요청에서는 다음과 같다.
```text
Request
  │
  ▼
JwtAuthenticationFilter
  │
  ▼
JwtProvider
  │
  ▼
UserAuthorityService
  │
  ▼
SecurityContext
  │
  ▼
Application
```

## 14. Authentication State Transition
인증 상태는 다음과 같이 변화한다.
```text
Unauthenticated
       │
       │ Login Success
       ▼
Authenticated
       │
       ├── Access Token Expiry
       │          │
       │          ▼
       │      Refresh
       │          │
       │          ▼
       │      Authenticated
       │
       └── Logout / Invalidation
                  │
                  ▼
             Unauthenticated
```
Refresh Token이 Replay 또는 정책 위반 상태로 판단되는 경우 정상적인 Authenticated 상태로 복귀해서는 안 된다.

## 15. PRD Traceability

| Flow | Related PRD |
| :--- | :--- |
| Login | PRD-FUNC-001-authentication |
| Authentication | PRD-FUNC-001-authentication |
| Access Token Validation | PRD-SEC-001-jwt |
| Authenticated Request | PRD-FUNC-001-authentication, PRD-SEC-001-jwt |
| User Authority | PRD-FUNC-003-iam, PRD-SEC-003-rbac |
| Refresh Token | PRD-SEC-002-refresh-token |
| Refresh Token Rotation | PRD-SEC-002-refresh-token |
| Refresh Token Replay Defense | PRD-SEC-002-refresh-token |
| Logout | PRD-FUNC-001-authentication |
| Token Invalidation | 관련 Token Invalidation PRD |
| Redis Failure | PRD-SEC-004-redis-failure |

## 16. Architecture Traceability
```text
PRD
 │
 ├── PRD-FUNC-001-authentication
 ├── PRD-SEC-001-jwt
 ├── PRD-SEC-002-refresh-token
 └── PRD-SEC-003-rbac
          │
          ▼
Authentication Flow
          │
          ▼
Component Architecture
          │
          ▼
ADR
          │
          ▼
TASK
          │
          ▼
Code
          │
          ▼
Test
          │
          ▼
Evidence
```

## 17. Implementation Boundary
본 문서는 현재 구현된 Authentication Component 간의 논리적인 처리 흐름을 정의한다.
다음 세부사항은 ADR에서 확정한다.

* JWT Claim 구성
* JWT 서명 알고리즘
* Access Token 만료 정책
* Refresh Token Rotation 방식
* Refresh Token Replay 처리 정책
* Redis Key 및 TTL 정책
* Token Blacklist 정책
* Redis 장애 시 Fail-Open / Fail-Closed 정책

Architecture에서는 흐름을 정의하고, 구체적인 기술 선택의 근거는 ADR에서 관리한다.
