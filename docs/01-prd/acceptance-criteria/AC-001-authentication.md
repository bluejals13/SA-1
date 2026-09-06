
# AC-001 — Authentication

## 1. 개요

### 1.1 목적

본 문서는 인증 기능에 대한 Acceptance Criteria를 정의한다.

PRD-FUNC-001-authentication에서 정의한 인증 기능 요구사항과 PRD-SEC-001-jwt에서 정의한 JWT 기반 인증 보안 요구사항이 실제 시스템에서 충족되는지를 검증하는 것을 목적으로 한다.

### 1.2 검증 대상

본 Acceptance Criteria는 다음 기능을 대상으로 한다.

- 사용자 인증 성공
- 사용자 인증 실패
- Access Token 발급
- 인증이 필요한 요청의 인증
- 유효하지 않은 Token 거부
- 만료된 Token 거부
- 인증되지 않은 요청 거부
- 인증과 인가의 분리

Refresh Token의 Rotation 및 Replay 방어는 AC-002-refresh-token에서 검증한다.

RBAC에 따른 권한 검증은 AC-003-rbac에서 검증한다.

## 2. Traceability

| Acceptance Criteria | PRD Requirement |
|---|---|
| AC-AUTH-001 | FR-AUTH-001, FR-AUTH-002 |
| AC-AUTH-002 | FR-AUTH-003 |
| AC-AUTH-003 | FR-AUTH-004 |
| AC-AUTH-004 | FR-AUTH-004, SEC-JWT-* |
| AC-AUTH-005 | FR-AUTH-004, PRD-SEC-001-jwt |
| AC-AUTH-006 | FR-AUTH-004, PRD-SEC-001-jwt |
| AC-AUTH-007 | FR-AUTH-004, PRD-SEC-001-jwt |
| AC-AUTH-008 | FR-AUTH-004, PRD-SEC-003-rbac |

## 3. Acceptance Criteria

### AC-AUTH-001 — 유효한 인증 정보로 인증 성공

**Given**

- 시스템에 유효한 사용자 계정이 존재한다.
- 사용자가 올바른 인증 정보를 제출한다.

**When**

사용자가 인증 요청을 수행한다.

**Then**

- 시스템은 인증 요청을 성공적으로 처리해야 한다.
- 해당 사용자를 식별할 수 있어야 한다.
- 이후 인증이 필요한 요청을 수행할 수 있는 인증 상태를 획득해야 한다.
- Access Token이 발급되는 경우 해당 Token은 시스템의 JWT 검증 정책을 만족해야 한다.

**Pass Condition**

유효한 인증 정보로 인증 요청을 수행했을 때 인증 성공 결과와 정상적인 인증 상태를 획득한다.

### AC-AUTH-002 — 잘못된 인증 정보 거부

**Given**

- 시스템에 사용자 계정이 존재한다.
- 사용자가 잘못된 인증 정보를 제출한다.

**When**

인증 요청을 수행한다.

**Then**

- 시스템은 인증을 성공 처리해서는 안 된다.
- 인증이 필요한 기능에 접근할 수 있는 인증 상태를 발급해서는 안 된다.
- 클라이언트가 인증 실패를 인지할 수 있는 일관된 결과를 반환해야 한다.

**Pass Condition**

잘못된 인증 정보로 인증을 시도했을 때 인증 성공 상태가 생성되지 않는다.

### AC-AUTH-003 — 인증되지 않은 요청 거부

**Given**

- 인증이 필요한 보호된 기능이 존재한다.
- 요청에 유효한 인증 정보가 없다.

**When**

인증 없이 보호된 기능을 요청한다.

**Then**

- 시스템은 요청을 정상적인 인증 요청으로 처리해서는 안 된다.
- 보호된 기능의 결과를 반환해서는 안 된다.
- 시스템이 정의한 인증 실패 결과를 반환해야 한다.

**Pass Condition**

인증 정보 없이 보호된 리소스에 접근할 수 없다.

### AC-AUTH-004 — 유효한 Access Token을 이용한 인증

**Given**

- 유효한 Access Token이 존재한다.
- 해당 Token이 시스템의 JWT 검증 정책을 만족한다.

