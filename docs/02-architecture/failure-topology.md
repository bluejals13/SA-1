# Failure Topology

## 1. 목적
본 문서는 SA-1 Backend의 주요 Component 및 Infrastructure 장애가 시스템의 인증, 인가, 데이터 처리에 미치는 영향을 정의한다.
대상 범위는 다음과 같다.

* Application
* MySQL
* Redis
* Authentication
* Authorization
* Token Invalidation
* External Request Path

본 문서는 장애 상황에서 시스템이 실제로 어떻게 동작하는지를 설명하고, 구현에서 확인되지 않은 장애 대응 정책을 임의로 확정하지 않는다.

## 2. Failure Topology Overview
전체적인 장애 구조는 다음과 같다.

```text
                         Client
                           │
                           ▼
                         Nginx
                           │
                           ▼
                       Backend
                      /       \
                     /         \
                    ▼           ▼
                 MySQL        Redis
                    │           │
                    │           ├── Refresh Token State
                    │           └── Token Invalidation State
                    │
                    └── User / IAM Data
```

주요 장애 지점은 다음과 같다.

```text
Client
  │
  ├── Network Failure
  │
  ▼
Nginx
  │
  ├── Proxy Failure
  │
  ▼
Backend
  │
  ├── Application Failure
  │
  ├── Authentication Failure
  │
  └── Authorization Failure
       │
       ├── MySQL Failure
       │
       └── Redis Failure
```

## 3. Failure Classification
장애를 다음과 같이 구분한다.

| Failure | Category | Primary Impact |
| :--- | :--- | :--- |
| Client Network Failure | Network | Request Delivery |
| Nginx Failure | Infrastructure | Request Routing |
| Backend Failure | Application | API Availability |
| MySQL Failure | Persistence | User / IAM Data |
| Redis Failure | State Store | Token Operations |
| JWT Validation Failure | Security | Authentication |
| Authorization Failure | Security | Resource Access |
| Token State Failure | Security | Refresh / Invalidation |

## 4. Client / Network Failure
Client와 Backend 사이의 네트워크 문제가 발생하면 요청 자체가 Backend에 도달하지 않을 수 있다.

```text
Client
  │
  X
  │
Backend
```

이 경우 Backend에서는 정상적인 API 요청으로 처리할 수 없다.
주요 영향은 다음과 같다.

* Request 전달 실패
* Response 수신 실패
* Timeout
* Connection Error

Application 내부의 Authentication 또는 Authorization 처리 이전 단계의 장애로 분류한다.

## 5. Nginx Failure
Nginx가 Backend 앞단에서 Proxy 역할을 수행하는 경우 Nginx 장애는 외부 요청 진입점에 영향을 준다.

```text
Client
  │
  ▼
 Nginx
  X
Backend
```

영향 범위:

```text
Nginx Failure
      │
      ▼
External Request
      │
      ▼
Backend 도달 불가
```

따라서 Backend Application 자체가 정상이어도 외부에서 API를 사용할 수 없는 상황이 발생할 수 있다.

## 6. Backend Application Failure
Backend Application 자체가 종료되거나 요청 처리가 불가능한 경우 전체 API 기능에 영향을 줄 수 있다.

```text
Client
  │
  ▼
Nginx
  │
  X
Backend
```

영향 범위는 Application의 단일 인스턴스 여부와 Deployment 구조에 따라 달라진다.
현재 Architecture에서는 구체적인 HA 정책을 임의로 확정하지 않는다.

## 7. MySQL Failure
MySQL은 User 및 IAM 관련 영속 데이터를 제공한다.

```text
Backend
   │
   X
MySQL
```

MySQL 장애가 발생하면 다음 기능에 영향을 줄 수 있다.

```text
MySQL Failure
     │
     ├── User 조회
     ├── User 변경
     ├── Role 조회
     ├── Permission 조회
     └── IAM 변경
```

따라서 Authentication 및 Authorization 역시 일부 영향을 받을 수 있다.

## 8. Authentication + MySQL Failure
Login 과정에서 User 정보를 확인하기 위해 MySQL 데이터가 필요한 경우 다음과 같은 흐름이 발생한다.

```text
Login
  │
  ▼
AuthService
  │
  ▼
UserRepository
  │
  X
MySQL
```

이 경우 정상적인 User Authentication을 완료할 수 없다.
따라서 MySQL 장애 상황에서 정상적인 인증 성공을 임의로 허용해서는 안 된다.

```text
MySQL Unavailable
       │
       ▼
User Verification Unavailable
       │
       ▼
Authentication Cannot Be Completed
```

## 9. IAM + MySQL Failure
IAM 데이터는 MySQL에 의존한다.

```text
User
Role
Permission
   │
   ▼
MySQL
```

MySQL 장애가 발생하면 IAM 관리 기능도 영향을 받는다.

```text
IAM Request
     │
     ▼
IAM Service
     │
     X
MySQL
```

영향 가능한 기능:
* User 관리
* User-Role 관리
* Role 관리
* Role-Permission 관리
* Permission 관리
* Menu 관리

