# Data Flow Architecture

## 1. 목적
본 문서는 SA-1 Backend에서 주요 데이터가 생성, 조회, 변경 및 저장되는 흐름을 정의한다.
주요 대상은 다음과 같다.

* User
* Role
* Permission
* User-Role 관계
* Role-Permission 관계
* Refresh Token State
* Token Invalidation State

본 문서는 component.txt, authentication-flow.txt, authorization-flow.txt에서 정의한 Component 관계를 데이터 흐름 관점으로 구체화한다.
현재 apms-sr 의 Backend 구조를 기준으로 작성하며, 실제 구현에서 확인되지 않은 데이터 구조를 임의로 확정하지 않는다.

## 2. Data Storage Overview
현재 주요 데이터 저장소는 MySQL과 Redis로 구분한다.

```text
                    Backend
                       │
              ┌────────┴────────┐
              │                 │
              ▼                 ▼
           MySQL              Redis
              │                 │
              │                 │
       Persistent Data       Token State
```

* **MySQL**: 장기적으로 유지되어야 하는 시스템 데이터를 저장한다.
* **Redis**: 인증 과정에서 필요한 Token 관련 상태를 저장한다.

## 3. Data Domain Overview
주요 데이터 Domain은 다음과 같다.

```text
MySQL
│
├── User
│
├── Role
│
├── Permission
│
├── User ↔ Role
│
└── Role ↔ Permission

Redis
│
├── Refresh Token State
│
└── Token Invalidation State
```

## 4. User Data Flow
User 데이터의 기본 흐름은 다음과 같다.

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
     MySQL
```

User 생성 또는 변경 시 Service Layer를 통해 Persistence Layer로 전달된다.

```text
Client
  │
  ▼
UserController
  │
  ▼
UserService
  │
  ▼
UserRepository
  │
  ▼
MySQL
```

User 조회도 동일하게 Repository를 통해 저장소에 접근한다.

```text
MySQL
  │
  ▼
UserRepository
  │
  ▼
UserService
  │
  ▼
UserController
  │
  ▼
Client
```

## 5. User Domain
User는 Authentication과 Authorization의 기준이 되는 핵심 데이터다.

```text
User
 │
 ├── Authentication Identity
 │
 └── Authorization Relationship
          │
          ▼
        Role
```

User의 구체적인 Attribute 및 Database Schema는 Data Model 및 실제 Entity 구현을 기준으로 별도 관리한다.

## 6. Role Data Flow
Role 관리의 기본 흐름은 다음과 같다.

```text
RoleAdminController
        │
        ▼
RoleAdminService
        │
        ▼
Role Persistence
        │
        ▼
      MySQL
```

Role과 Permission의 관계 변경은 RolePermissionService를 통해 처리한다.

```text
RoleAdminController
        │
        ▼
RoleAdminService
        │
        ▼
RolePermissionService
        │
        ▼
     MySQL
```

## 7. Permission Data Flow
Permission 관리의 기본 흐름은 다음과 같다.

```text
PermissionAdminController
          │
          ▼
PermissionAdminService
          │
          ▼
      Permission
          │
          ▼
        MySQL
```

Permission은 Role과 연결되어 Authorization에 사용된다.

## 8. User-Role Data Flow
User와 Role의 관계는 IAM에서 중요한 Authorization 데이터다.

```text
User
 │
 │ User-Role Relationship
 ▼
Role
```

관리자에 의한 관계 변경은 다음과 같다.

```text
UserAdminController
       │
       ▼
UserAdminService
       │
       ▼
UserRoleService
       │
       ▼
    MySQL
```

## 9. Role-Permission Data Flow
Role과 Permission의 관계는 다음과 같다.

```text
Role
 │
 │ Role-Permission Relationship
 ▼
Permission
```

관리자에 의한 변경 흐름은 다음과 같다.

```text
RoleAdminController
        │
        ▼
RoleAdminService
        │
        ▼
RolePermissionService
        │
        ▼
      MySQL
```

## 10. Authorization Data Flow
User, Role, Permission 데이터는 Authorization 판단에 사용된다.

```text
             MySQL
                │
       ┌────────┼────────┐
       ▼        ▼        ▼
      User     Role   Permission
       │        │        │
       └────┬───┘        │
            │            │
            └──────┬─────┘
                   ▼
          UserAuthorityService
                   │
                   ▼
             Authorization
```

개념적인 관계는 다음과 같다.

```text
User
 │
 ▼
User-Role
 │
 ▼
Role
 │
 ▼
Role-Permission
 │
 ▼
Permission
```

## 11. Authentication Data Flow
Authentication에서는 User 데이터와 Token 데이터가 서로 다른 저장소에서 관리된다.

```text
                    AuthService
                    /         \
                   /           \
                  ▼             ▼
               MySQL          Redis
                  │             │
                  ▼             ▼
                User       Token State
