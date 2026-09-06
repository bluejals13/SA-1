# Authorization Flow

## 1. 목적
본 문서는 인증된 사용자의 권한 정보를 기반으로 보호된 시스템 기능에 대한 접근을 판단하는 Authorization 처리 흐름을 정의한다.
`authentication-flow.txt`에서 인증된 사용자의 Identity가 구성된 이후, User → Role → Permission 관계를 통해 권한이 결정되는 흐름을 설명한다.
본 문서는 현재 26-05adf의 IAM 및 Security 구조를 기준으로 작성한다.

## 2. Authorization Overview
전체 Authorization 흐름은 다음과 같다.

```text
Client
  ↓
Access Token
  ↓
JwtAuthenticationFilter
  ↓
JwtProvider
  ↓
Authenticated User
  ↓
UserAuthorityService
  ↓
User
  ↓
Role
  ↓
Permission
  ↓
Authorization Decision
  ↓
Protected Resource
```

핵심 개념은 다음과 같다.

* **Authentication**: 요청자가 누구인지 확인
* **Authorization**: 해당 요청자가 무엇을 수행할 수 있는지 판단

## 3. Authorization Boundary
Authorization은 Authentication이 완료된 이후 수행된다.

```text
External Request
       │
       ▼
Authentication
       │
       ├── Failure → Reject
       │
       ▼
Authenticated User
       │
       ▼
Authorization
       │
       ├── Denied → Reject
       │
       ▼
Protected Resource
```

따라서 인증에 성공했다고 해서 모든 API에 접근할 수 있는 것은 아니다.

## 4. User Authority Construction
인증된 사용자에 대한 Authority 정보는 UserAuthorityService를 통해 구성된다.

```text
Authenticated User
        │
        ▼
UserAuthorityService
        │
        ▼
       User
```

User와 연결된 Role 및 Permission을 기반으로 Security Layer에서 사용할 권한 정보를 구성한다.

```text
User
 │
 └── Role
      │
      └── Permission
```

## 5. User → Role → Permission
IAM의 기본 권한 관계는 다음과 같다.

```text
User
 │
 │ User-Role
 ▼
Role
 │
 │ Role-Permission
 ▼
Permission
```

이를 Authorization 관점에서 표현하면 다음과 같다.

```text
User
 ↓
"어떤 Role을 가지고 있는가?"
 ↓
Role
 ↓
"해당 Role이 어떤 Permission을 가지고 있는가?"
 ↓
Permission
 ↓
"요청한 Resource에 접근 가능한가?"
```

## 6. User-Role Relationship
사용자에게 Role이 연결되면 해당 사용자는 해당 Role에서 정의된 권한을 사용할 수 있는 기반을 갖는다.

```text
User
  │
  ▼
UserRoleService
  │
  ▼
Role
```

관리자 기능에서 User와 Role의 관계를 변경할 수 있다.
주요 Component 관계는 다음과 같다.

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
User ↔ Role
```

User-Role 관계의 변경은 이후 Authorization 판단에 영향을 준다.

## 7. Role-Permission Relationship
Role은 하나 이상의 Permission과 연결될 수 있다.

```text
Role
 │
 ▼
RolePermissionService
 │
 ▼
Permission
```

관리자 기능에서는 Role과 Permission의 관계를 관리한다.

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
Role ↔ Permission
```

## 8. Permission
Permission은 보호된 시스템 기능에 대한 접근 권한을 표현한다.

```text
Permission
     │
     ▼
Protected Resource
```

Permission 자체는 User에게 직접 연결되는 것이 아니라 Role을 통해 User에게 전달되는 구조를 사용한다.

```text
User
  │
  ▼
Role
  │
  ▼
Permission
```

이 구조가 현재 시스템의 RBAC 모델이다.

## 9. RBAC Flow
전체 RBAC 흐름은 다음과 같다.

```text
                 User
                  │
                  │
                  ▼
                Role
                  │
                  │
                  ▼
             Permission
                  │
                  ▼
          Authorization Check
                  │
             ┌────┴────┐
             │         │
           Allow      Deny
             │         │
             ▼         ▼
       Protected API   Reject
```

