
# Architecture Overview

## 1. 목적

본 문서는 시스템의 전체 아키텍처를 정의하고, 주요 애플리케이션 구성요소와 외부 인프라 간의 관계를 설명한다.

본 문서는 01-prd에서 정의한 요구사항을 실제 시스템 구조와 연결하기 위한 상위 수준의 Architecture 문서다.

구체적인 기술 선택 및 설계 결정은 Architecture 상세 문서와 ADR에서 정의한다.

## 2. 시스템 전체 구조

현재 시스템은 다음과 같은 구조로 구성된다.

```text
                         Client
                           │
                           ▼
                    ┌─────────────┐
                    │    Nginx    │
                    │ Reverse     │
                    │ Proxy       │
                    └──────┬──────┘
                           │
                    /api/* │
                           ▼
                ┌─────────────────────┐
                │   Spring Boot       │
                │      Backend        │
                └──────────┬──────────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
     ┌─────────┐      ┌─────────┐     ┌────────────┐
     │  Auth   │      │   IAM   │     │ Application│
     │         │      │         │     │  Services  │
     └────┬────┘      └────┬────┘     └─────┬──────┘
          │                │                 │
          ├───────┐        │                 │
          ▼       │        ▼                 ▼
       ┌─────┐    │    ┌────────┐        ┌────────┐
       │ JWT │    │    │ User   │        │ MySQL  │
       └─────┘    │    │ Role   │        └────────┘
                  │    │ Perm.  │
                  │    └────────┘
                  │
                  ▼
              ┌────────┐
              │ Redis  │
              │        │
              │Refresh │
              │Token   │
              │Blacklist
              └────────┘
````

 ## 3. 주요 시스템 구성요소

 ### 3.1 Client

 사용자는 Web Client를 통해 시스템에 접근한다.

 Client의 요청은 시스템의 단일 진입점인 Nginx를 통해 Backend API로 전달된다.

 ### 3.2 Nginx

 Nginx는 외부 요청의 단일 진입점 역할을 수행한다.

 주요 역할은 다음과 같다.

 - Frontend 정적 리소스 제공
- `/api/*` 요청을 Backend로 전달
- 외부 요청과 Backend 간 Reverse Proxy 역할
- 필요한 HTTP Header 및 CORS 처리

 현재 Docker 환경에서는 Nginx가 외부 HTTP 요청을 수신하고 Backend의 8080 포트로 API 요청을 전달한다.

 ### 3.3 Spring Boot Backend

 Backend는 시스템의 핵심 비즈니스 로직과 인증/인가 기능을 담당한다.

 Backend 내부는 기능별 Package 구조를 기준으로 분리되어 있다.

```txt
backend
└── src/main/java/com/example/demo
    ├── auth
    └── iam
```

 Auth 영역은 인증과 Token 처리를 담당하고, IAM 영역은 사용자·Role·Permission 기반의 접근 제어 관리 기능을 담당한다.

 ## 4. Authentication Architecture

 인증 영역은 다음 구성요소로 구성된다.

```txt
Client
  │
  ▼
AuthController
  │
  ▼
AuthService
  │
  ├── JwtProvider
  │
  ├── RefreshTokenRepository
  │
  └── TokenBlacklistService
```

 Access Token 기반 인증 요청은 다음과 같이 처리된다.

```txt
Client
  │
  │ Access Token
  ▼
JwtAuthenticationFilter
  │
  ▼
JwtProvider
  │
  ├── Token 검증
  └── 인증 정보 생성
  │
  ▼
Security Context
  │
  ▼
Protected API
```

 관련 구현은 `auth/security`, `auth/jwt` 영역에 위치한다.

 ## 5. Refresh Token Architecture

 Refresh Token은 Access Token 갱신을 위한 인증 상태 관리에 사용된다.

 전체적인 구조는 다음과 같다.

```txt
Client
  │
  │ Refresh Token
  ▼
AuthController
  │
  ▼
AuthService
  │
  ├── RefreshTokenRepository
  │
  ├── Token validation
  │
  └── Token rotation
  │
  ▼
Redis
```

 Refresh Token의 구체적인 Rotation, Replay 방어 및 상태 관리 정책은 다음 문서에서 정의한다.

 - `PRD-SEC-002-refresh-token`
- 관련 Architecture 상세 문서
- 관련 ADR

 Architecture 문서에서는 해당 요구사항을 만족하기 위한 실제 구성요소와 데이터 흐름을 설명한다.

 ## 6. Authorization / IAM Architecture

 인가 영역은 User, Role, Permission의 관계를 기반으로 구성된다.

```txt
User
  │
  ▼
Role
  │
  ▼
Permission
  │
  ▼
Protected Resource
```

 IAM 영역의 주요 구성요소는 다음과 같다.

```txt
iam
├── user
├── role
├── permission
├── admin
└── menu
```

 주요 역할은 다음과 같다.

 - `User`: 시스템 사용자 및 사용자 상태 관리
- `Role`: 사용자에게 부여되는 역할 관리
- `Permission`: 기능 접근 권한 관리
- `UserRoleService`: User와 Role 관계 관리
- `RolePermissionService`: Role과 Permission 관계 관리
- `UserAuthorityService`: 인증된 사용자의 권한 구성
- `Admin Service / Controller`: IAM 관리 기능 제공

 ## 7. Authorization Request Flow

 인증된 사용자의 보호 API 요청은 다음과 같은 흐름으로 처리된다.

```txt
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
JWT Validation
  │
  ▼
User Authentication
  │
  ▼
User Authority
  │
  ▼
Role / Permission
  │
  ▼
Authorization
  │
  ├── Allowed ──→ Protected API
  │
  └── Denied ──→ Authorization Error
```

 따라서 인증(Authentication)과 인가(Authorization)는 서로 다른 책임으로 구성된다.

```txt
Authentication
    ↓
"누구인가?"

Authorization
    ↓
"무엇을 할 수 있는가?"
```

 ## 8. Persistence Architecture

 시스템의 영속성 계층은 MySQL을 사용한다.

```txt
Spring Boot
     │
     ▼
Repository
     │
     ▼
MySQL
```

 MySQL은 사용자 및 IAM 관련 데이터를 포함한 애플리케이션의 영속 데이터를 관리한다.
 Database Schema 변경은 현재 프로젝트의 Migration 구조를 통해 관리한다.

 ## 9. Redis Architecture

 Redis는 인증 상태와 Token 관련 상태 관리에 사용된다.

 현재 Architecture 기준으로 Redis의 주요 용도는 다음과 같다.

 - Refresh Token 상태 관리
- JWT Blacklist 상태 관리
- TTL 기반 Token 상태 관리

```txt
Spring Boot
     │
     ├───────────────┐
     ▼               ▼
Refresh Token     Token Blacklist
     │               │
     └───────┬───────┘
             ▼
           Redis
```

 Redis 장애 상황에서 인증 보안 정책을 어떻게 유지할 것인지는 별도의 Failure Architecture 및 ADR에서 정의한다.

 ## 10. Infrastructure Architecture

 현재 시스템은 Docker Compose 기반으로 여러 실행 단위를 구성한다.
 주요 서비스는 다음과 같다.

 | Component | Responsibility |
| --- | --- |
| Nginx | 외부 요청 진입점 및 Reverse Proxy |
| Backend | REST API 및 비즈니스 로직 |
| MySQL | 영속 데이터 저장 |
| Redis | Token 상태 관리 |
| Prometheus | Metrics 수집 |
| VictoriaMetrics | Metrics 저장 |
| Grafana | Monitoring Dashboard |
| Node Exporter | Host Metrics |
| cAdvisor | Container Metrics |

 현재 구성에서는 Docker 내부 네트워크를 통해 각 서비스가 통신한다.

 ## 11. Monitoring Architecture

 Backend는 Actuator 기반 Metrics를 제공하고 Monitoring Stack에서 이를 수집한다.

```txt
Spring Boot
     │
     │ Metrics
     ▼
Prometheus
     │
     ├──────────────→ VictoriaMetrics
     │
     ▼
Grafana
```

 Infrastructure 및 Container Metric은 Node Exporter와 cAdvisor를 통해 수집된다.

 ## 12. Port and Network Overview

 현재 Docker Compose 환경의 주요 통신 구조는 다음과 같다.

 | Service | Container Port | Host Port | Purpose |
| --- | --- | --- | --- |
| Nginx | 80 | 80 | External Entry Point |
| Backend | 8080 | 8080 | REST API |
| MySQL | 3306 | 3307 | Database |
| Redis | 6379 | Internal | Token State |
| Prometheus | 9090 | 9090 | Metrics |
| VictoriaMetrics | 8428 | 8428 | Metrics Storage |
| Grafana | 3000 | 3000 | Monitoring |

 세부적인 포트 및 Docker Network 설정은 `container.txt`와 `system-context.txt`에서 관리한다.

 ## 13. Architecture Traceability

 Architecture는 01-prd의 요구사항을 다음과 같이 구현 구조와 연결한다.

 ### PRD-FUNC-001 Authentication

```txt
PRD-FUNC-001 Authentication
        │
        ▼
Authentication Architecture
        │
        ├── AuthController
        ├── AuthService
        ├── JwtProvider
        └── JwtAuthenticationFilter
```

 ### PRD-FUNC-002 User

```txt
PRD-FUNC-002 User
        │
        ▼
User Architecture
        │
        ├── User
        ├── UserController
        ├── UserService
        └── UserRepository
```

 ### PRD-FUNC-003 IAM

```txt
PRD-FUNC-003 IAM
        │
        ▼
Authorization Architecture
        │
        ├── User
        ├── Role
        ├── Permission
        ├── UserRoleService
        ├── RolePermissionService
        └── UserAuthorityService
```

 ### PRD-SEC-001 JWT

```txt
PRD-SEC-001 JWT
        │
        ▼
JwtProvider
JwtAuthenticationFilter
SecurityConfig
```

 ### PRD-SEC-002 Refresh Token

```txt
PRD-SEC-002 Refresh Token
        │
        ▼
AuthService
RefreshTokenRepository
Redis
```

 ### PRD-SEC-003 RBAC

```txt
PRD-SEC-003 RBAC
        │
        ▼
UserAuthorityService
Role
Permission
SecurityConfig
```

 ### PRD-SEC-005 Redis Failure

```txt
PRD-SEC-005 Redis Failure
        │
        ▼
Redis
RefreshTokenRepository
TokenBlacklistService
Failure Handling
```

 ## 14. Architecture Boundary

 본 문서는 시스템의 현재 구조와 구성요소 간 관계를 정의한다.

 다음 사항은 Architecture 상세 문서 또는 ADR에서 구체적으로 정의한다.

 - JWT Claim 및 서명 정책
- Refresh Token Rotation 세부 정책
- Replay 방어 방식
- Redis 장애 처리 방식
- RBAC 권한 판정 방식
- Database Schema
- Transaction 경계
- Docker 세부 구성
- Monitoring 및 운영 정책

 이러한 세부 설계 결정은 Architecture 문서에서 임의로 확정하지 않고 ADR을 통해 결정한다.

 ## 15. Architecture → ADR → Implementation

 전체 추적 구조는 다음과 같다.

```txt
PRD
 ↓
Architecture
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

 Architecture는 요구사항을 실제 시스템 구성요소와 연결하는 중간 계층이며, 특정 구현 선택의 의사결정 근거는 ADR에서 관리한다.

