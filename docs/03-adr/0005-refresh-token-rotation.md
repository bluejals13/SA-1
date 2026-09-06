

# ADR-0005: Refresh Token Rotation

**Status:** Accepted  
**Date:** 2026-09-06  
**Decision Type:** Architecture  
**Related PRD:** PRD-SEC-002-refresh-token  
**Related TASK:** TBD  
**Related Repository:** `26-05adf`  

---

## 1. Context & Drivers

APMS.SR 시스템은 `ADR-0004`에 따라 Access Token으로 짧은 수명(Short-lived)의 JWT를 사용한다. 
Access Token이 만료될 때마다 사용자가 다시 로그인하는 불편함을 방지하기 위해, 만료 기간이 긴 Refresh Token을 사용하여 새로운 Access Token을 발급받는 인증 갱신 메커니즘이 필요하다.

그러나 Refresh Token이 탈취될 경우, 해커는 해당 토큰이 만료될 때까지 지속적으로 새로운 Access Token을 발급받아 시스템에 접근할 수 있는 보안 위협(Replay Attack)이 존재한다.

따라서 사용자 편의성을 유지하면서도 토큰 탈취 및 재사용 공격을 탐지하고 방어할 수 있는 안전한 Refresh Token 아키텍처 결정이 필요하다.

본 결정의 주요 판단 기준은 다음과 같다.

1. Refresh Token 탈취 시 피해 최소화 및 탈취 탐지(Replay Detection) 기능
2. 다중 디바이스 또는 동시 요청 환경에서의 정합성
3. `ADR-0003`에서 결정된 Redis 기반 아키텍처와의 통합 효율성
4. 시스템 운영 및 구현의 복잡도 최소화

---

## 2. Options Considered

### Option A — Static Refresh Token

최초 로그인 시 발급된 Refresh Token을 만료 기간 전까지 계속해서 재사용하여 Access Token을 갱신한다.

**Advantages**
* 구현이 매우 단순하다.
* 네트워크 지연이나 동시성 문제(Concurrency)로 인한 갱신 실패 이슈가 발생하지 않는다.

**Disadvantages**
* Refresh Token이 탈취될 경우, 서버에서 사용자가 직접 로그아웃하거나 토큰이 만료되기 전까지 공격자가 무단으로 시스템을 사용할 수 있다 (Replay 방어 불가).

---

### Option B — Refresh Token Rotation (Strict)

Access Token을 갱신할 때마다 기존 Refresh Token을 폐기(Invalidate)하고 **새로운 Refresh Token을 1회용(One-time use)으로 발급**한다.

**Advantages**
* 토큰 탈취를 효과적으로 탐지할 수 있다. 만약 이미 사용된(폐기된) Refresh Token으로 다시 갱신 요청이 들어오면, 토큰이 탈취된 것으로 간주하고 해당 사용자의 모든 인증 세션(Token Family)을 즉시 삭제할 수 있다.
* 보안성이 매우 뛰어나다.

**Disadvantages**
* 클라이언트(Frontend)에서 다중 API 요청 중 동시에 Refresh 요청이 발생할 경우(Race Condition), 하나를 제외한 나머지 요청은 폐기된 토큰을 사용한 것으로 간주되어 정상적인 사용자가 로그아웃 처리될 수 있다.
* 프론트엔드에서 갱신 요청을 동기화(Mutex/Lock)하는 추가 구현이 필요하다.

---

### Option C — Refresh Token Rotation with Grace Period

Option B(RTR)를 사용하되, 기존 Refresh Token을 즉시 폐기하지 않고 아주 짧은 유예 기간(Grace Period, 예: 10초~30초) 동안 유효한 상태로 유지한다.

**Advantages**
* Strict RTR의 보안성을 유지하면서도, 네트워크 지연이나 프론트엔드의 동시 다발적인 Refresh 요청으로 인한 동시성 문제를 서버 단에서 우아하게 해결할 수 있다.

**Disadvantages**
* 유예 기간(Grace Period) 내에 발생하는 Replay Attack은 방어할 수 없는 보안 취약 시간(Window)이 존재한다.
* Redis 상태 관리 로직(Family 관리 및 지연 삭제 등)이 Option B에 비해 복잡해진다.

---

## 3. Decision

현재 시스템의 Refresh Token 관리 메커니즘으로 Option B — Refresh Token Rotation (Strict) 를 채택한다.

