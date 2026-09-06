

# System Context

## 1. 목적

본 문서는 시스템과 외부 사용자 및 주요 외부/인프라 구성요소 사이의 관계를 정의한다.

시스템 내부의 상세 구현이나 클래스 구조는 본 문서의 범위에 포함하지 않는다.

본 문서는 `architecture-overview.txt`의 전체 시스템 구조를 시스템 경계 관점에서 표현하고, 이후 Container 및 Component Architecture의 기준을 제공한다.

## 2. System Context

전체 시스템의 Context는 다음과 같다.

```text
                         ┌──────────────────┐
                         │      User        │
                         │                  │
                         │ Web Browser /    │
                         │ API Client       │
                         └────────┬─────────┘
                                  │
                                  │ HTTP
                                  ▼
                         ┌──────────────────┐
                         │      Nginx       │
                         │                  │
                         │ Reverse Proxy /  │
                         │ Entry Point      │
                         └────────┬─────────┘
                                  │
                                  │ HTTP
                                  ▼
                    ┌──────────────────────────┐
                    │                          │
                    │     SA-1 Backend        │
                    │                          │
                    │  Authentication / IAM   │
                    │  Business Services      │
                    │                          │
                    └───────┬──────────┬───────┘
                            │          │
                 ┌──────────┘          └──────────┐
                 │                                │
                 ▼                                ▼
        ┌─────────────────┐             ┌─────────────────┐
        │      MySQL      │             │      Redis      │
        │                 │             │                 │
        │ Persistent Data │             │ Token State     │
        └─────────────────┘             └─────────────────┘

                    ┌──────────────────────────┐
                    │     Monitoring Stack     │
                    │                          │
                    │ Prometheus / Victoria-   │
                    │ Metrics / Grafana        │
                    └────────────▲─────────────┘
                                 │
                                 │ Metrics
                                 │
                           SA-1 Backend
````

 ## 3. System Boundary

 ### 3.1 System Boundary

 SA-1의 핵심 시스템 경계는 다음과 같다.

```txt
┌──────────────────────────────────────────────┐
│                  SA-1 System                 │
│                                              │
│   Nginx                                      │
│      │                                       │
│      ▼                                       │
│   Spring Boot Backend                        │
│      │                                       │
│      ├── Authentication                      │
│      ├── Authorization / IAM                 │
│      └── Application Services                │
│                                              │
└──────────────────────────────────────────────┘
```

 MySQL과 Redis는 Backend가 사용하는 외부 상태 관리 인프라로 취급한다.

 Monitoring Stack은 시스템의 운영 상태를 관측하기 위한 외부 인프라 영역으로 취급한다.

 ## 4. External Actors

 ### 4.1 User

 사용자는 Web Browser 또는 API Client를 통해 시스템에 접근한다.

 주요 상호작용은 다음과 같다.

 - 인증 요청
- Access Token을 이용한 보호 API 요청
- Refresh Token을 이용한 인증 상태 갱신
- 로그아웃
- 사용자 기능 이용
- 권한에 따른 보호 기능 이용

 사용자는 시스템 내부의 인증 및 인가 구현 방법을 직접 알 필요가 없다.

 ## 5. Entry Point

 ### 5.1 Nginx

 Nginx는 외부 Client와 SA-1 Backend 사이의 진입점 역할을 한다.

```txt
User / Client
      │
      │ HTTP
      ▼
    Nginx
      │
      │ API Request
      ▼
   Backend
```

 Nginx의 주요 책임은 다음과 같다.

 - 외부 HTTP 요청 수신
- API 요청 Reverse Proxy
- Frontend와 Backend 간 진입점 제공
- 필요한 Header 전달

 Backend 내부의 인증 및 인가 판단은 Nginx가 아닌 Backend Security Layer에서 수행한다.

 ## 6. Backend Context

 Backend는 시스템의 핵심 애플리케이션 영역이다.

 주요 책임은 다음과 같다.

 ### Authentication

 사용자의 인증과 인증 상태 관리를 담당한다.

```txt
Authentication
 ├── Login
 ├── Access Token
 ├── Refresh Token
 └── Logout / Invalidation
```

 ### IAM / Authorization

 사용자와 Role 및 Permission을 기반으로 접근 권한을 관리한다.

```txt
User
  ↓
Role
  ↓
Permission
  ↓
Protected Resource
```

 ### Application Services

 인증 및 IAM 이외의 시스템 기능에 대한 비즈니스 로직을 수행한다.

 ## 7. MySQL Context

 MySQL은 Backend가 사용하는 영속 데이터 저장소다.

```txt
Backend
   │
   │ Persistent Data
   ▼
 MySQL
```

 MySQL에는 시스템에서 장기적으로 유지해야 하는 애플리케이션 데이터가 저장된다.

 대표적인 데이터 영역은 다음과 같다.

 - User
- Role
- Permission
- IAM 관련 관계 데이터
- 기타 애플리케이션 영속 데이터

 구체적인 Schema 및 Entity 관계는 `data-flow.txt` 및 이후 Data Architecture에서 정의한다.

 ## 8. Redis Context

 Redis는 Backend가 Token 관련 상태를 관리하기 위해 사용하는 상태 저장 인프라다.

```txt
Backend
   │
   ├── Refresh Token State
   │
   └── Token Invalidation State
            │
            ▼
          Redis
