

# ADR-0002: Database Architecture

 **Status:** Accepted\
 **Date:** 2026-09-06\
 **Decision Type:** Architecture\
 **Related PRD:** PRD-FUNC-002-user, PRD-FUNC-003-iam\
 **Related TASK:** TBD\
 **Related Repository:** `26-05adf`

---

 ## 1. Context & Drivers

 APMS.SR 시스템은 User, Role, Permission 및 이들 간의 관계 데이터를 영구적으로 저장하고 관리해야 한다.

 특히 IAM 영역에서는 사용자와 역할, 역할과 권한 사이의 관계를 명확하게 관리해야 하며, 데이터 생성·수정·삭제 과정에서 데이터 정합성과 무결성을 보장할 필요가 있다.

 또한 `ADR-0001`에서 단일 Spring Boot Backend Application을 기본 아키텍처로 결정했으므로, 현재 시스템 규모에 적합하면서 개발·테스트 및 운영 환경에서 재현 가능한 Database 아키텍처가 필요하다.

 본 결정의 주요 판단 기준은 다음과 같다.

 1. 관계형 데이터 모델과의 적합성
2. 데이터 무결성 및 트랜잭션 일관성
3. 애플리케이션과의 통합 용이성
4. Docker Compose 기반 실행 환경의 재현성
5. 현재 시스템 규모에 적합한 운영 단순성
6. 향후 성능 및 확장성 검증 가능성

---

 ## 2. Options Considered

 ### Option A — Single MySQL Relational Database

 하나의 MySQL 데이터베이스를 Primary Persistent Database로 사용한다.

 User, Role, Permission 및 관련 관계 데이터를 관계형 모델로 관리하고, 애플리케이션의 영속 데이터를 하나의 데이터베이스에서 관리한다.

 **Advantages**

 - User, Role, Permission 간 관계를 명확하게 표현할 수 있다.
- Foreign Key 및 Transaction을 통한 데이터 무결성 관리가 용이하다.
- 복잡한 관계형 조회 및 Join에 적합하다.
- 현재 프로젝트 규모에서 운영 구조가 단순하다.
- Docker Compose를 이용하여 개발 및 테스트 환경을 재현하기 쉽다.

 **Disadvantages**

 - 단일 데이터베이스가 장애 지점이 될 수 있다.
- 데이터베이스 자체의 Scale-out에는 한계가 있다.
- 대규모 읽기 트래픽이 발생할 경우 Replica 등의 추가 구성이 필요할 수 있다.

---

 ### Option B — NoSQL Database

 MongoDB와 같은 NoSQL 데이터베이스를 사용한다.

 **Advantages**

 - 스키마 변경에 상대적으로 유연하다.
- 특정 형태의 비정형 데이터 저장에 적합하다.
- 데이터 구조에 따라 수평 확장이 용이할 수 있다.

 **Disadvantages**

 - 현재 IAM 데이터의 관계형 구조와 직접적인 적합성이 낮다.
- User, Role, Permission 간 관계를 별도로 관리해야 할 수 있다.
- 관계형 데이터에 대한 복잡한 조회 및 무결성 관리가 현재 요구사항과 비교하여 불필요하게 복잡해질 수 있다.
- 현재 프로젝트에서 NoSQL 도입으로 얻는 실질적인 이점이 제한적이다.

---

 ### Option C — Primary / Replica RDBMS

 Primary Database와 하나 이상의 Replica Database를 구성하여 쓰기와 읽기를 분리한다.

 **Advantages**

 - 읽기 트래픽을 Replica로 분산할 수 있다.
- 향후 대규모 조회 트래픽에 대응할 수 있다.
- 데이터베이스 읽기 확장에 유리하다.

 **Disadvantages**

 - Replication 구성이 추가된다.
- Replication Lag을 고려해야 한다.
- 장애 발생 시 Failover 정책이 필요하다.
- 개발 및 테스트 환경의 복잡도가 증가한다.
- 현재 시스템 규모에서는 필요한 복잡도보다 운영 비용이 크다.

