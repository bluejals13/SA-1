# ADR-0001: Application Architecture

* **Status:** Accepted
* **Date:** 2026-09-06
* **Decision Type:** Architecture
* **Related PRD:** TBD
* **Related TASK:** TBD
* **Related Repository:** `26-05adf`

---

## 1. Context

APMS.SR은 React 기반 Frontend와 Spring Boot 기반 Backend로 구성되는 Full-Stack Web Application이다.

현재 시스템은 인증, 인가, 사용자 및 도메인 기능, 데이터 접근, 외부 인프라 연동, 모니터링 및 배포까지 하나의 서비스 흐름 안에서 관리해야 한다.

아키텍처를 결정할 때 다음 요구사항을 고려해야 한다.

1. Frontend와 Backend의 책임을 명확하게 분리해야 한다.
2. Backend 내부에서 Presentation, Business Logic, Data Access의 책임을 분리해야 한다.
3. 인증 및 인가와 같은 보안 관심사를 애플리케이션의 핵심 비즈니스 로직과 적절히 분리해야 한다.
4. MySQL 및 Redis와 같은 Infrastructure와의 의존성을 관리할 수 있어야 한다.
5. 단일 애플리케이션으로 개발 및 검증 가능한 수준의 복잡도를 유지해야 한다.
6. 테스트를 통해 주요 기능과 보안 정책을 검증할 수 있어야 한다.
7. Docker Compose를 이용하여 개발 및 실행 환경을 재현할 수 있어야 한다.
8. 향후 성능, 장애 및 운영 상황을 측정하고 검증할 수 있어야 한다.

따라서 아키텍처는 단순히 기술을 많이 사용하는 방향이 아니라, **현재 요구사항을 충족하면서도 구현·테스트·운영·검증이 가능한 구조**를 기준으로 결정한다.

---

## 2. Decision Drivers

다음 기준을 중심으로 아키텍처를 평가한다.

### 2.1 Simplicity

현재 프로젝트 규모에서 불필요한 분산 시스템 복잡도를 추가하지 않는다.

### 2.2 Separation of Responsibilities

Frontend, API, Business Logic, Persistence, Infrastructure의 책임을 명확하게 분리한다.

### 2.3 Testability

인증, 인가, 비즈니스 로직 및 데이터 접근을 독립적으로 테스트할 수 있어야 한다.

### 2.4 Security

Authentication 및 Authorization을 일관된 보안 경계 안에서 처리할 수 있어야 한다.

### 2.5 Reproducibility

Docker Compose 등을 이용하여 동일한 시스템 구성을 재현할 수 있어야 한다.

### 2.6 Operability

Logging, Metrics, Health Check 등을 통해 실행 중인 시스템의 상태를 관찰할 수 있어야 한다.

### 2.7 Verifiability

보안, 성능, 장애 대응 등의 품질 특성을 실제 실행을 통해 검증할 수 있어야 한다.

---

## 3. Options Considered

### Option A — Monolithic Layered Architecture

하나의 Spring Boot Backend Application 안에서 여러 도메인 및 기능을 관리하고, 내부 계층을 분리한다.

```text
Browser
   ↓
Nginx
   ↓
React / Spring Boot
   ↓
┌───────────────────────────┐
│ Spring Boot Application   │
│                           │
│ Controller                │
│    ↓                      │
│ Service                   │
│    ↓                      │
│ Repository                │
└───────────────────────────┘
   ↓              ↓
 MySQL           Redis
```

**Advantages**

* 구조가 단순하다.
* 개발 및 배포 과정이 비교적 단순하다.
* 전체 요청 흐름을 추적하기 쉽다.
* Integration Test 및 E2E Test 구성이 용이하다.
* 현재 프로젝트 규모에서 운영 복잡도가 낮다.

**Disadvantages**

* 서비스 규모가 커질 경우 애플리케이션의 결합도가 증가할 수 있다.
* 독립적인 서비스별 확장이 어렵다.
* 하나의 배포 단위에 여러 기능이 포함된다.

---

### Option B — Microservices Architecture

기능 또는 도메인을 여러 독립적인 서비스로 분리한다.

```text
Browser
   ↓
API Gateway
   ↓
┌──────────┬──────────┬──────────┐
│ Auth     │ User     │ Domain   │
│ Service  │ Service  │ Service  │
└──────────┴──────────┴──────────┘
       ↓         ↓         ↓
    Database / Message Broker
```

**Advantages**

* 서비스 단위의 독립적인 배포가 가능하다.
* 서비스별 독립적인 확장이 가능하다.
* 장애 격리 및 조직 단위 분리가 용이할 수 있다.

**Disadvantages**

* 네트워크 통신과 서비스 간 계약이 추가된다.
* Distributed Transaction 문제가 발생할 수 있다.
* Logging, Monitoring, Deployment, Testing의 복잡도가 증가한다.
* 현재 프로젝트 규모에서는 문제보다 인프라 복잡도가 커질 가능성이 높다.

---

### Option C — Modular Monolith

하나의 애플리케이션으로 배포하되 내부적으로 도메인 및 모듈 경계를 강하게 분리한다.