## 10. Authorization + MySQL Failure
Authorization 과정에서 Runtime에 User / Role / Permission 데이터를 조회해야 하는 경우 MySQL 장애가 Authorization 판단에도 영향을 줄 수 있다.

```text
Authenticated User
        │
        ▼
UserAuthorityService
        │
        X
      MySQL
```

이 경우 필요한 Authority 정보를 정상적으로 구성할 수 없다.

```text
Authority Resolution Failure
          │
          ▼
Authorization Decision Unavailable
```

Fail-open / Fail-closed 여부는 보안상 중요한 정책이므로 별도의 ADR에서 확정한다.

## 11. Redis Failure
Redis는 Authentication 관련 상태 데이터의 저장소로 사용된다.

```text
Backend
   │
   X
Redis
```

현재 주요 영향 대상은 다음과 같다.

```text
Redis
│
├── Refresh Token State
│
└── Token Invalidation State
```

따라서 Redis 장애는 특히 Token Lifecycle에 영향을 준다.

## 12. Refresh Token + Redis Failure
Refresh Token 갱신 과정에서 Redis의 Token State를 확인할 수 없는 경우 정상적인 Refresh 처리를 완료할 수 없다.

```text
Refresh Request
      │
      ▼
AuthService
      │
      ▼
RefreshTokenRepository
      │
      X
    Redis
```

정상적인 Token State 검증 없이 새로운 인증 상태를 발급해서는 안 된다.

```text
Redis Unavailable
      │
      ▼
Refresh Token State Unavailable
      │
      ▼
Refresh Cannot Be Safely Validated
```

따라서 이 영역은 보안상 Fail-Closed가 요구되는지 ADR에서 명시적으로 결정해야 한다.

## 13. Token Invalidation + Redis Failure
Logout 또는 Token Invalidation 과정에서도 Redis가 사용될 수 있다.

```text
Logout
  │
  ▼
TokenBlacklistService
  │
  X
Redis
```

이 경우 Token Invalidation State를 정상적으로 기록할 수 없다.

```text
Invalidation Request
       │
       ▼
Redis Failure
       │
       ▼
Invalidation State Not Persisted
```

따라서 "Logout 성공"을 어떤 조건에서 반환할 것인지 명확한 정책이 필요하다.

## 14. JWT Validation Failure
JWT가 존재하더라도 다음과 같은 문제가 발생할 수 있다.

```text
Access Token
     │
     ▼
JwtAuthenticationFilter
     │
     ▼
JwtProvider
     │
     ├── Invalid Signature
     ├── Expired Token
     ├── Invalid Claims
     └── Invalid Token
```

이 경우 Authentication을 성공 상태로 만들지 않는다.

```text
JWT Validation Failure
        │
        ▼
Authentication Failure
        │
        ▼
Protected Resource Denied
```

## 15. Authorization Failure
Authentication은 성공했지만 필요한 Permission이 없는 경우 Authorization에서 거부한다.

```text
Authenticated User
       │
       ▼
Authorization Check
       │
       ▼
Permission Missing
       │
       ▼
Access Denied
```

이것은 Application Failure가 아니라 정상적인 Security Decision이다.

## 16. Token Blacklist State Failure
Token Invalidation 상태가 정상적으로 조회되지 않는 경우 Token의 현재 유효성을 안전하게 판단하기 어려울 수 있다.

```text
Access Token
     │
     ▼
JwtAuthenticationFilter
     │
     ▼
TokenBlacklistService
     │
     X
Redis
```

이 상황에서는 시스템이 다음 중 어떤 정책을 사용할지 결정해야 한다.

```text
Redis Failure
     │
     ├── Fail Open
     │
     └── Fail Closed
```

보안 정책에 대한 최종 결정은 ADR에서 수행한다.

## 17. Failure Propagation
하나의 Infrastructure 장애가 여러 기능으로 전파될 수 있다.

**MySQL**
```text
MySQL Failure
     │
     ├── User
     ├── Authentication
     ├── IAM
     └── Authorization
```

**Redis**
```text
Redis Failure
     │
     ├── Refresh Token
     ├── Token Rotation
     └── Token Invalidation
```

따라서 장애 분석 시 개별 API가 아니라 Dependency Graph를 기준으로 영향 범위를 판단한다.

## 18. Failure Boundary
현재 시스템의 주요 Dependency Boundary는 다음과 같다.

```text
┌─────────────────────────────────────┐
│              Backend                │
│                                     │
│  Auth      User       IAM           │
│   │         │         │             │
│   │         │         │             │
└───┼─────────┼─────────┼─────────────┘
    │         │         │
    ▼         ▼         ▼
  Redis     MySQL     MySQL
```

Authentication은 두 저장소에 걸쳐 동작할 수 있다.

```text
Authentication
      │
      ├── User Identity → MySQL
      │
      └── Token State → Redis
```

Authorization은 IAM Persistence에 의존한다.

```text
Authorization
      │
      ▼
User / Role / Permission
      │
      ▼
MySQL
```