---

 ## 3. Decision

 현재 시스템의 Primary Persistent Database로 **Option A — Single MySQL Relational Database**를 채택한다.

 MySQL은 User, Role, Permission 및 관련 관계 데이터를 관계형 모델로 관리하기에 적합하며, Transaction과 데이터 무결성 제약을 통해 시스템의 영속 데이터에 대한 일관성을 유지할 수 있다.

 현재 시스템에서는 별도의 Database Replica 구성을 도입하지 않는다.

 읽기 확장이나 장애 대응을 위해 Replica가 필요한지는 향후 실제 트래픽 및 운영 데이터를 측정한 후 별도의 Architecture 또는 Database ADR을 통해 결정한다.

 또한 Refresh Token과 같이 별도의 상태 관리가 필요한 데이터는 Database Architecture와 동일한 저장소로 강제하지 않으며, 해당 데이터의 저장 방식은 Redis 관련 ADR에서 별도로 결정한다.

---

 ## 4. Rationale

 ### 4.1 Relational Data Model

 현재 IAM의 핵심 데이터는 다음과 같은 관계를 가진다.

```
User
  │
  └── User ↔ Role
              │
              └── Role ↔ Permission
```

 이와 같이 여러 Entity와 관계 데이터를 명확하게 관리해야 하므로 관계형 데이터베이스가 적합하다고 판단했다.

 특히 관계 데이터에 대한 조회와 변경 과정에서 데이터 무결성을 유지하는 것이 중요하다.

 ### 4.2 Data Integrity

 User, Role, Permission과 이들 간 관계 데이터는 시스템의 권한 판정에 직접적인 영향을 줄 수 있다.

 따라서 데이터 저장 과정에서 Transaction과 무결성 제약을 활용할 수 있는 관계형 데이터베이스를 사용하는 것이 적절하다고 판단했다.

 ### 4.3 Operational Simplicity

 현재 시스템은 하나의 Spring Boot Backend Application을 중심으로 동작한다.

 이러한 구조에서 Database까지 Primary / Replica 또는 다중 Database 구조로 확장하면 개발 및 테스트 환경과 운영 환경의 복잡도가 증가한다.

 현재는 단일 MySQL 구조를 유지하여 시스템 전체 구성을 단순하게 관리한다.

 ### 4.4 Separation of State Management

 모든 데이터를 하나의 저장소에 집중시키는 것이 아니라 데이터 특성에 따라 저장소의 역할을 구분한다.

```
MySQL
 └── Persistent Domain Data

Redis
 └── Short-lived / Security-related State
```

 Redis의 구체적인 사용 목적과 장애 처리 정책은 `ADR-0003`에서 별도로 정의한다.

---

 ## 5. Consequences

 ### Positive

 #### 데이터 정합성 관리

 관계형 데이터 모델과 Transaction을 이용하여 User, Role, Permission 관계의 일관성을 관리하기 쉽다.

 #### 관계형 조회 용이

 IAM 데이터의 관계를 Join 등을 통해 명확하게 조회할 수 있다.

 #### 운영 단순성

 현재 단계에서는 하나의 Primary Database만 관리하므로 Database 운영 복잡도가 낮다.

 #### 테스트 용이성

 Docker Compose 기반의 동일한 Database 환경을 구성하여 Integration Test 및 데이터 정합성 검증을 반복적으로 수행할 수 있다.

 #### 확장 판단 기준 확보

 향후 실제 트래픽과 Database 성능을 측정한 뒤 Replica 또는 다른 확장 구조의 필요성을 판단할 수 있다.

---

 ### Negative

 #### Single Point of Failure

 단일 MySQL 인스턴스에 문제가 발생하면 영속 데이터에 접근하는 기능 전체에 영향을 줄 수 있다.

 #### 읽기 확장 한계

 읽기 트래픽이 크게 증가하면 단일 Database 구조가 병목이 될 수 있다.

 #### 장애 대응 한계

 현재 구조에서는 Database 수준의 고가용성 및 자동 Failover를 별도로 제공하지 않는다.

 #### 향후 확장 비용

 실제 운영 규모가 증가하여 Replica 또는 고가용성 Database가 필요해질 경우 추가적인 Architecture 변경이 필요하다.

---

 ## 6. Verification

 본 ADR의 결정은 다음 항목을 실제 실행을 통해 검증한다.

 ### 6.1 Data Integrity

 - User, Role, Permission 관계 데이터의 생성·수정·삭제 과정에서 데이터 무결성이 유지되는지 확인한다.