**Advantages**

* Monolith의 단순성을 유지할 수 있다.
* 도메인 간 결합도를 낮출 수 있다.
* 향후 특정 모듈을 독립 서비스로 분리하기 위한 기반이 될 수 있다.

**Disadvantages**

* 초기 설계와 모듈 경계 관리가 추가로 필요하다.
* 실제 서비스 분리가 필요하지 않은 상황에서 추가적인 구조적 복잡도가 발생할 수 있다.

---

## 4. Decision

**Option A — Monolithic Layered Architecture**를 현재 시스템의 기본 아키텍처로 채택한다.

Backend는 하나의 Spring Boot Application으로 구성하고, 내부적으로 다음과 같은 책임 분리를 유지한다.

```text
Controller
    ↓
Service
    ↓
Repository
    ↓
Database
```

Authentication 및 Authorization과 같은 Cross-Cutting Security Concern은 Spring Security를 중심으로 처리한다.

Frontend는 React Application으로 분리하고, Nginx를 Reverse Proxy 및 정적 리소스 제공 계층으로 사용한다.

Infrastructure는 다음과 같이 구성한다.

```text
Browser
   ↓
Nginx
   ↓
React / Spring Boot
   ↓
┌───────────────┐
│ MySQL         │
│ Redis         │
└───────────────┘

Prometheus
   ↓
Grafana
```

실행 환경은 Docker Compose를 기준으로 구성하여 애플리케이션과 주요 Infrastructure의 실행 구성을 재현 가능하게 관리한다.

---

## 5. Why This Decision

### 5.1 Current Scale

현재 프로젝트의 핵심 문제는 서비스 간 분리 자체가 아니다.

현재 더 중요한 문제는 다음과 같다.

* Authentication의 정확성
* Authorization의 정확성
* Refresh Token 보안
* 데이터 접근 성능
* 장애 상황의 처리
* 성능 측정
* 운영 상태 관찰
* 테스트 결과의 재현성

따라서 현재 단계에서는 Microservices로 분리하는 것보다 **하나의 서비스에서 이러한 품질 특성을 깊게 검증하는 것**이 더 높은 가치를 가진다.

### 5.2 Reduced Distributed Complexity

Microservices를 도입하면 다음 문제가 추가된다.

* Service-to-Service communication
* API contract
* Distributed logging
* Distributed tracing
* Deployment coordination
* Failure propagation
* Distributed testing

이러한 문제들은 실제 요구사항이 발생하기 전까지는 프로젝트의 핵심 검증 대상이 아니다.

따라서 현재는 이를 의도적으로 도입하지 않는다.

### 5.3 Testability

하나의 Backend Application으로 구성하면 인증부터 데이터 접근까지의 전체 흐름을 Integration Test 및 E2E Test로 검증하기 쉽다.

특히 다음과 같은 보안 시나리오를 실제 실행으로 검증하는 것이 중요하다.

```text
Authentication
      ↓
Access Token
      ↓
Refresh Token
      ↓
Rotation
      ↓
Replay Detection
      ↓
Authorization
```

### 5.4 Operational Simplicity

현재 시스템은 Docker Compose를 통해 Application, Database, Redis 등의 실행 환경을 함께 구성할 수 있다.

이를 통해 개발 환경과 검증 환경의 차이를 줄이고, 장애 및 성능 테스트를 반복 실행할 수 있는 기반을 유지한다.

---

## 6. Consequences

### Positive

#### 단순한 배포 구조

하나의 Backend Application을 중심으로 배포할 수 있으므로 배포 구조가 단순하다.

#### 낮은 운영 복잡도

서비스 간 네트워크 통신 및 분산 시스템 관리가 필요하지 않다.

#### 통합 검증 용이

Authentication, Authorization, Business Logic, Persistence를 연결한 전체 요청 흐름을 검증하기 쉽다.

#### 재현 가능한 환경

Docker Compose를 이용하여 주요 실행 환경을 일관되게 구성할 수 있다.

#### 현재 문제에 집중 가능

불필요한 인프라 확장보다 Security, Performance, Failure Handling 및 Observability 검증에 집중할 수 있다.

---

### Negative

#### Scale-out 경계

전체 Backend Application이 하나의 배포 단위이므로 특정 기능만 독립적으로 확장하기 어렵다.

#### Application Coupling

프로젝트 규모가 커질수록 내부 모듈 간 결합도가 증가할 수 있다.

#### 장애 범위

하나의 Application에 문제가 발생할 경우 여러 기능에 영향을 줄 수 있다.

#### 향후 분리 비용

향후 실제 요구사항에 의해 Microservices가 필요해질 경우 도메인 경계를 다시 정리하고 서비스 분리 비용을 부담해야 한다.

---

## 7. Rejected Alternatives

### Microservices

현재 프로젝트에서 실제 서비스 분리를 요구하는 규모 또는 운영상의 문제가 확인되지 않았기 때문에 채택하지 않는다.

Microservices는 기술적 수준을 보여주기 위한 목적으로 도입하지 않는다.

### Kubernetes

