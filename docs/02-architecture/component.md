# Component Architecture

## 1. 목적
본 문서는 SA-1 Backend Container 내부의 주요 Application Component와 Component 간 의존 관계를 정의한다.

container.txt에서 정의한 Backend Container를 내부 Component 수준으로 구체화하며, 현재 26-05adf의 실제 Package 및 Class 구조를 기준으로 작성한다.

본 문서에서는 구현에 존재하지 않는 Component를 임의로 추가하지 않는다.

## 2. Backend Component Structure
Backend는 크게 Authentication과 IAM 영역으로 구성된다.

```text
Backend
│
├── auth
│   ├── jwt
│   └── security
│
└── iam
    ├── user
    ├── role
    ├── permission
    ├── admin
    └── menu
```

현재 Repository에서도 auth 내부에 jwt, security Package가 존재하고, iam 내부에는 admin, menu, permission, role, user Package가 존재한다.

## 3. Authentication Components
Authentication 영역의 주요 Component 관계는 다음과 같다.

```text
                 ┌──────────────────┐
                 │  AuthController  │
                 └────────┬─────────┘
                          │
                          ▼
                 ┌──────────────────┐
                 │    AuthService   │
                 └────┬─────┬─────┬─┘
                      │     │     │
              ┌───────┘     │     └──────────┐
              ▼             ▼                ▼
       ┌────────────┐ ┌──────────────┐ ┌────────────────────┐
       │ JwtProvider│ │RefreshToken  │ │TokenBlacklistService│
       │            │ │Repository    │ │                    │
       └────────────┘ └──────┬───────┘ └────────────────────┘
                             │
                             ▼
                           Redis
```

Authentication 관련 실제 Package는 다음과 같이 구분된다.

```text
auth
├── jwt
│   └── JwtProvider
│
└── security
    ├── AuthController
    ├── AuthService
    ├── JwtAuthenticationFilter
    ├── SecurityConfig
    ├── RefreshTokenRepository
    ├── TokenBlacklistService
    └── UserAuthorityService
```

## 4. AuthController
AuthController는 외부 Client의 Authentication 관련 요청을 수신하는 Controller Component다.

주요 책임은 다음과 같다.

* 인증 관련 API Endpoint 제공
* 인증 요청 전달
* Token 관련 요청 전달
* 로그아웃 및 인증 상태 종료 요청 전달

Controller는 Authentication의 정책이나 Token 검증 로직을 직접 담당하지 않고 Service 및 Security Component에 책임을 위임한다.

## 5. AuthService
AuthService는 Authentication 관련 Application Logic을 담당한다.

주요 책임 영역은 다음과 같다.

```text
AuthService
├── User Authentication
├── Access Token 처리
├── Refresh Token 처리
├── Token Rotation 관련 처리
└── Logout / Invalidation 관련 처리
```

구체적인 Token 보안 정책은 다음 PRD 및 ADR에서 정의한다.

* PRD-SEC-001-jwt
* PRD-SEC-002-refresh-token
* PRD-SEC-004-redis-failure

## 6. JwtProvider
JwtProvider는 JWT 생성 및 검증을 담당하는 Component다.

구조적으로 다음과 같이 연결된다.

```text
AuthService
     │
     ▼
JwtProvider
     │
     ├── JWT 생성
     └── JWT 검증
```

또한 인증된 요청에서는 JwtAuthenticationFilter가 JwtProvider를 사용하여 요청의 JWT를 처리한다.

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
Authentication
```

JWT의 구체적인 Claim, 서명 및 검증 정책은 PRD-SEC-001-jwt 및 관련 ADR에서 정의한다.

## 7. JwtAuthenticationFilter
JwtAuthenticationFilter는 보호된 HTTP 요청에서 Authentication 정보를 구성하기 위한 Security Component다.

```text
HTTP Request
     │
     ▼
JwtAuthenticationFilter
     │
     ▼
JwtProvider
     │
     ▼
Authenticated Principal
     │
     ▼
Security Context
```

Filter는 Controller보다 앞선 Security 처리 단계에서 동작하며, 인증되지 않은 보호 요청이 Application Component까지 정상적으로 도달하지 않도록 한다.

## 8. SecurityConfig
SecurityConfig는 Backend의 Spring Security 구성을 담당한다.

주요 책임은 다음 영역이다.

* Security Filter Chain 구성
* Endpoint 접근 정책 구성
* 인증 및 인가 처리 연결
* Security Component 등록 및 구성

개별 권한 데이터의 관리는 IAM Component에서 담당하고, SecurityConfig는 해당 인증/인가 구조를 Security Framework에 연결한다.

## 9. RefreshTokenRepository
RefreshTokenRepository는 Refresh Token 관련 상태를 저장하고 조회하기 위한 Component다.

```text
AuthService
     │
     ▼