```

User의 인증 정보는 MySQL 기반 User Domain을 사용하고, Refresh Token 등의 상태는 Redis를 사용한다.

## 12. Login Data Flow
Login 요청의 데이터 흐름은 다음과 같다.

```text
Client
  │
  │ Credentials
  ▼
AuthController
  │
  ▼
AuthService
  │
  ├──────────────→ UserRepository
  │                      │
  │                      ▼
  │                    MySQL
  │
  ▼
JwtProvider
  │
  ├── Access Token
  │
  └── Refresh Token
          │
          ▼
       Redis State
```

핵심적으로 Login 과정에서는 다음 데이터 흐름이 존재한다.

```text
Credentials
    ↓
User Identity
    ↓
Authentication Result
    ↓
Access Token
    ↓
Refresh Token State
```

## 13. Access Token Data Flow
Access Token은 기본적으로 Client와 Backend 사이에서 인증 상태를 전달한다.

```text
Client
  │
  │ Access Token
  ▼
JwtAuthenticationFilter
  │
  ▼
JwtProvider
  │
  ▼
Authenticated Identity
```

JWT 자체의 저장 및 Claim 정책은 PRD-SEC-001-jwt와 관련 ADR에서 정의한다.

## 14. Refresh Token Data Flow
Refresh Token은 Authentication State 갱신에 사용된다.

```text
Client
  │
  │ Refresh Token
  ▼
AuthController
  │
  ▼
AuthService
  │
  ▼
RefreshTokenRepository
  │
  ▼
Redis
```

Redis에 저장된 Token State와 요청된 Refresh Token을 비교하여 갱신 가능 여부를 판단한다.

## 15. Refresh Token Rotation Data Flow
Rotation이 수행되는 경우 기존 Token State를 새로운 Token State로 변경한다.

```text
Old Refresh Token
        │
        ▼
    Validation
        │
        ▼
Rotation Processing
        │
   ┌────┴────┐
   ▼         ▼
Invalidate  New Token
   │         │
   └────┬────┘
        ▼
      Redis
```

개별 Key 구조와 Atomicity 보장 방법은 ADR에서 정의한다.

## 16. Refresh Token Replay Data Flow
이미 사용되었거나 무효화된 Refresh Token이 다시 제출되는 경우 정상적인 갱신 데이터 흐름으로 처리하지 않는다.

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

Replay 방어 정책은 PRD-SEC-002-refresh-token과 연결된다.

## 17. Token Invalidation Data Flow
Token을 무효화해야 하는 경우 해당 상태를 Redis에서 관리한다.

```text
Logout / Invalidation
        │
        ▼
TokenBlacklistService
        │
        ▼
      Redis
        │
        ▼
Invalidated Token State
```

이후 해당 Token이 요청에 사용되었을 때 Authentication 과정에서 유효한 상태로 처리되지 않도록 한다.

## 18. Authorization Read Flow
Authorization 판단 시 필요한 데이터의 논리적인 흐름은 다음과 같다.

```text
Authentication
      │
      ▼
User Identity
      │
      ▼
User
      │
      ▼
Role
      │
      ▼
Permission
      │
      ▼
Authority
      │
      ▼
Authorization Decision
```

Persistence 관점에서는 MySQL의 IAM 데이터가 기준이 된다.

```text
MySQL
 │
 ├── User
 ├── Role
 ├── Permission
 ├── User-Role
 └── Role-Permission
          │
          ▼
 UserAuthorityService
          │
          ▼
 Authorization
```

## 19. IAM Administration Data Flow
IAM 관리 기능은 크게 네 가지 데이터 흐름으로 구분된다.

**User**
```text
UserAdminController
      ↓
UserAdminService
      ↓
User Data
      ↓
MySQL
```

**User-Role**
```text
UserAdminController
      ↓
UserRoleService
      ↓
User ↔ Role
      ↓
MySQL
```

**Role-Permission**
```text
RoleAdminController
      ↓
RoleAdminService
      ↓
RolePermissionService
      ↓
Role ↔ Permission
      ↓
MySQL
```

**Permission**
```text
PermissionAdminController
      ↓
PermissionAdminService
      ↓
Permission
      ↓
MySQL
```

## 20. Menu Data Flow
Menu는 별도의 IAM 관리 데이터로 처리된다.

```text
MenuAdminController
        │
        ▼
MenuAdminService
        │
        ▼
       Menu
        │
        ▼
      MySQL
```

Menu와 Permission 사이의 구체적인 데이터 관계는 현재 Data Flow에서 임의로 확정하지 않는다.

## 21. Write Flow
주요 영속 데이터 변경은 다음 패턴을 따른다.

```text
Client
  │
  ▼
Controller
  │
  ▼
Service
  │
  ▼
Repository
  │
  ▼
MySQL
```

대표적인 예는 다음과 같다.

**User Management**
```text
Controller
   ↓
Service
   ↓
Repository
   ↓
MySQL
```

**Role Management**
```text
Controller
   ↓
Service
   ↓
Repository / Relation Processing
   ↓