현재 애플리케이션의 배포 및 운영 문제를 해결하기 위해 Kubernetes가 필요한 상황이 아니므로 본 ADR의 아키텍처 결정에 포함하지 않는다.

향후 실제 운영 요구사항 또는 배포/확장 문제로 필요성이 확인되는 경우 별도의 ADR로 결정한다.

### Kafka / Message Broker

현재 핵심 요구사항에 비동기 이벤트 처리 또는 서비스 간 메시징이 필수적이지 않으므로 도입하지 않는다.

### Redis Cluster

현재 Redis 사용 자체가 필요한 것과 Redis를 분산 Cluster로 운영해야 하는 것은 별개의 문제다.

현재 규모에서 Cluster 운영을 요구하는 장애 허용성 또는 확장성 요구사항이 확인되지 않았으므로 채택하지 않는다.

---

## 8. Conditions for Reconsideration

현재의 Monolithic Layered Architecture는 영구적인 구조적 제약이 아니다.

다음과 같은 문제가 실제로 발생하면 아키텍처를 재검토한다.

### 8.1 독립적인 Scale 요구

특정 기능의 트래픽이 전체 Application과 명확하게 다른 수준으로 증가하고 독립적인 확장이 필요해지는 경우.

### 8.2 배포 독립성 요구

특정 도메인의 변경을 전체 Application 배포 없이 독립적으로 배포해야 하는 요구가 발생하는 경우.

### 8.3 장애 격리 요구

특정 기능의 장애가 전체 Application에 영향을 주는 문제가 반복적으로 발생하고 구조적 격리가 필요한 경우.

### 8.4 조직 / 시스템 경계 변화

실제 운영 환경에서 도메인 또는 팀 단위의 독립적인 소유권이 필요해지는 경우.

### 8.5 측정 가능한 병목

현재 구조로 해결할 수 없는 성능 또는 운영상의 병목이 실제 측정 결과로 확인되는 경우.

아키텍처 변경이 필요해지면 기존 ADR을 수정하지 않고 **새로운 ADR을 생성하여 변경 이유와 영향을 기록한다.**

---

## 9. Verification

이 ADR의 결정은 문서만으로 완료되는 것이 아니다.

현재 아키텍처가 실제 요구사항을 만족하는지 다음 영역을 통해 검증한다.

### Functional Verification

* 주요 API 요청 흐름
* Authentication
* Authorization
* Business Logic
* Persistence

### Security Verification

* Authentication failure
* Authorization failure
* Refresh Token Rotation
* Refresh Token Replay
* Logout / Token Invalidation
* RBAC boundary

### Performance Verification

* API latency
* RPS
* p50 / p95 / p99
* Error Rate
* Database query performance
* Resource utilization

### Failure Verification

* Redis failure
* Database failure
* Application restart
* Invalid authentication state
* Deployment failure / rollback

### Operational Verification

* Health Check
* Logging
* Metrics
* Monitoring
* Recovery procedure

검증 결과는 테스트 코드의 존재만으로 판단하지 않고, 실제 실행 결과와 Evidence를 통해 확인한다.

---

## 10. Relationship to Other ADRs

이 ADR은 전체 Application Architecture의 기본 방향을 정의한다.

세부 기술 선택은 각각 별도의 ADR에서 관리한다.

| ADR      | Decision                 |
| -------- | ------------------------ |
| ADR-0001 | Application Architecture |
| ADR-0002 | Database Architecture    |
| ADR-0003 | Redis                    |
| ADR-0004 | JWT                      |
| ADR-0005 | Refresh Token Rotation   |
| ADR-0006 | RBAC                     |

따라서 이 문서는 MySQL, Redis, JWT, Refresh Token Rotation 또는 RBAC의 세부 구현 방식을 결정하지 않는다.

해당 결정은 각각의 ADR에서 관리한다.

---

## 11. Related Documents

### Architecture

* `docs/02-architecture/architecture-overview.md`
* `docs/02-architecture/system-context.md`
* `docs/02-architecture/container.md`
* `docs/02-architecture/component.md`
* `docs/02-architecture/data-flow.md`
* `docs/02-architecture/failure-topology.md`

### Requirements

* `docs/01-prd/`

### Tasks

* `docs/04-task-progress/`

### Implementation

* `26-05adf`

### Evidence

* `PR-1A1`

---

## 12. Decision Summary

> APMS.SR은 현재 요구사항과 프로젝트 규모를 기준으로 **Monolithic Layered Architecture**를 채택한다.
>
> Backend는 Spring Boot 단일 Application으로 구성하고 Controller → Service → Repository의 책임 분리를 유지한다.
>
> Frontend는 React로 분리하고 Nginx를 Reverse Proxy로 사용한다.
>
> MySQL과 Redis는 Application 외부 Infrastructure로 관리하며 Docker Compose를 통해 실행 환경을 재현한다.
>
> Microservices, Kubernetes, Kafka, Redis Cluster 등의 추가 분산 기술은 현재 요구사항에 포함하지 않는다.
>
> 향후 실제 성능, 장애, 배포 또는 운영 요구사항이 현재 구조의 한계를 입증하는 경우 새로운 ADR을 통해 아키텍처를 재검토한다.