- 잘못된 관계 데이터가 저장되지 않는지 확인한다.
- Transaction 실패 시 데이터가 의도하지 않은 상태로 남지 않는지 확인한다.

 ### 6.2 Query Performance

 IAM 관련 주요 조회에 대해 다음 지표를 측정한다.

 - Query Latency
- API Latency
- p50
- p95
- p99
- Error Rate

 특히 User → Role → Permission 관계를 조회하는 주요 API의 Database Query 성능을 확인한다.

 ### 6.3 Concurrent Transaction

 동시 요청 환경에서 다음 항목을 검증한다.

 - Transaction Isolation
- Concurrent Update
- Rollback
- 데이터 정합성

 ### 6.4 Persistence

 Docker Compose 환경에서 Database Container를 재시작한 이후 영속 데이터가 정상적으로 유지되는지 확인한다.

 ### 6.5 Failure

 Database 연결 장애 또는 Database Container 장애 상황에서 애플리케이션이 해당 장애를 적절하게 처리하는지 확인한다.

 장애 상황에서 데이터가 손상되거나 잘못된 권한 상태가 생성되지 않는지도 검증한다.

---

 ## 7. Conditions for Reconsideration

 다음과 같은 상황이 실제 측정 또는 운영 결과로 확인되면 본 결정을 재검토한다.

 ### 7.1 Read Traffic Increase

 Database Read Traffic이 증가하여 단일 Database가 명확한 병목으로 확인되는 경우.

 ### 7.2 Availability Requirement

 Database 장애가 허용되지 않는 수준의 Availability 요구사항이 발생하는 경우.

 ### 7.3 Storage Growth

 데이터 규모가 증가하여 현재 Database 구성만으로 관리하기 어려운 상황이 발생하는 경우.

 ### 7.4 Measured Performance Bottleneck

 실제 Performance Test 또는 운영 Metrics를 통해 Database가 시스템의 주요 병목으로 확인되는 경우.

 이러한 상황이 발생하면 기존 ADR의 내용을 임의로 수정하지 않고 새로운 ADR을 생성하여 변경 이유와 영향을 기록한다.

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

 `ADR-0001`은 전체 Application Architecture를 결정하고, 본 ADR은 그중 **영속 데이터 저장을 담당하는 Database Architecture**를 결정한다.

 Redis의 구체적인 역할은 `ADR-0003`에서 별도로 관리한다.

 JWT 및 Refresh Token과 관련된 보안 정책은 `ADR-0004`, `ADR-0005`에서 각각 관리한다.

 RBAC의 권한 모델 및 판정 정책은 `ADR-0006`에서 관리한다.

---

 ## 9. Related Documents

 ### Architecture

 - `docs/02-architecture/architecture-overview.md`
- `docs/02-architecture/container.md`
- `docs/02-architecture/component.md`
- `docs/02-architecture/data-flow.md`
- `docs/02-architecture/failure-topology.md`

 ### Requirements

 - `docs/01-prd/functional/PRD-FUNC-002-user.md`
- `docs/01-prd/functional/PRD-FUNC-003-iam.md`

 ### Implementation

 - `26-05adf`

 ### Evidence

 - `PR-1A1`

---

 ## 10. Decision Summary

 > APMS.SR은 현재 시스템의 영속 데이터 저장을 위해 **Single MySQL Relational Database**를 채택한다.
>
>  User, Role, Permission 및 관련 관계 데이터를 관계형 모델로 관리하고 Transaction과 데이터 무결성 제약을 활용하여 데이터의 일관성을 유지한다.
>
>  현재 단계에서는 Database Replica 또는 별도의 분산 Database 구성을 도입하지 않는다.
>
>  Refresh Token과 같은 별도의 상태 데이터는 저장 특성에 따라 Redis에서 관리하며, Redis의 구체적인 사용 정책은 별도의 ADR에서 결정한다.
>
>  향후 실제 성능, 트래픽, 장애 및 Availability 요구사항이 현재 Database 구조의 한계를 입증하는 경우 새로운 ADR을 통해 Database Architecture를 재검토한다.

