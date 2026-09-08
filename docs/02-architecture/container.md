
# Container Architecture

# 1. 목적
본 문서는 SA-1 시스템을 구성하는 주요 실행 단위(Container)와 Container 간 통신 관계를 정의한다.
system-context.txt에서 정의한 시스템 경계를 실제 실행 환경의 구성요소 수준으로 구체화한다.
본 문서는 현재 apms-sr 의 Docker Compose 및 Backend 구조를 기준으로 작성하며, 구현에 존재하지 않는 Container나 통신 경로를 임의로 정의하지 않는다.

# 2. Container Overview

현재 시스템의 주요 실행 단위는 다음과 같다.
```txt
                           Client
                              │
                              │ HTTP
                              ▼
                    ┌──────────────────┐
                    │      Nginx       │
                    │ Reverse Proxy    │
                    └────────┬─────────┘
                             │
                             │ HTTP
                             ▼
                    ┌──────────────────┐
                    │     Backend      │
                    │   Spring Boot    │
                    └──────┬─────┬─────┘
                           │     │
              ┌────────────┘     └────────────┐
              │                               │
              ▼                               ▼
       ┌──────────────┐                ┌──────────────┐
       │    MySQL     │                │    Redis     │
       │  Persistent  │                │ Token State  │
       │     Data     │                │              │
       └──────────────┘                └──────────────┘

                    Monitoring Containers
                           │
                           ▼
                  ┌──────────────────┐
                  │    Prometheus    │
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │ VictoriaMetrics  │
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │     Grafana      │
                  └──────────────────┘

              Host / Container Metrics
                     │
              ┌──────┴──────┐
              ▼             ▼
        Node Exporter     cAdvisor
```

# 3. Container 목록

| Container | 역할 | 주요 통신 대상 |
|---|---|---|
| Nginx | 외부 요청 진입점 / Reverse Proxy | Client, Backend |
| Backend | REST API / Business / Auth / IAM | Nginx, MySQL, Redis, Monitoring |
| MySQL | 영속 데이터 저장 | Backend |
| Redis | Token 관련 상태 저장 | Backend |
| Prometheus | Metrics 수집 | Backend, Exporters |
| VictoriaMetrics | Metrics 저장 | Prometheus |
| Grafana | Metrics 시각화 | VictoriaMetrics / Prometheus |
| Node Exporter | Host Metrics 제공 | Prometheus |
| cAdvisor | Container Metrics 제공 | Prometheus |

# 4. Nginx Container
Responsibility

Nginx Container는 외부 HTTP 요청을 수신하고 Backend로 API 요청을 전달한다.
```txt
Client
  │
  │ HTTP
  ▼
Nginx
  │
  │ /api/*
  ▼
Backend
```

주요 책임은 다음과 같다.

* 외부 HTTP 요청 수신
* Reverse Proxy
* API 요청 전달
* Frontend 정적 리소스 제공
* 필요한 HTTP Header 전달

인증 및 인가의 최종 판단은 Backend에서 수행한다.

# 5. Backend Container

Backend Container는 Spring Boot Application을 실행한다.

주요 책임은 다음과 같다.
```txt
Backend
├── Authentication
├── Authorization / IAM
├── Application Services
├── Persistence
└── Monitoring Endpoint
```

Backend는 시스템의 핵심 애플리케이션 Container이며 다음 Container와 통신한다.
```txt
Backend
 ├──→ MySQL
 ├──→ Redis
 └──→ Monitoring Stack
```
# 6. Authentication Container Boundary

Authentication은 별도의 Container가 아니라 Backend Container 내부의 애플리케이션 모듈이다.
```txt
Backend Container
│
└── auth
    ├── security
    └── jwt
```

주요 구성요소는 다음과 같다.

* AuthController
* AuthService
* JwtProvider
* JwtAuthenticationFilter
* SecurityConfig
* RefreshTokenRepository
* TokenBlacklistService
* UserAuthorityService


따라서 다음과 같이 표현한다.
```txt
Client
  │
  ▼
Nginx Container
  │
  ▼
Backend Container
  │
  └── Authentication Module
```