**When**

Access Token을 사용하여 인증이 필요한 요청을 수행한다.

**Then**

- 시스템은 Token을 검증해야 한다.
- Token에 연결된 사용자를 식별해야 한다.
- 해당 요청을 인증된 요청으로 처리할 수 있어야 한다.

**Pass Condition**

유효한 Access Token을 가진 사용자가 인증이 필요한 기능에 정상적으로 접근할 수 있다.

### AC-AUTH-005 — 위조 또는 변조된 Token 거부

**Given**

정상적으로 발급된 Access Token이 존재한다.

**When**

Token의 서명 검증에 실패하도록 Token을 변조하여 보호된 기능을 요청한다.

**Then**

- 시스템은 해당 Token을 유효한 인증 수단으로 처리해서는 안 된다.
- 보호된 기능에 접근할 수 없어야 한다.

**Pass Condition**

변조된 JWT를 이용한 보호된 요청이 거부된다.

### AC-AUTH-006 — 만료된 Access Token 거부

**Given**

Access Token의 유효기간이 만료된 상태다.

**When**

만료된 Token으로 보호된 기능을 요청한다.

**Then**

- 시스템은 해당 Token을 유효한 인증 수단으로 처리해서는 안 된다.
- 보호된 기능에 접근할 수 없어야 한다.

**Pass Condition**

만료된 Access Token을 이용한 요청이 인증 단계에서 거부된다.

### AC-AUTH-007 — 잘못된 Token 형식 거부

**Given**

인증이 필요한 보호된 기능이 존재한다.

**When**

다음과 같은 잘못된 인증 정보를 이용하여 요청한다.

- 빈 Token
- 잘못된 JWT 형식
- 지원하지 않는 Token 형식
- 검증할 수 없는 Token

**Then**

- 시스템은 해당 요청을 인증된 요청으로 처리해서는 안 된다.
- 보호된 기능에 접근할 수 없어야 한다.

**Pass Condition**

검증할 수 없는 인증 Token으로 보호된 기능에 접근할 수 없다.

### AC-AUTH-008 — 인증과 인가의 분리

**Given**

- 사용자가 정상적으로 인증되어 있다.
- 해당 사용자가 특정 보호된 기능에 필요한 권한을 가지고 있지 않다.

**When**

사용자가 해당 기능에 접근한다.

**Then**

- 시스템은 사용자의 인증 상태와 권한 보유 여부를 별도로 판단해야 한다.
- 인증되었다는 이유만으로 해당 기능에 접근할 수 있어서는 안 된다.
- 권한이 없는 요청은 인가 실패로 처리되어야 한다.

**Pass Condition**

정상적으로 인증된 사용자라도 필요한 권한이 없으면 해당 보호된 기능에 접근할 수 없다.

## 4. Verification Matrix

| ID | Scenario | Expected Result | Related PRD |
|---|---|---|---|
| AC-AUTH-001 | 정상 로그인 | 인증 성공 | FR-AUTH-001, FR-AUTH-002 |
| AC-AUTH-002 | 잘못된 인증 정보 | 인증 실패 | FR-AUTH-003 |
| AC-AUTH-003 | Token 없는 보호 요청 | 접근 거부 | FR-AUTH-004 |
| AC-AUTH-004 | 정상 Access Token | 접근 허용 | FR-AUTH-004 |
| AC-AUTH-005 | 변조 JWT | 접근 거부 | PRD-SEC-001-jwt |
| AC-AUTH-006 | 만료 JWT | 접근 거부 | PRD-SEC-001-jwt |
| AC-AUTH-007 | 잘못된 Token | 접근 거부 | PRD-SEC-001-jwt |
| AC-AUTH-008 | 인증은 성공했지만 권한 없음 | 인가 거부 | PRD-SEC-003-rbac |

## 5. Implementation Verification

현재 구현과의 검증 대상은 다음과 같다.

```txt
AC-001-authentication
        ↓
AuthController
        ↓
AuthService
        ↓
JwtProvider
        ↓
JwtAuthenticationFilter
        ↓
SecurityConfig
        ↓
Test
        ↓
Execution
        ↓
Evidence
```