## 19. Failure Handling Principle
장애 상황에서는 "정상 처리할 수 없는 상태를 정상 성공으로 변환하지 않는다"는 원칙을 적용한다.
특히 Security 관련 기능은 다음을 구분한다.

```text
Dependency Available
       │
       ▼
Normal Security Decision
```

반면:

```text
Dependency Unavailable
       │
       ▼
Security Decision Unavailable
       │
       ▼
Explicit Failure Policy Required
```

특히 다음 항목은 명시적인 정책 결정이 필요하다.

* Redis 장애 시 Refresh 허용 여부
* Redis 장애 시 Token Invalidation 처리
* MySQL 장애 시 Authorization 처리
* 권한 데이터 조회 실패 시 접근 허용 여부
* Token Blacklist 조회 실패 시 접근 허용 여부

## 20. Recovery Boundary
장애 복구 후 데이터의 정상 여부도 별도로 고려해야 한다.

```text
Failure
  │
  ▼
Dependency Recovery
  │
  ▼
State Validation
  │
  ▼
Normal Operation
```

특히 Redis의 경우 단순히 Process가 다시 살아나는 것과 Token State가 정상적으로 복구되는 것은 동일하지 않을 수 있다.
따라서 다음을 별도 검토 대상으로 둔다.

* Redis Persistence
* Redis Restart
* Token State Recovery
* TTL State
* Blacklist State Recovery

구체적인 복구 방식은 Infrastructure 및 Operations Architecture에서 정의한다.

## 21. Failure Matrix

| Failure Point | Authentication | Authorization | Refresh | Invalidation |
| :--- | :--- | :--- | :--- | :--- |
| Client Network | 영향 | 영향 | 영향 | 영향 |
| Nginx | 영향 | 영향 | 영향 | 영향 |
| Backend | 실패 | 실패 | 실패 | 실패 |
| MySQL | 영향 | 영향 | 영향 가능 | 영향 가능 |
| Redis | 영향 가능 | 영향 가능 | 직접 영향 | 직접 영향 |
| Invalid JWT | 실패 | 접근 불가 | 해당 없음 | 해당 없음 |
| Missing Permission | 성공 | 거부 | 해당 없음 | 해당 없음 |

영향 가능 항목은 실제 Runtime Dependency 및 구현 방식에 따라 달라질 수 있으므로 Architecture/ADR에서 확정한다.

## 22. Failure Traceability

| Failure Scenario | Related Requirement |
| :--- | :--- |
| Invalid JWT | PRD-SEC-001-jwt |
| Refresh Token State Unavailable | PRD-SEC-002-refresh-token |
| RBAC Authority Resolution Failure | PRD-SEC-003-rbac |
| Token Invalidation Failure | 관련 Token Invalidation PRD |
| User Data Unavailable | PRD-FUNC-002-user |
| IAM Data Unavailable | PRD-FUNC-003-iam |

## 23. ADR Candidates
다음 항목은 단순한 구현 문제가 아니라 Architecture Decision이 필요한 영역이다.

**ADR Candidate 1 — Redis Failure Policy**
```text
Redis Failure
     │
     ├── Refresh
     ├── Blacklist
     └── Token Validation
```
장애 시 Fail-Open 또는 Fail-Closed 정책 결정.

**ADR Candidate 2 — Authorization Dependency Failure**
```text
User / Role / Permission
          │
          X
        MySQL
```
Authority 조회 실패 시 접근 정책 결정.

**ADR Candidate 3 — Token Invalidation Reliability**
```text
Logout
  │
  ▼
Invalidation
  │
  X
Redis
```
Invalidation State 기록 실패 시 Logout 결과 정책 결정.

## 24. Final Failure Topology
전체 Failure Topology는 다음과 같다.

```text
                              Client
                                │
                         Network Failure
                                │
                                ▼
                              Nginx
                                │
                         Proxy Failure
                                │
                                ▼
                             Backend
                           /         \
                          /           \
                         ▼             ▼
                      MySQL          Redis
                        │               │
             ┌──────────┼───────┐      ├── Refresh Token
             │          │       │      └── Invalidation
             ▼          ▼       ▼
           User        Role  Permission
             │          │       │
             └──────┬───┴───────┘
                    ▼
             Authorization
                    │
              ┌─────┴─────┐
              ▼           ▼
            Allow        Deny


Authentication Path

Client
  │
  ▼
AuthController
  │
  ▼
AuthService
  │
  ├──────────────→ MySQL
  │
  └──────────────→ Redis
                       │
                       ▼
                 Token State


Failure Principle

Dependency Failure
       │
       ▼
Security Decision Unavailable
       │
       ▼
Explicit Failure Policy
       │
       ▼
ADR
       │
       ▼
Test
       │
       ▼
Execution / Evidence
```

## 25. Architecture Traceability

```text
PRD
 ↓
Architecture
 ↓
Component
 ↓
Data Flow
 ↓
Failure Topology
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

Failure Topology의 목적은 장애를 단순히 "발생할 수 있다"고 기록하는 것이 아니라, 어떤 Dependency 장애가 어떤 Security Boundary를 통과하여 어떤 기능에 영향을 주는지 추적 가능하게 만드는 것이다.