보안 요구사항(PRD-SEC-002-refresh-token)을 가장 안전하게 충족하기 위해, 
한 번 사용된 Refresh Token은 즉시 Invalidated 상태로 전환하고,
새로운 Refresh Token을 발급하는 Strict RTR 방식을 적용한다.

기존 Refresh Token은 단순 삭제하지 않는다. 사용 완료된 Refresh Token의 JTI와 invalidatedAt을 Redis에 기록하고,
Replay Detection을 위해 해당 상태를 설정된 TTL 동안 보존한다.

이후 동일한 JTI가 다시 제출되면 이미 사용된 Refresh Token으로 판단하여 Replay Attack으로 처리한다.

정상적인 Rotation에서는 기존 Refresh Token을 Invalidated 상태로 전환하고 invalidatedAt을 기록한 후,
새로운 Refresh Token을 현재 유효한 토큰으로 등록한다. 
이전 JTI의 Invalidated 상태는 Replay Detection을 위해 별도로 TTL 동안 보존한다.

동일 Token Family에서 Replay가 탐지된 경우
해당 Family의 인증 상태를 Revoked 처리하여 
이후 동일 Family의 Refresh Token을 이용한 갱신을 거부한다.

---

## 4. Rationale

### 4.1 Security First (Replay Detection)
정적 Refresh Token(Option A)은 구현이 단순하지만 보안상 취약하다. Strict RTR(Option B)은 해커가 토큰을 탈취하더라도, 정상 사용자가 토큰을 갱신하는 순간 해커의 토큰이 무효화되고, 반대로 해커가 먼저 갱신하더라도 정상 사용자의 갱신 시도 시 탈취가 탐지되어 전체 세션이 차단되므로 보안성이 매우 높다.

### 4.2 Responsibility Separation
서버(Backend)는 인증 데이터의 무결성과 보안 정책(1회용 토큰)을 강제하는 것에 집중한다. 여러 API 요청이 동시에 토큰 갱신을 유발하는 동시성 문제는 클라이언트(Frontend)의 상태 관리 영역에 해당하므로, Frontend에서 갱신 요청을 직렬화(Serialize)하여 책임을 분리하는 것이 구조적으로 깔끔하다.

### 4.3 Redis Synergy
`ADR-0003`에서 단일 Redis 인스턴스를 도입하기로 결정했다. Redis는 In-Memory 기반으로 트랜잭션과 빠른 상태 변경을 지원하므로, 토큰 폐기 및 발급, Family 전체 무효화와 같은 빈번한 I/O 작업을 지연 없이 처리하기에 최적화되어 있다.

---

## 5. Consequences

### Positive

#### 토큰 탈취 방어력 극대화
Refresh Token이 유출되더라도, Replay Detection을 통해 즉각적인 세션 파기 및 피해 최소화가 가능하다.

#### 서버 로직 단순화
현재 유효한 Token 확인 → 기존 Token을 Invalidated 상태로 전환하고 invalidatedAt 기록 → 새로운 Token 등록의 
전체 과정은 단일 원자적 작업으로 처리한다. 
이를 통해 동일한 Refresh Token에 대한 동시 요청 중 하나만 Rotation에 성공하도록 보장한다.

> Redis Transaction 또는 Lua Script 등 Redis의 원자적 처리 방식을 사용하여 구현한다.
---

### Negative

#### 프론트엔드 복잡도 증가
클라이언트 단에서 401 Unauthorized 에러를 인터셉트하고, 중복된 Refresh Token 요청이 발생하지 않도록 Queue를 구성하거나 Promise를 활용해 토큰 갱신을 동기화해야 하는 구현 부담이 존재한다.

#### 잦은 Redis 쓰기(Write) 작업
Access Token이 만료될 때마다 새로운 Refresh Token을 발급하고 저장해야 하므로, Option A(Static)에 비해 Redis에 대한 쓰기 연산(I/O) 비용이 증가한다 (단, Redis 성능상 큰 병목은 아님).

---

## 6. Verification

본 ADR의 결정은 다음 항목을 실제 실행을 통해 검증한다.

### 6.1 Normal Rotation
* Refresh Token을 이용해 정상적으로 Access Token과 새로운 Refresh Token이 발급되는지 확인한다.
* 이전 Refresh Token의 JTI가 사용 완료 상태로 기록되는지 확인한다.
  invalidatedAt이 정상적으로 기록되는지 확인한다.
* 사용 완료된 Refresh Token이 다시 제출될 경우 Replay Attack으로 탐지되는지 확인한다.
  Invalidated Token State가 설정된 TTL 이후 자동으로 제거되는지 확인한다.