```

 Redis는 특히 다음 인증 기능과 관련된다.

 - Refresh Token 상태 관리
- Refresh Token Rotation
- Token Invalidation / Blacklist
- TTL 기반 상태 관리

 Redis 장애 상황에서의 보안 처리 정책은 `failure-topology.txt` 및 관련 ADR에서 정의한다.

 ## 9. Monitoring Context

 시스템의 운영 상태는 Monitoring Stack을 통해 관측한다.

```txt
             Metrics
                │
                ▼
       ┌─────────────────┐
       │  SA-1 Backend   │
       └────────┬────────┘
                │
                ▼
          ┌───────────┐
          │ Prometheus│
          └─────┬─────┘
                │
                ▼
       ┌─────────────────┐
       │ VictoriaMetrics │
       └────────┬────────┘
                │
                ▼
          ┌──────────┐
          │ Grafana  │
          └──────────┘
```

 Monitoring 시스템은 애플리케이션의 핵심 비즈니스 처리에는 직접 참여하지 않는다.

 주요 목적은 다음과 같다.

 - Application Metrics 수집
- Infrastructure Metrics 수집
- Container Metrics 수집
- Dashboard 제공
- 장애 및 성능 상태 관측

 ## 10. Main Interaction Flows

 ### 10.1 Authentication

```txt
User
  │
  │ Login
  ▼
Nginx
  │
  ▼
Backend
  │
  ├── Authenticate User
  ├── Issue Access Token
  └── Issue Refresh Token
```

 ### 10.2 Authenticated Request

```txt
User
  │
  │ Access Token
  ▼
Nginx
  │
  ▼
Backend
  │
  ├── Authenticate
  ├── Authorize
  └── Process Request
```

 ### 10.3 Token Refresh

```txt
User
  │
  │ Refresh Token
  ▼
Nginx
  │
  ▼
Backend
  │
  ▼
Redis
  │
  └── Token State
        │
        ▼
    Backend
        │
        └── New Authentication State
```

 ### 10.4 IAM Management

```txt
Authorized Admin
      │
      ▼
    Nginx
      │
      ▼
   Backend
      │
      ├── User Management
      ├── Role Management
      ├── Permission Management
      └── Menu Management
            │
            ▼
          MySQL
```

 ## 11. Security Boundary

 인증 및 인가의 책임은 Backend 내부에 있다.

```txt
External Request
       │
       ▼
     Nginx
       │
       ▼
Authentication
       │
       ▼
Authorization
       │
       ▼
Protected Resource
```

 Nginx가 요청을 Backend로 전달했다고 해서 요청이 인증 또는 인가된 것으로 간주하지 않는다.

 Backend는 보호된 기능에 대해 독립적으로 인증 및 인가를 수행해야 한다.

 ## 12. Failure Boundary

 주요 외부 상태 인프라 장애 경계는 다음과 같다.

```txt
                Backend
               /       \
              /         \
             ▼           ▼
          MySQL        Redis
             │           │
             X           X
          DB Failure   Redis Failure
```

 각 장애가 Backend에 미치는 영향은 별도로 정의한다.

 특히 Redis 장애는 Refresh Token 및 Token Invalidation과 직접적으로 연결되므로 인증 보안 관점에서 별도의 Failure Architecture가 필요하다.

 ## 13. Context Responsibilities

 | Context | Responsibility |
| --- | --- |
| User / Client | 시스템 기능 요청 |
| Nginx | External Entry Point / Reverse Proxy |
| Backend | Business / Authentication / Authorization |
| MySQL | Persistent Data |
| Redis | Token State |
| Prometheus | Metrics Collection |
| VictoriaMetrics | Metrics Storage |
| Grafana | Monitoring Visualization |

 ## 14. Architecture Traceability

 System Context는 다음 PRD와 연결된다.

 | Architecture Area | Related PRD |
| --- | --- |
| Authentication | PRD-FUNC-001-authentication |
| User | PRD-FUNC-002-user |
| IAM | PRD-FUNC-003-iam |
| JWT | PRD-SEC-001-jwt |
| Refresh Token | PRD-SEC-002-refresh-token |
| RBAC | PRD-SEC-003-rbac |
| Token Invalidation | PRD-SEC-004-token-invalidation |
| Redis Failure | PRD-SEC-005-redis-failure |

 ## 15. Boundary of This Document

 본 문서는 시스템과 외부 구성요소 간의 관계를 정의한다.

 다음 내용은 본 문서에서 상세하게 정의하지 않는다.

 - Controller / Service / Repository 구조
- Spring Security Filter Chain 상세 구조
- JWT Claim 상세 구조
- Refresh Token Rotation 구현
- Redis Key Schema
- Database Schema
- Transaction 경계
- Docker Container 상세 구성
- 구체적인 장애 복구 전략

 해당 내용은 다음 Architecture 문서 및 ADR에서 정의한다.

 ## 16. Next Architecture Layers

 System Context 이후 상세 Architecture는 다음 순서로 내려간다.

```txt
System Context
      ↓
Container Architecture
      ↓
Component Architecture
      ↓
Authentication Flow
      ↓
Authorization Flow
      ↓
Data Flow
      ↓
Failure Topology
```

 각 문서는 현재 구현된 시스템을 기준으로 작성하며, 구현에 존재하지 않는 구성요소나 동작을 Architecture에 임의로 추가하지 않는다.