RBAC의 정책 요구사항은 PRD-SEC-003-rbac에서 정의한다.

## 10. Authenticated Request Flow
실제 요청 처리의 전체 흐름은 다음과 같다.

```text
Client
  │
  │ Authorization: Bearer <JWT>
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
Authorities
  │
  ▼
Authorization
  │
  ├── Allow
  │     │
  │     ▼
  │  Controller
  │     │
  │     ▼
  │  Service
  │
  └── Deny
        │
        ▼
      Reject
```

## 11. JWT → Authority Flow
JWT가 인증된 사용자 Identity를 제공하고, IAM 데이터가 Authorization에 필요한 Authority를 제공한다.

```text
JWT
 │
 ▼
JwtProvider
 │
 ▼
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
Authorities
```

따라서 JWT와 RBAC의 책임은 구분한다.

* **JWT**: Authentication Identity
* **Role / Permission**: Authorization Authority

## 12. Protected API Decision
보호된 API에 요청이 들어오면 다음 조건을 순서대로 확인한다.

```text
Request
  │
  ▼
JWT Authentication
  │
  ├── Invalid → Reject
  │
  ▼
Authenticated User
  │
  ▼
Authority
  │
  ├── Required Permission 없음
  │          │
  │          ▼
  │        Reject
  │
  ▼
Required Permission 있음
  │
  ▼
Allow
```

즉, 다음 두 조건을 모두 만족해야 한다.
1. Valid Authentication
2. Required Authority

## 13. Authorization Failure
인증에는 성공했지만 필요한 권한이 없는 경우 접근을 허용해서는 안 된다.

```text
Authenticated
      │
      ▼
Authorization Check
      │
      ├── Permission Exists
      │        ↓
      │      Allow
      │
      └── Permission Missing
               ↓
             Deny
```

이 경우 Authentication Failure와 Authorization Failure를 구분한다.

* **Authentication Failure**: 신원을 확인할 수 없음
* **Authorization Failure**: 신원은 확인했지만 권한이 없음

## 14. User Administration Flow
관리자가 사용자의 IAM 정보를 관리하는 흐름은 다음과 같다.

```text
Admin
  │
  ▼
UserAdminController
  │
  ▼
UserAdminService
  │
  ├── User Management
  │
  └── UserRoleService
          │
          ▼
       User ↔ Role
```

사용자의 Role 변경은 이후 Authorization 판단에 영향을 준다.

## 15. Role Administration Flow
관리자가 Role을 관리하는 흐름이다.

```text
Admin
  │
  ▼
RoleAdminController
  │
  ▼
RoleAdminService
  │
  ├── Role Management
  │
  └── RolePermissionService
          │
          ▼
      Role ↔ Permission
```

## 16. Permission Administration Flow
관리자가 Permission을 관리하는 흐름이다.

```text
Admin
  │
  ▼
PermissionAdminController
  │
  ▼
PermissionAdminService
  │
  ▼
Permission
```

Permission의 생성 및 관리 결과는 Role-Permission 관계를 통해 Authorization에 반영된다.

## 17. Menu Administration Flow
Menu는 IAM 관리 영역에서 별도의 관리 Component를 가진다.

```text
Admin
  │
  ▼
MenuAdminController
  │
  ▼
MenuAdminService
  │
  ▼
Menu
```

Menu와 Permission의 구체적인 연결 규칙은 현재 Component Architecture에서 확정하지 않는다.
해당 관계가 별도의 기술적 결정으로 필요한 경우 ADR에서 정의한다.

## 18. Authorization Data Flow
Authorization에 필요한 주요 데이터 관계는 다음과 같다.

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
```

이를 요청 처리와 연결하면 다음과 같다.

```text
Access Token
     │
     ▼
Authenticated User
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
Authorization Decision
```

## 19. Authorization State Change
IAM 관리자가 User-Role 또는 Role-Permission 관계를 변경하면 이후 Authorization 결과에 영향을 줄 수 있다.

```text
IAM Administration
       │
       ├── User ↔ Role 변경
       │
       └── Role ↔ Permission 변경
                 │
                 ▼
          Authorization State
                 │
                 ▼
       Future Authorization Request