### 6.2 Replay Detection
* 이미 Invalidated 상태인 Refresh Token으로 다시 갱신을 시도했을 때 Replay Attack으로 탐지되고 갱신이 거부되는지 확인한다.
* Replay Detection 발생 시 해당 Token Family가 Revoked 상태로 전환되는지 확인한다.
* Family가 Revoked 상태가 된 이후 동일 Family의 최신 Refresh Token을 이용한 갱신도 거부되는지 확인한다.
* Invalidated 상태 및 invalidatedAt 정보가 설정된 TTL 이후 자동으로 제거되는지 확인한다.

> 해당 시점 이후 최신 Refresh Token을 포함한 동일 Token Family의 인증 상태가 더 이상 유효하지 않도록 처리되는지 확인한다.

### 6.3 Concurrent Refresh
* 동일한 Refresh Token을 사용하여 동시에 여러 개의 Refresh 요청을 발생시킨다.
* 하나의 요청만 정상적으로 Rotation되고 나머지 요청은 재사용된 Token으로 판단되어 거부되는지 확인한다.
* 동시 요청 이후 Redis에 유효한 Refresh Token 상태가 중복 생성되지 않는지 확인한다.


---

## 7. Conditions for Reconsideration

다음과 같은 상황이 실제 측정 또는 운영 결과로 확인되면 본 결정을 재검토한다.

### 7.1 Frontend Synchronization Failure
다양한 클라이언트 환경(웹, 모바일 앱 등)에서 프론트엔드 단의 토큰 갱신 동기화 처리가 지속적으로 실패하여, 정상적인 사용자가 잦은 강제 로그아웃을 경험하는 빈도가 높아지는 경우 (Option C의 Grace Period 도입 검토).

### 7.2 Redis Write Bottleneck
극단적인 트래픽 상황에서 모든 사용자의 잦은 Refresh Rotation 쓰기 연산이 Redis의 성능 한계(CPU/Memory)를 초과하여 병목으로 작용하는 경우.

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

본 `ADR-0005`는 `ADR-0004`(JWT)에서 채택한 Stateless Access Token의 단점을 보완하기 위한 보안 정책을 정의한다.
또한 이 과정에서 발생하는 모든 상태 변경 및 탈취 감지 로직은 `ADR-0003`에서 결정한 Redis 인프라 위에서 구현된다.

---

## 9. Related Documents

### Architecture
* `docs/02-architecture/authentication-flow.md`
* `docs/02-architecture/data-flow.md`
* `docs/02-architecture/failure-topology.md`

### Requirements
* `docs/01-prd/security/PRD-SEC-002-refresh-token.md`

### Implementation
* `26-05adf`

### Evidence
* `PR-1A1`

---

## 10. Decision Summary

> APMS.SR은 토큰 탈취 피해를 방어하고 보안성을 극대화하기 위해 **Strict Refresh Token Rotation(RTR)** 메커니즘을 채택한다.
> 
> 사용자가 Access Token을 갱신할 때마다 기존 Refresh Token은 사용 완료(Invalidated) 상태로 전환하며, 
  해당 JTI와 invalidatedAt을 기록하여 일정 기간 보존한다. 이 상태는 Replay Detection을 위해 설정된 TTL 동안 보존된다.
  사용 완료된 Refresh Token이 다시 사용되는 경우 Replay Attack으로 판단하고, 해당 Token Family를 무효화한다.

> 사용 완료된 Refresh Token이 다시 사용되는 경우는 Replay Attack으로 판단하고, 
   해당 Token Family를 Revoked 상태로 전환하여 추가적인 Token Rotation을 차단한다.
> Refresh Token의 상태 변경과 신규 Token 발급 과정은 원자적으로 처리하여 동일 Token에 대한 동시 Rotation을 방지한다.

> 동시 다발적인 API 요청으로 인한 Refresh 요청은 Frontend에서 토큰 갱신을 동기화하여 처리하며, 
  정상 사용자의 동시성 문제로 인해 서버의 Replay Detection 정책이 우회되지 않도록 한다.
> 
> 향후 정상 사용자의 강제 로그아웃 빈도가 허용치를 초과하거나 다양한 클라이언트 환경에서 
  Strict RTR의 동시성 문제가 지속적으로 발생할 경우, 새로운 ADR을 통해 Grace Period 방식 도입을 재검토한다.