RefreshTokenRepository
     │
     ▼
    Redis
```

Refresh Token의 저장 방식 및 Rotation 정책은 구현 Component와 별도로 PRD/ADR에서 관리한다.

관련 요구사항:
* PRD-SEC-002-refresh-token

## 10. TokenBlacklistService
TokenBlacklistService는 무효화된 Token의 상태를 관리하는 Component다.

```text
Logout / Invalidation
        │
        ▼
TokenBlacklistService
        │
        ▼
      Redis
```

Token Invalidation의 정책 및 Redis 장애 상황의 보안 처리는 별도의 Security PRD와 Failure Architecture에서 정의한다.

## 11. UserAuthorityService
UserAuthorityService는 인증된 사용자의 권한 정보를 Security Layer에서 사용할 수 있도록 구성하는 Component다.

주요 관계는 다음과 같다.

```text
Authenticated User
       │
       ▼
UserAuthorityService
       │
       ├── User
       ├── Role
       └── Permission
```

이를 통해 Authentication과 Authorization을 연결한다.

```text
Authentication
      │
      ▼
UserAuthorityService
      │
      ▼
Authorization
```

RBAC 관련 상세 정책은 PRD-SEC-003-rbac에서 정의한다.

## 12. IAM Components
IAM은 Backend 내부에서 다음 영역으로 분리된다.

```text
iam
├── user
├── role
├── permission
├── admin
└── menu
```

실제 Repository에서도 이 5개 Package가 별도 영역으로 존재한다.

## 13. User Components
User 영역은 일반적인 사용자 조회 및 관리 기능을 담당한다.

```text
UserController
      │
      ▼
UserService
      │
      ▼
UserRepository
      │
      ▼
    User
```

각 Component의 책임은 다음과 같다.

| Component | Responsibility |
| :--- | :--- |
| UserController | User API Endpoint |
| UserService | User Application Logic |
| UserRepository | User Persistence Access |
| User | User Domain Model |

User 관련 기능 요구사항은 PRD-FUNC-002-user와 연결된다.

## 14. Role Components
Role 관리 영역은 다음 구조를 가진다.

```text
RoleAdminController
        │
        ▼
RoleAdminService
        │
        ├── Role
        │
        └── RolePermissionService
```

RolePermissionService는 Role과 Permission의 관계를 관리한다.

```text
Role
 │
 ▼
RolePermissionService
 │
 ▼
Permission
```

## 15. Permission Components
Permission 관리 영역은 다음과 같다.

```text
PermissionAdminController
          │
          ▼
PermissionAdminService
          │
          ▼
      Permission
```

Permission은 시스템의 세부 접근 권한을 표현하고 Role과 결합하여 RBAC 구조를 구성한다.

## 16. User-Role Components
관리자 영역에서 User와 Role의 관계를 관리한다.

```text
UserAdminController
       │
       ▼
UserAdminService
       │
       └── User management

UserAdminController
       │
       ▼
UserRoleService
       │
       ▼
User ↔ Role
```

User와 Role의 연결 관계는 Authorization에서 사용되는 권한 구성의 기반이 된다.

## 17. Menu Components
Menu 관리 영역은 다음 구조를 가진다.

```text
MenuAdminController
       │
       ▼
MenuAdminService
       │
       ▼
     Menu
```

Menu는 IAM 관리 영역의 구성요소로 취급한다.

Menu와 Permission의 구체적인 관계가 별도의 설계 결정으로 필요한 경우 ADR에서 정의한다.

## 18. IAM Component Relationship
IAM 전체 관계는 다음과 같이 표현한다.

```text
                     User
                      │
                      │ UserRole
                      ▼
                     Role
                      │
                      │ RolePermission
                      ▼
                  Permission
                      │
                      ▼
                 Protected API
```

```text
Admin APIs
    │
    ├── UserAdminController
    │       ├── UserAdminService
    │       └── UserRoleService
    │
    ├── RoleAdminController
    │       ├── RoleAdminService
    │       └── RolePermissionService
    │
    ├── PermissionAdminController
    │       └── PermissionAdminService
    │
    └── MenuAdminController
            └── MenuAdminService