```

단, 이미 발급된 JWT에 Role/Permission 정보가 포함되어 있는 경우 해당 정보의 변경이 기존 Token에 언제 반영되는지는 JWT 및 Authorization 설계에서 별도로 결정해야 한다.
이 부분은 Architecture에서 임의로 결정하지 않으며 관련 ADR의 설계 결정 대상으로 둔다.

## 20. Security Separation
Authentication과 Authorization의 책임을 다음과 같이 분리한다.

```text
┌──────────────────────────────────────┐
│           Authentication             │
│                                      │
│ JWT / Identity / Authentication      │
└──────────────────┬───────────────────┘
                   │
                   ▼
┌──────────────────────────────────────┐
│           Authorization              │
│                                      │
│ User / Role / Permission / RBAC      │
└──────────────────┬───────────────────┘
                   │
                   ▼
            Protected Resource
```

## 21. Component Mapping

| Authorization Responsibility | Component |
| :--- | :--- |
| JWT Authentication | JwtAuthenticationFilter |
| JWT Validation | JwtProvider |
| Authority Construction | UserAuthorityService |
| User Management | UserAdminService |
| User-Role Management | UserRoleService |
| Role Management | RoleAdminService |
| Role-Permission Management | RolePermissionService |
| Permission Management | PermissionAdminService |
| Menu Management | MenuAdminService |

## 22. PRD Traceability

| Authorization Flow | Related PRD |
| :--- | :--- |
| Authenticated Request | PRD-FUNC-001-authentication |
| User | PRD-FUNC-002-user |
| IAM | PRD-FUNC-003-iam |
| JWT Identity | PRD-SEC-001-jwt |
| RBAC | PRD-SEC-003-rbac |
| Token Invalidation | 관련 Token Invalidation PRD |

## 23. Architecture Traceability
```text
PRD-FUNC-001
       │
       ├── Authentication
       │
       ▼
PRD-SEC-001
       │
       └── JWT
              │
              ▼
      JwtAuthenticationFilter
              │
              ▼
       UserAuthorityService
              │
              ▼
        User / Role / Permission
              │
              ▼
         Authorization
              │
              ▼
         Protected API
```

IAM 관리 영역은 별도의 관리 흐름으로 연결된다.
```text
PRD-FUNC-003
       │
       ▼
IAM Administration
       │
       ├── User / Role
       ├── Role / Permission
       ├── Permission
       └── Menu
       │
       ▼
Authorization State
```

## 24. Boundary
본 문서는 Authorization의 논리적인 처리 흐름을 정의한다.
다음 세부사항은 본 문서에서 확정하지 않는다.

* Permission Naming Convention
* Spring Security 권한 표현 방식
* JWT에 Role/Permission을 포함할지 여부
* Runtime 권한 조회 여부
* 권한 변경 즉시 반영 정책
* Cache 정책
* 권한 변경과 기존 Token의 관계

이러한 기술적 선택은 ADR에서 정의한다.

## 25. Final Authorization Flow
최종적으로 현재 시스템의 Authorization 흐름은 다음과 같이 정리한다.

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
                Authenticated User
                       │
                       ▼
              UserAuthorityService
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
           User                 Role
                                 │
                                 ▼
                            Permission
                                 │
                                 ▼
                       Authorization Check
                                 │
                    ┌────────────┴────────────┐
                    │                         │
                  Allow                      Deny
                    │                         │
                    ▼                         ▼
             Protected API                 Reject
```

관리자에 의한 IAM 변경은 다음 경로를 통해 Authorization 상태에 영향을 준다.

```text
Admin
 │
 ├── UserAdminController
 │       ↓
 │   UserAdminService
 │       ↓
 │   UserRoleService
 │
 ├── RoleAdminController
 │       ↓
 │   RoleAdminService
 │       ↓
 │   RolePermissionService
 │
 ├── PermissionAdminController
 │       ↓
 │   PermissionAdminService
 │
 └── MenuAdminController
         ↓
     MenuAdminService

              ↓
        IAM State
              ↓
      Authorization
```
