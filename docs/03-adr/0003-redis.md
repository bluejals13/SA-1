

# ADR-0003: Redis

**Status:** Accepted
**Date:** 2026-09-06
**Decision Type:** Architecture
**Related PRD:** PRD-SEC-002-refresh-token, PRD-SEC-004-redis-failure
**Related TASK:** TBD
**Related Repository:** `26-05adf`

---

## 1. Context & Drivers

APMS.SR 시스템은 Authentication Flow를 처리하는 과정에서 Access Token의 갱신(Refresh)과 무효화(Invalidation/Logout) 상태를 관리해야 한다. 

이러한 Token 관련 상태 데이터는 영구적으로 보존되어야 하는 IAM 데이터(MySQL)와 달리, 수명(TTL)이 제한적이며 매우 빈번하고 빠른 읽기/쓰기가 발생한다는 특징이 있다. 

따라서 `ADR-0001`에서 결정한 단일 Application 구조를 유지하면서, 짧은 수명의 보안 상태 데이터를 안전하고 효율적으로 관리할 수 있는 아키텍처 결정이 필요하다.

본 결정의 주요 판단 기준은 다음과 같다.

1. 짧은 수명(Time-bounded) 데이터의 생명주기(TTL) 관리 용이성
2. 인증 과정에서 발생하는 높은 읽기/쓰기 처리 성능 (Low Latency)
3. 영속성 데이터(MySQL)와의 책임 및 장애 격리(Failure Isolation)
4. Docker Compose 기반 실행 환경의 재현성
5. 현재 시스템 규모에 적합한 운영 단순성

---

## 2. Options Considered

### Option A — Single Redis Instance

하나의 Redis 인스턴스를 Token State(Refresh Token, Blacklist) 전용 저장소로 사용한다.

**Advantages**

* In-Memory 기반으로 읽기/쓰기 성능이 매우 뛰어나다.
* Native TTL(Time-To-Live) 기능을 제공하여 만료된 Token State를 애플리케이션의 개입 없이 자동으로 정리할 수 있다.
* MySQL에 가해지는 인증 관련 부하를 원천적으로 분리할 수 있다.
* 단일 인스턴스 구성으로 Docker Compose를 통한 개발 및 테스트 환경 구성이 매우 단순하다.

**Disadvantages**

* 단일 인스턴스 장애 시 Refresh Token 발급 및 Logout 처리에 즉각적인 영향을 미친다.
* 메모리 기반이므로 장애 또는 재시작 시 Persistence 설정(AOF/RDB)에 따라 데이터 유실 가능성이 존재한다.

---

### Option B — Store Token State in MySQL

기존에 채택한 MySQL(Option A of ADR-0002)에 Token State 테이블을 추가하여 함께 관리한다.

**Advantages**

* 새로운 Infrastructure(Redis)를 추가할 필요가 없어 운영 요소가 줄어든다.
* 트랜잭션을 통해 도메인 데이터와 인증 데이터를 일관되게 롤백할 수 있다.

**Disadvantages**

* 모든 인증(검증, 갱신, 로그아웃) 요청이 MySQL에 집중되어 영속성 데이터 조회 성능에 병목을 유발할 수 있다.
* 만료된 Token을 삭제하기 위한 별도의 Batch 또는 Scheduler 구현이 강제된다.
* 짧은 수명의 데이터를 RDBMS에 끊임없이 쓰고 지우는 과정에서 불필요한 I/O 오버헤드가 발생한다.

---

### Option C — Redis Cluster / Sentinel

고가용성(HA)을 보장하기 위해 다중 노드로 구성된 Redis Cluster 또는 Sentinel 구조를 도입한다.

**Advantages**

* Master 노드 장애 시 자동으로 Failover가 수행되어 인증 기능의 중단을 방지한다.
* 대규모 트래픽 발생 시 확장이 용이하다.

**Disadvantages**

* 인프라 구성 및 유지보수 복잡도가 급격히 증가한다.
* `ADR-0001`에서 정의한 '현재 프로젝트 규모에서의 단순성' 원칙에 위배된다.
* 분산 환경에서의 네트워크 파티션 및 동기화 지연 문제를 추가로 고려해야 한다.

---

## 3. Decision

현재 시스템의 Authentication State 저장소로
Option A — Single Redis Instance를 채택한다.

Refresh Token 및 Token Invalidation과 같이 수명과 상태 관리가
필요한 인증 관련 데이터를 Redis에서 관리한다.

Redis의 TTL 기능을 활용하여 인증 상태의 만료 생명주기를 관리하고,
MySQL이 담당하는 영속적인 사용자 및 권한 데이터와 인증 상태를
분리한다.

현재 시스템 규모에서는 고가용성을 위한 Redis Cluster 또는
Sentinel 구성을 도입하지 않는다.

Redis 장애 시 인증 및 인가 기능의 처리 방식은
PRD-SEC-004-redis-failure에 정의된 보안 정책을 따른다.


---

## 4. Rationale

### 4.1 Automated Lifecycle Management (TTL)

Refresh Token 및 Token Invalidation State는
영구적으로 보존해야 하는 도메인 데이터와 달리
명확한 유효 기간을 가진다.

Redis의 TTL을 활용하여 인증 상태의 만료 생명주기를 관리하고,
정상적인 만료 처리를 위해 별도의 애플리케이션 Scheduler에
의존하지 않는 구조를 선택한다.

### 4.2 Separation of Concerns & Isolation

MySQL은 "사용자 및 권한 관계가 어떻게 구성되어 있는가(Long-lived)"를 책임지고, Redis는 "현재 사용자의 인증 세션 상태가 어떠한가(Short-lived)"를 책임진다. 이를 분리함으로써 데이터베이스 부하를 격리하고, 장애 전파를 차단할 수 있다.

