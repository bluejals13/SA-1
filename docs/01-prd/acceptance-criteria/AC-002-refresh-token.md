# AC-002 — Refresh Token

## 1. 개요

### 1.1 목적

본 문서는 Refresh Token 기능에 대한 Acceptance Criteria를 정의한다.

PRD-FUNC-001-authentication의 인증 상태 갱신 요구사항과 PRD-SEC-002-refresh-token의 Refresh Token 보안 요구사항이 실제 시스템에서 충족되는지를 검증한다.

### 1.2 검증 대상

본 Acceptance Criteria는 다음을 대상으로 한다.

- 정상적인 Refresh Token 갱신
- Refresh Token 검증
- Access Token 재발급
- Refresh Token Rotation
- 이전 Refresh Token 재사용 방지
- 만료된 Refresh Token 거부
- 무효화된 Refresh Token 거부
- 잘못된 Refresh Token 거부
- 동시 Refresh 요청 처리

## 2. Traceability

| Acceptance Criteria | PRD Requirement |
|---|---|
| AC-REFRESH-001 | SEC-REFRESH-001, SEC-REFRESH-003 |
| AC-REFRESH-002 | SEC-REFRESH-005, SEC-REFRESH-006 |
| AC-REFRESH-003 | SEC-REFRESH-007 |
| AC-REFRESH-004 | SEC-REFRESH-008 |
| AC-REFRESH-005 | SEC-REFRESH-010 |
| AC-REFRESH-006 | SEC-REFRESH-011 |
| AC-REFRESH-007 | SEC-REFRESH-003, SEC-REFRESH-006 |
| AC-REFRESH-008 | SEC-REFRESH-009 |
| AC-REFRESH-009 | SEC-REFRESH-009 |
| AC-REFRESH-010 | SEC-REFRESH-012 |

## 3. Acceptance Criteria

### AC-REFRESH-001 — 정상 Refresh Token 갱신

**Given**

- 정상적으로 인증된 사용자가 존재한다.
- 유효한 Refresh Token이 존재한다.
- Refresh Token이 현재 갱신 가능한 상태다.

**When**

사용자가 Refresh Token을 이용하여 인증 상태 갱신을 요청한다.

**Then**

- 시스템은 Refresh Token의 유효성을 검증해야 한다.
- 해당 Refresh Token과 연결된 사용자를 식별해야 한다.
- 새로운 Access Token을 발급해야 한다.
- 갱신 요청은 정상적으로 처리되어야 한다.

**Pass Condition**

유효한 Refresh Token으로 새로운 Access Token을 정상적으로 획득할 수 있다.

### AC-REFRESH-002 — 사용자 연계 검증

**Given**

Refresh Token이 특정 사용자와 연결되어 있다.

**When**

해당 Refresh Token을 이용하여 인증 상태 갱신을 요청한다.

**Then**

- 시스템은 Refresh Token에 연결된 사용자를 기준으로 인증 상태를 갱신해야 한다.
- 다른 사용자의 인증 상태로 Token을 발급해서는 안 된다.

**Pass Condition**

Refresh Token과 다른 사용자의 인증 상태가 생성되지 않는다.

### AC-REFRESH-003 — Refresh Token Rotation

**Given**

유효한 Refresh Token A가 존재한다.

**When**

Refresh Token A를 이용하여 갱신 요청을 성공적으로 수행한다.

**Then**

- 새로운 인증 상태를 획득할 수 있어야 한다.
- 새로운 Refresh Token B가 발급되어야 한다.
- 기존 Refresh Token A는 더 이상 정상적인 갱신에 사용할 수 없어야 한다.

**Pass Condition**

```txt
Refresh Token A
      ↓
   refresh
      ↓
Refresh Token B
      +
A → invalid
```