```

## 19. Authentication → Authorization Relationship
Authentication과 IAM은 다음과 같이 연결된다.

```text
                 Authentication
                        │
                        ▼
               JwtAuthenticationFilter
                        │
                        ▼
                  User Identity
                        │
                        ▼
              UserAuthorityService
                        │
                        ▼
               Role / Permission
                        │
                        ▼
                 Authorization
                        │
                        ▼
                 Protected API
```

따라서 두 영역의 책임은 다음과 같이 분리된다.

* **Authentication**: 사용자 신원 확인
* **Authorization**: 확인된 사용자의 접근 권한 판단

## 20. Persistence Relationship
Application Component와 Persistence Component의 기본 관계는 다음과 같다.

```text
Controller
    │
    ▼
Service
    │
    ▼
Repository
    │
    ▼
Database
```

User 영역에서는 다음 구조를 가진다.

```text
UserController
    ↓
UserService
    ↓
UserRepository
    ↓
MySQL
```

Token 관련 상태는 별도의 Repository/Service를 통해 Redis와 연결된다.

```text
AuthService
    │
    ├── RefreshTokenRepository ──→ Redis
    │
    └── TokenBlacklistService ───→ Redis
```

## 21. Component Dependency Rules
현재 Architecture에서는 다음 책임 분리를 유지한다.

* **Controller**: 외부 요청을 수신하고 Application Service를 호출한다.
* **Service**: Application 및 Domain 관련 처리를 수행한다.
* **Repository**: Persistence 접근을 담당한다.
* **Security Component**: 인증 및 인가와 관련된 Security 처리를 담당한다.
* **Domain**: 시스템의 핵심 데이터를 표현한다.

## 22. Component Boundary
Component Architecture에서는 다음 사항을 정의하지 않는다.

* 구체적인 JWT Claim 값
* JWT 서명 알고리즘의 선택 근거
* Refresh Token Rotation 알고리즘
* Redis Key Naming
* Database Index 설계
* Transaction 세부 정책
* 장애 복구 절차

이러한 기술적 선택과 설계 결정은 ADR에서 관리한다.

## 23. Traceability

| Component | Related PRD |
| :--- | :--- |
| AuthController / AuthService | PRD-FUNC-001-authentication |
| JwtProvider / JwtAuthenticationFilter | PRD-SEC-001-jwt |
| RefreshTokenRepository | PRD-SEC-002-refresh-token |
| TokenBlacklistService | PRD-SEC-004-redis-failure |
| UserController / UserService / UserRepository | PRD-FUNC-002-user |
| UserAuthorityService | PRD-FUNC-003-iam, PRD-SEC-003-rbac |
| Role Components | PRD-FUNC-003-iam, PRD-SEC-003-rbac |
| Permission Components | PRD-FUNC-003-iam, PRD-SEC-003-rbac |
| UserRole Components | PRD-FUNC-003-iam, PRD-SEC-003-rbac |
| Menu Components | PRD-FUNC-003-iam |

## 24. Architecture → ADR
Component Architecture에서 기술적 선택이 필요한 부분은 ADR로 넘긴다.

```text
Component Architecture
        │
        ├── JWT
        │     ↓
        │   ADR
        │
        ├── Refresh Token
        │     ↓
        │   ADR
        │
        ├── RBAC
        │     ↓
        │   ADR
        │
        └── Redis Failure
              ↓
             ADR
```

Architecture는 현재 구조를 설명하고, ADR은 왜 해당 구조와 기술적 결정을 선택했는지를 설명한다.

## 25. Final Component Structure
현재 Backend Component 구조의 핵심은 다음과 같다.

```text
Backend
│
├── Authentication
│   │
│   ├── AuthController
│   ├── AuthService
│   ├── JwtProvider
│   ├── JwtAuthenticationFilter
│   ├── SecurityConfig
│   ├── RefreshTokenRepository
│   ├── TokenBlacklistService
│   └── UserAuthorityService
│
└── IAM
    │
    ├── User
    │   ├── UserController
    │   ├── UserService
    │   ├── UserRepository
    │   └── User
    │
    ├── Role
    │   ├── RoleAdminController
    │   ├── RoleAdminService
    │   └── RolePermissionService
    │
    ├── Permission
    │   ├── PermissionAdminController
    │   └── PermissionAdminService
    │
    ├── Admin
    │   ├── UserAdminController
    │   ├── UserAdminService
    │   └── UserRoleService
    │
    └── Menu
        ├── MenuAdminController
        └── MenuAdminService
```

이 구조를 기준으로 이후 Authentication Flow와 Authorization Flow에서 Component 간 실제 실행 순서를 정의한다.