Authentication을 독립적인 Microservice로 분리하지 않는다.

# 7. IAM Container Boundary

IAM 역시 별도의 Container가 아니라 Backend Container 내부의 모듈이다.
```txt
Backend Container
│
└── iam
    ├── user
    ├── role
    ├── permission
    ├── admin
    └── menu
```

주요 책임은 다음과 같다.

* User 관리
* Role 관리
* Permission 관리
* User-Role 관계 관리
* Role-Permission 관계 관리
* Menu 관리
* 관리자 기능 제공

구조적으로 다음과 같다.
```txt
Backend Container
│
└── IAM Module
      │
      ├── User
      ├── Role
      ├── Permission
      └── Menu
```
# 8. MySQL Container

MySQL Container는 영속 데이터를 저장한다.
```txt
Backend
   │
   │ JDBC / Database Connection
   ▼
MySQL
```

주요 데이터 영역은 다음과 같다.

* User
* Role
* Permission
* User-Role 관계
* Role-Permission 관계
* 기타 애플리케이션 데이터

MySQL은 애플리케이션의 장기 영속 상태를 담당한다.

# 9. Redis Container

Redis Container는 Token 관련 상태를 저장한다.
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

주요 사용 목적은 다음과 같다.

* Refresh Token 상태 관리
* Refresh Token Rotation 관련 상태
* Token Blacklist / Invalidation
* TTL 기반 상태 관리

Redis는 인증 기능의 일부 상태를 담당하지만 인증 로직 자체를 수행하지 않는다.

인증 정책과 Token 검증은 Backend에서 수행한다.

# 10. Monitoring Containers
## 10.1 Prometheus

Prometheus는 Backend 및 Exporter의 Metrics를 수집한다.
```txt
Backend ────────┐
                │
Node Exporter ──┼──→ Prometheus
                │
cAdvisor ───────┘
```
## 10.2 VictoriaMetrics

VictoriaMetrics는 Metrics의 장기 저장을 담당한다.
```txt
Prometheus
     │
     ▼
VictoriaMetrics
```
## 10.3 Grafana

Grafana는 수집된 Metrics를 Dashboard 형태로 제공한다.
```txt
Metrics Storage
      │
      ▼
   Grafana
```

Monitoring Container는 사용자 인증 및 비즈니스 요청의 처리 경로에 직접 포함되지 않는다.

# 11. Node Exporter

Node Exporter는 Host 수준의 Metrics를 제공한다.
```txt
Host
 │
 ▼
Node Exporter
 │
 ▼
Prometheus
```

애플리케이션 비즈니스 로직에는 관여하지 않는다.

# 12. cAdvisor

cAdvisor는 Container 수준의 리소스 Metrics를 제공한다.
```txt
Docker Containers
       │
       ▼
    cAdvisor
       │
       ▼
   Prometheus
```

주요 관측 대상은 Container의 CPU, Memory 및 기타 Runtime Metric이다.

# 13. Container Communication

주요 통신 관계는 다음과 같다.
```txt
Client
  │
  │ HTTP
  ▼
Nginx
  │
  │ HTTP
  ▼
Backend
  │
  ├──────────────→ MySQL
  │
  └──────────────→ Redis
```

Monitoring 경로는 별도로 존재한다.
```txt
Backend
   │
   │ Metrics
   ▼
Prometheus
   │
   ▼
VictoriaMetrics
   │
   ▼
Grafana
```

Infrastructure Metrics는 다음과 같다.
```txt
Host ───────────→ Node Exporter ──┐
                                  │
Containers ────→ cAdvisor ───────┼──→ Prometheus
                                  │
Backend ──────────────────────────┘
```
# 14. Network Boundary

Container 간 내부 통신은 Docker Network를 통해 이루어진다.

외부 Client는 Backend Container에 직접 접근하는 대신 Nginx를 통해 접근한다.
```txt
External Network
       │
       ▼
    Nginx
       │
       ▼
Internal Docker Network
       │
       ├── Backend
       ├── MySQL
       ├── Redis
       └── Monitoring
```

