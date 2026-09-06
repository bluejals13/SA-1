# PRD-SEC-004 — Token Invalidation

## 1. 개요

### 1.1 목적

시스템은 로그아웃 또는 기타 인증 상태 종료가 발생한 경우 기존에 발급된 인증 Token이 더 이상 유효한 인증 수단으로 사용되지 않도록 무효화할 수 있어야 한다.

이를 통해 이미 발급된 Token이 인증 상태 종료 이후에도 계속 사용되는 것을 방지해야 한다.

본 문서는 인증 Token의 무효화가 무엇을 보장해야 하는지를 정의하며, 특정 저장소, 캐시 또는 구현 기술을 요구하지 않는다.

### 1.2 범위

본 요구사항은 다음 Token Invalidation 기능을 대상으로 한다.

- 로그아웃에 따른 인증 상태 종료
- Access Token 무효화
- Refresh Token 무효화
- 무효화된 Access Token의 재사용 방지
- 무효화된 Refresh Token의 재사용 방지
- 사용자별 인증 상태 독립성
- Token Invalidation 상태의 일관성

Refresh Token Rotation 및 Replay 방어는 PRD-SEC-002-refresh-token에서 정의한다.

JWT의 기본적인 서명 및 만료 검증은 PRD-SEC-001-jwt에서 정의한다.

Redis 등의 인프라 장애 상황에서의 처리 정책은 PRD-SEC-005-redis-failure에서 정의한다.

## 2. Security Requirements

### SEC-INVALID-001 — 로그아웃에 따른 인증 상태 종료

사용자가 로그아웃을 요청한 경우 시스템은 해당 인증 상태를 종료해야 한다.

로그아웃이 정상적으로 완료된 이후 기존 인증 상태를 계속 유효한 상태로 취급해서는 안 된다.

### SEC-INVALID-002 — Access Token 무효화

시스템은 로그아웃 또는 인증 상태 종료 시 해당 인증 상태와 연결된 Access Token을 이후 인증에 사용할 수 없도록 무효화할 수 있어야 한다.

### SEC-INVALID-003 — 무효화된 Access Token 거부

무효화된 Access Token이 이후 보호된 요청에 사용된 경우 시스템은 해당 요청을 인증된 요청으로 처리해서는 안 된다.

### SEC-INVALID-004 — Refresh Token 무효화

로그아웃 또는 인증 상태 종료 시 해당 인증 상태와 연결된 Refresh Token은 이후 인증 상태 갱신에 사용할 수 없도록 무효화되어야 한다.

### SEC-INVALID-005 — 무효화된 Refresh Token 재사용 방지

무효화된 Refresh Token이 다시 제출된 경우 시스템은 이를 이용한 새로운 Access Token 또는 인증 상태의 발급을 허용해서는 안 된다.

### SEC-INVALID-006 — Access Token과 Refresh Token의 독립적인 무효화

Access Token과 Refresh Token은 서로 다른 사용 목적을 가지므로 각각의 무효화 상태를 적절하게 관리할 수 있어야 한다.

Access Token이 무효화되었다는 사실만으로 Refresh Token이 자동으로 유효하다고 판단해서는 안 되며, 반대로 Refresh Token의 무효화 여부만으로 Access Token의 유효성을 판단해서도 안 된다.

### SEC-INVALID-007 — 인증 상태 단위 무효화

시스템은 무효화 대상이 되는 인증 상태를 식별할 수 있어야 하며, 특정 인증 상태의 종료가 다른 사용자의 인증 상태에 영향을 주어서는 안 된다.

### SEC-INVALID-008 — 무효화 상태의 지속성

Token이 무효화된 이후 해당 Token의 유효기간이 남아 있더라도 시스템은 정의된 무효화 정책에 따라 해당 Token을 계속 거부해야 한다.

Token의 원래 만료 시점까지 기다리는 것만으로 무효화 요구사항을 충족해서는 안 된다.

### SEC-INVALID-009 — 무효화와 인증 검증의 연계

시스템은 보호된 요청을 처리할 때 Token 자체의 유효성뿐만 아니라 해당 Token이 현재 무효화된 상태인지도 확인해야 한다.

유효한 서명을 가진 Token이라도 무효화된 상태라면 인증된 요청으로 처리해서는 안 된다.

### SEC-INVALID-010 — 무효화 상태의 사용자 간 독립성

특정 사용자의 Token을 무효화하거나 인증 상태를 종료하는 과정이 다른 사용자의 Token 또는 인증 상태를 의도하지 않게 무효화해서는 안 된다.

## 3. Invalidation Boundaries

본 문서에서는 다음 구현 방법을 요구하지 않는다.

- Redis
- Database
- Cache
- Blacklist
- 특정 JWT 라이브러리
- 특정 Framework
- 특정 Token 저장 구조
- 특정 삭제 방식

구체적인 구현 방법 및 인프라 구성은 Architecture 및 ADR에서 정의한다.

## 4. Functional Traceability

| Requirement | Related Functional PRD |
|---|---|
| SEC-INVALID-001 | PRD-FUNC-001-authentication / FR-AUTH-006 |
| SEC-INVALID-002 | PRD-FUNC-001-authentication / FR-AUTH-006 |
| SEC-INVALID-003 | PRD-FUNC-001-authentication / FR-AUTH-004 |
| SEC-INVALID-004 | PRD-FUNC-001-authentication / FR-AUTH-006 |
| SEC-INVALID-005 | PRD-FUNC-001-authentication / FR-AUTH-005 |
| SEC-INVALID-007 | PRD-FUNC-001-authentication / FR-AUTH-007 |
| SEC-INVALID-009 | PRD-FUNC-001-authentication / FR-AUTH-004 |

## 5. Related Security PRD

| Related PRD | Relationship |
|---|---|
| PRD-SEC-001-jwt | Token의 기본 유효성 및 인증 검증 |
| PRD-SEC-002-refresh-token | Refresh Token Rotation 및 Replay 방어 |
| PRD-SEC-003-rbac | 무효화되지 않은 인증 상태의 권한 적용 |
| PRD-SEC-005-redis-failure | Token 상태 관리 인프라 장애 처리 |

## 6. Implementation Evidence

본 PRD는 특정 구현을 요구하지 않는다.

현재 구현과 관련된 주요 구성요소는 다음과 같다.

- AuthController
- AuthService
- TokenBlacklistService
- RefreshTokenRepository
- JwtAuthenticationFilter

현재 구현과의 추적은 다음 단계에서 연결한다.

```txt
PRD-SEC-004-token-invalidation
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