MySQL
```

## 22. Read Flow
조회 데이터는 역방향으로 Client까지 전달된다.

```text
MySQL
  │
  ▼
Repository
  │
  ▼
Service
  │
  ▼
Controller
  │
  ▼
Client
```

## 23. Redis State Flow
Redis에 저장되는 인증 관련 상태는 다음과 같이 구분한다.

```text
Redis
│
├── Refresh Token State
│
└── Token Invalidation State
```

두 상태 모두 Authentication Security에 사용되지만 책임은 구분한다.

```text
RefreshTokenRepository
        │
        ▼
Refresh Token State

TokenBlacklistService
        │
        ▼
Token Invalidation State
```

## 24. Data Ownership
각 데이터의 주요 책임 Component는 다음과 같다.

| Data | Primary Component | Storage |
| :--- | :--- | :--- |
| User | UserService / UserRepository | MySQL |
| Role | RoleAdminService | MySQL |
| Permission | PermissionAdminService | MySQL |
| User-Role | UserRoleService | MySQL |
| Role-Permission | RolePermissionService | MySQL |
| Refresh Token State | RefreshTokenRepository | Redis |
| Token Invalidation State | TokenBlacklistService | Redis |
| Menu | MenuAdminService | MySQL |

## 25. Consistency Boundary
MySQL 데이터와 Redis Token State는 서로 다른 저장소이므로 동일한 데이터 일관성 모델로 취급하지 않는다.

```text
              Backend
             /       \
            ▼         ▼
         MySQL      Redis
           │           │
     Persistent      Token
       State         State
```

특히 다음 데이터는 서로 다른 lifecycle을 가진다.

```text
User / Role / Permission
        │
        ▼
  Long-lived State

Refresh Token / Blacklist
        │
        ▼
  Time-bounded State
```

따라서 Redis Token State와 MySQL IAM State의 동기화가 필요한 경우 별도의 Architecture 또는 ADR에서 정의한다.

## 26. Data Flow Traceability

| Data Flow | Related PRD |
| :--- | :--- |
| User Data | PRD-FUNC-002-user |
| User Authentication | PRD-FUNC-001-authentication |
| JWT | PRD-SEC-001-jwt |
| Refresh Token State | PRD-SEC-002-refresh-token |
| User / Role / Permission | PRD-FUNC-003-iam |
| RBAC Authority | PRD-SEC-003-rbac |
| Token Invalidation | 관련 Token Invalidation PRD |
| Redis Failure | PRD-SEC-004-redis-failure |

## 27. Data Flow → Implementation
최종적인 데이터 추적 경로는 다음과 같다.

```text
PRD
 ↓
Architecture
 ↓
Component
 ↓
Data Flow
 ↓
Service
 ↓
Repository
 ↓
Storage
 ↓
Test
 ↓
Execution
 ↓
Evidence
```

**Authentication의 경우:**
```text
PRD-SEC-001
      ↓
JwtProvider
      ↓
JwtAuthenticationFilter
      ↓
Authentication
      ↓
Protected Request
```

**Refresh Token의 경우:**
```text
PRD-SEC-002
      ↓
AuthService
      ↓
RefreshTokenRepository
      ↓
Redis
      ↓
Refresh / Rotation / Replay Defense
```

**IAM의 경우:**
```text
PRD-FUNC-003
      ↓
IAM Service
      ↓
Repository
      ↓
MySQL
      ↓
User / Role / Permission
      ↓
Authorization
```

## 28. Boundary
본 문서는 데이터의 논리적인 이동과 저장소 경계를 정의한다.
다음 세부사항은 별도 문서에서 정의한다.

* Entity 상세 Schema
* Database Table 상세 구조
* Index
* Redis Key Naming
* Redis TTL
* Transaction Isolation
* Atomic Operation
* Cache Strategy
* Cross-Store Transaction
* Backup / Restore

## 29. Final Data Flow
전체 시스템의 주요 데이터 흐름은 다음과 같다.

```text
                         Client
                           │
              ┌────────────┴────────────┐
              │                         │
        Credentials                 Access Token
              │                         │
              ▼                         ▼
        AuthController        JwtAuthenticationFilter
              │                         │
              ▼                         ▼
         AuthService                JwtProvider
              │                         │
        ┌─────┴─────┐                   ▼
        │           │              Authenticated User
        ▼           ▼                   │
     MySQL        Redis                 ▼
        │           │            UserAuthorityService
        ▼           ▼                   │
       User    Token State              ▼
                                    Role / Permission
                                        │
                                        ▼
                                  Authorization
                                        │
                                        ▼
                                  Protected API


IAM Administration
        │
        ├── User
        ├── User-Role
        ├── Role
        ├── Role-Permission
        ├── Permission
        └── Menu
                │
                ▼
              MySQL
```

이 구조를 기준으로 이후 Failure Topology에서는 각 Storage 및 Container 장애가 이 데이터 흐름에 어떤 영향을 주는지 정의한다.