따라서 외부 요청 경로와 내부 서비스 간 통신 경로를 구분한다.

# 15. Port Overview

현재 구성의 주요 Port는 다음과 같다.

| Service | Container Port | Host Port | 용도 |
|---|---:|---:|---|
| Nginx | 80 | 80 | External HTTP |
| Backend | 8080 | 8080 | REST API |
| MySQL | 3306 | 3307 | Database |
| Redis | 6379 | Internal | Token State |
| Prometheus | 9090 | 9090 | Metrics |
| VictoriaMetrics | 8428 | 8428 | Metrics Storage |
| Grafana | 3000 | 3000 | Dashboard |


실제 Docker Compose 설정과 환경별 Port 차이가 존재하는 경우 Compose 설정을 우선한다.

# 16. Request Flow
## 16.1 일반 API 요청
```txt
Client
  │
  ▼
Nginx :80
  │
  ▼
Backend :8080
  │
  ├── Business Logic
  │
  └── MySQL :3306
```
## 16.2 Authentication Request
```txt
Client
  │
  ▼
Nginx
  │
  ▼
Backend
  │
  ├── AuthService
  ├── JwtProvider
  └── Refresh Token State
          │
          ▼
        Redis
```
## 16.3 Authorization Request
```txt
Client
  │
  │ Access Token
  ▼
Nginx
  │
  ▼
Backend
  │
  ├── JWT Authentication
  ├── User Authority
  ├── Role
  └── Permission
          │
          ▼
      Protected API
```
# 17. Failure Boundaries

Container 장애가 발생했을 때 영향을 받는 주요 영역은 다음과 같다.

| 장애 Container | 주요 영향 |
|---|---|
| Nginx | 외부 HTTP 접근 불가 |
| Backend | API 및 인증/인가 기능 사용 불가 |
| MySQL | 영속 데이터 접근 기능 영향 |
| Redis | Refresh Token / Token Invalidation 기능 영향 |
| Prometheus | Metrics 수집 영향 |
| VictoriaMetrics | Metrics 저장 영향 |
| Grafana | Dashboard 조회 영향 |
| Node Exporter | Host Metrics 수집 영향 |
| cAdvisor | Container Metrics 수집 영향 |


특히 Redis 장애는 인증 보안 정책과 직접적으로 연결되므로 별도의 failure-topology.txt에서 상세하게 다룬다.

# 18. Architecture Traceability

Container Architecture는 다음 PRD 및 Architecture 문서와 연결된다.

| Container / Module | Related Requirement |
|---|---|
| Backend / Authentication | PRD-FUNC-001-authentication |
| Backend / User | PRD-FUNC-002-user |
| Backend / IAM | PRD-FUNC-003-iam |
| JwtProvider / Security | PRD-SEC-001-jwt |
| Redis / Refresh Token | PRD-SEC-002-refresh-token |
| UserAuthority / Role / Permission | PRD-SEC-003-rbac |
| Redis | PRD-SEC-004-redis-failure |


# 19. Container Boundary

본 문서에서는 실행 단위와 Container 간 통신 관계를 정의한다.

다음 내용은 본 문서의 상세 범위가 아니다.

* Controller / Service / Repository 내부 구조
* JWT Claim 상세
* Security Filter Chain 상세
* Refresh Token Rotation 알고리즘
* Redis Key 구조
* Database Table 및 Entity 관계
* 개별 Component 간 상세 호출 관계

해당 내용은 다음 Architecture 문서에서 정의한다.

# 20. Next Architecture Layer

Container Architecture 다음에는 Backend 내부의 Component 구조를 정의한다.
```txt
System Context
      ↓
Container
      ↓
Component
      ↓
Authentication Flow
      ↓
Authorization Flow
      ↓
Data Flow
      ↓
Failure Topology
```

Component Architecture에서는 실제 Backend Package 및 클래스 구조를 기준으로 Controller, Service, Domain, Repository, Security Component의 관계를 구체화한다.