### 4.3 Operational Simplicity

`ADR-0001`과 `ADR-0002`의 결정과 마찬가지로, 현재 단계에서는 불필요한 분산 시스템 복잡도를 추가하지 않는다. 단일 Redis 컨테이너는 운영이 단순하며 로컬 및 통합 테스트 환경에서 일관되게 재현할 수 있다.

---

## 5. Consequences

### Positive

#### 성능 향상
In-Memory 저장소의 특성상 API Latency(특히 Token Refresh 및 인가 필터 단계)를 최소화할 수 있다.

#### 구조적 단순함
단일 Redis 인스턴스를 사용하여 배포 파이프라인 및 로컬 테스트 환경(Docker Compose)의 복잡도를 낮게 유지할 수 있다.

#### 유연한 스키마
JSON 형태나 Key-Value 형태로 토큰 탈취 방지(Replay Detection) 등을 위한 부가적인 상태 데이터를 쉽게 확장하여 저장할 수 있다.

---

### Negative

#### Single Point of Failure (SPOF)
Redis 인스턴스가 다운될 경우, Token Refresh 흐름과 Token Blacklist 검증 로직이 실패하게 된다.

#### Data Volatility
Redis는 MySQL과 동일한 수준의 영속성을 보장하는 저장소가 아니므로,
Redis 장애 또는 재시작에 따른 인증 상태 손실 가능성을 고려해야 한다.

---

## 6. Verification

본 ADR의 결정은 다음 항목을 실제 실행을 통해 검증한다.

### 6.1 TTL & expiration

* Refresh Token의 TTL이 정의된 만료 시간과 일치하는지 확인한다.
* 만료된 Refresh Token이 더 이상 유효한 인증 상태로 사용되지 않는지 확인한다.
* Blacklist 상태가 정의된 TTL 이후 더 이상 유효하지 않은 상태로 처리되는지 확인한다.


### 6.2 Latency

* `JwtAuthenticationFilter`의 Blacklist 조회 과정에서 발생하는
  Redis 접근 latency를 측정한다.
* Token Refresh 과정의 Redis 접근 latency를 측정한다.

### 6.3 Failure Handling (Redis Failure Topology)

* Redis 장애 발생 시 `AuthService`와 `JwtAuthenticationFilter`가
  Redis 연결 실패를 적절히 감지하는지 확인한다.
* Redis 장애 시의 인증/인가 처리 결과가
  PRD-SEC-004-redis-failure에 정의된 정책과 일치하는지 검증한다.

---

## 7. Conditions for Reconsideration

다음과 같은 상황이 실제 측정 또는 운영 결과로 확인되면 본 결정을 재검토한다.

### 7.1 High Availability Requirement

인증 인프라 장애로 인한 서비스 중단 시간이 비즈니스 한계치를 초과하여 자동 Failover(Sentinel/Cluster)가 필수로 요구되는 경우.

### 7.2 Memory Bottleneck

활성 세션 및 Refresh Token의 양이 급증하여 단일 Redis 인스턴스의 메모리 용량(Max Memory)을 초과하는 경우.

### 7.3 Data Persistence Criticality

Redis 장애 재기동 시 발생하는 일시적인 토큰 유실(AOF/RDB Gap)이 심각한 보안 취약점이나 사용자 불만으로 이어지는 경우.

---

## 8. Relationship to Other ADRs

| ADR | Decision |
| --- | --- |
| ADR-0001 | Application Architecture |
| ADR-0002 | Database Architecture |
| ADR-0003 | Redis |
| ADR-0004 | JWT |
| ADR-0005 | Refresh Token Rotation |
| ADR-0006 | RBAC |

`ADR-0001`은 단일 Backend 구조를 정의하며, `ADR-0002`는 영속성 데이터(MySQL)를 다룬다. 본 `ADR-0003`은 **휘발성 상태 관리(Redis)**의 아키텍처를 결정한다. 
Redis에 어떤 데이터를 어떻게 담고 검증할 것인지(Replay Defense 등)에 대한 구체적인 암호화 및 비즈니스 정책은 `ADR-0004`와 `ADR-0005`에서 정의한다.

---

## 9. Related Documents

### Architecture

* `docs/02-architecture/architecture-overview.md`
* `docs/02-architecture/data-flow.md`
* `docs/02-architecture/failure-topology.md`

### Requirements

* `docs/01-prd/security/PRD-SEC-002-refresh-token.md`
* `docs/01-prd/security/PRD-SEC-004-redis-failure.md`

### Implementation

* `26-05adf`

### Evidence

* `PR-1A1`

---

## 10. Decision Summary

> APMS.SR은 인증 과정에서 발생하는 짧은 수명(Time-bounded)의 상태 데이터(Refresh Token, Blacklist)를 관리하기 위해 **Single Redis Instance**를 채택한다.
>
> In-Memory 저장소와 TTL 기능을 활용하여 성능을 극대화하고 만료된 세션을 자동으로 정리하며, MySQL과의 데이터 책임을 분리하여 영속성 데이터베이스를 보호한다.
>
> 현재 단계에서는 고가용성을 위한 Redis Cluster를 도입하지 않으며,
  단일 노드 장애 시나리오는 PRD-SEC-004-redis-failure에 정의된
  애플리케이션의 장애 대응 정책을 통해 통제한다.
>
> 향후 가용성 요구사항 증가 또는 메모리 병목이 실제 확인될 경우
  새로운 ADR을 통해 Redis 아키텍처(Cluster/Sentinel)를 재검토한다.


