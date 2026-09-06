# AC-003 — RBAC

## 1. 개요

### 1.1 목적

본 문서는 Role-Based Access Control(RBAC) 기능에 대한 Acceptance Criteria를 정의한다.

PRD-FUNC-003-iam 및 PRD-SEC-003-rbac에서 정의한 사용자 Role 및 Permission 기반 접근 제어 요구사항이 실제 시스템에서 충족되는지를 검증하는 것을 목적으로 한다.

### 1.2 검증 대상

본 Acceptance Criteria는 다음을 대상으로 한다.

- 사용자와 Role의 연결
- Role과 Permission의 연결
- 인증된 사용자의 권한 구성
- Permission 기반 접근 제어
- 권한이 없는 요청의 거부
- Role 변경에 따른 권한 변경
- Permission 변경에 따른 권한 변경
- IAM 관리 기능의 접근 제어
- 사용자 간 권한 독립성
- 최소 권한 적용

## 2. Traceability

| Acceptance Criteria | PRD Requirement |
|---|---|
| AC-RBAC-001 | SEC-RBAC-001 |
| AC-RBAC-002 | SEC-RBAC-002 |
| AC-RBAC-003 | SEC-RBAC-003 |
| AC-RBAC-004 | SEC-RBAC-004 |
| AC-RBAC-005 | SEC-RBAC-005 |
| AC-RBAC-006 | SEC-RBAC-006 |
| AC-RBAC-007 | SEC-RBAC-007 |
| AC-RBAC-008 | SEC-RBAC-008 |
| AC-RBAC-009 | SEC-RBAC-009 |
| AC-RBAC-010 | SEC-RBAC-010 |
| AC-RBAC-011 | SEC-RBAC-011 |

## 3. Acceptance Criteria

### AC-RBAC-001 — 사용자 Role 연결

**Given**

- 정상적인 사용자가 존재한다.
- 시스템에 연결 가능한 Role이 존재한다.

**When**

사용자에게 Role을 연결한다.

**Then**

- 해당 사용자는 연결된 Role을 기준으로 권한을 구성할 수 있어야 한다.
- 다른 사용자에게 동일한 Role이 의도하지 않게 연결되어서는 안 된다.

**Pass Condition**

사용자와 Role의 연결 관계가 정상적으로 구성되고 해당 사용자에게 적용된다.

### AC-RBAC-002 — Role Permission 연결

**Given**

- 시스템에 Role이 존재한다.
- 시스템에 Permission이 존재한다.

**When**

Permission을 Role에 연결한다.

**Then**

- 해당 Role은 연결된 Permission을 가지게 된다.
- 해당 Role을 가진 사용자는 해당 Permission을 기준으로 접근 권한을 구성할 수 있어야 한다.

**Pass Condition**

Role에 연결된 Permission이 정상적으로 해당 Role의 권한으로 적용된다.

### AC-RBAC-003 — 권한 있는 사용자의 접근 허용

**Given**

- 사용자가 인증되어 있다.
- 사용자에게 보호된 기능에 필요한 Permission이 존재한다.

**When**

사용자가 해당 보호 기능을 요청한다.

**Then**

- 시스템은 사용자의 인증 상태를 확인해야 한다.
- 사용자의 권한을 확인해야 한다.
- 필요한 Permission이 존재하는 경우 요청을 정상적으로 처리할 수 있어야 한다.

**Pass Condition**

필요한 Permission을 가진 인증 사용자가 보호 기능에 접근할 수 있다.

### AC-RBAC-004 — 권한 없는 사용자의 접근 거부

**Given**

- 사용자가 정상적으로 인증되어 있다.
- 사용자에게 보호된 기능에 필요한 Permission이 없다.

**When**

사용자가 해당 기능을 요청한다.

**Then**

- 시스템은 사용자가 인증되었다는 이유만으로 요청을 허용해서는 안 된다.
- 요청은 인가 실패로 처리되어야 한다.
- 보호된 기능의 정상적인 결과를 반환해서는 안 된다.

**Pass Condition**

인증은 성공했지만 필요한 Permission이 없는 사용자는 해당 기능에 접근할 수 없다.

### AC-RBAC-005 — Role 변경 반영

**Given**

- 사용자에게 Role A가 연결되어 있다.
- Role A를 통해 Permission A를 가지고 있다.
- Permission B는 가지고 있지 않다.

**When**

사용자의 Role을 Role B로 변경한다.

**Then**

- 이후 접근 제어에서는 변경된 Role을 기준으로 권한을 판단해야 한다.
- Role 변경 이전의 권한을 무조건 유지해서는 안 된다.

**Pass Condition**

Role 변경 이후 사용자의 접근 권한이 새로운 Role의 권한 정책에 맞게 변경된다.

### AC-RBAC-006 — Permission 변경 반영

**Given**

- Role A에 Permission A가 연결되어 있다.
- 해당 Role을 가진 사용자가 존재한다.

**When**

Role A의 Permission 구성을 변경한다.

**Then**

- 이후 접근 제어에서 변경된 Permission 상태를 반영할 수 있어야 한다.
- 제거된 Permission을 계속 유효한 권한으로 처리해서는 안 된다.

**Pass Condition**

Permission 변경 이후 접근 결과가 변경된 Permission 정책과 일치한다.

### AC-RBAC-007 — 인증과 인가 분리

**Given**

- 사용자가 정상적으로 인증되어 있다.
- 특정 보호 기능에 필요한 Permission을 가지고 있지 않다.

**When**

해당 보호 기능에 접근한다.

**Then**

- 인증 성공 여부와 인가 가능 여부를 별도로 판단해야 한다.
- 인증 성공만으로 접근이 허용되어서는 안 된다.

**Pass Condition**

인증된 사용자라도 권한이 없으면 보호 기능에 접근할 수 없다.

### AC-RBAC-008 — 권한 없는 요청의 일관된 거부

**Given**

- 보호된 기능에 특정 Permission이 요구된다.
- 요청 사용자가 해당 Permission을 보유하지 않는다.

**When**

사용자가 해당 기능에 접근한다.

**Then**

- 시스템은 요청을 정상적인 성공 요청으로 처리해서는 안 된다.
- 권한 부족을 나타내는 일관된 결과를 반환해야 한다.

**Pass Condition**

권한 없는 요청이 정의된 인가 실패 처리로 일관되게 거부된다.

### AC-RBAC-009 — IAM 관리 기능 보호

**Given**

- 사용자 관리, Role 관리 또는 Permission 관리 기능이 존재한다.
- 해당 관리 기능에 접근하기 위한 Permission이 정의되어 있다.

**When**

권한이 없는 사용자가 IAM 관리 기능에 접근한다.

**Then**

- 해당 요청은 거부되어야 한다.
- IAM 관리 기능의 변경 작업을 수행할 수 없어야 한다.

**Pass Condition**

필요한 관리 권한이 없는 사용자는 IAM 관리 기능을 사용할 수 없다.

### AC-RBAC-010 — 사용자 간 권한 독립성

**Given**

- 사용자 A와 사용자 B가 존재한다.
- 두 사용자의 Role 또는 Permission 구성이 서로 다르다.

**When**

사용자 A의 Role 또는 Permission을 변경한다.

**Then**

- 사용자 B의 권한은 의도하지 않게 변경되어서는 안 된다.
- 사용자 B는 자신의 권한 정책에 따라 계속 접근할 수 있어야 한다.

**Pass Condition**

한 사용자의 권한 변경이 다른 사용자의 권한에 영향을 주지 않는다.

### AC-RBAC-011 — 최소 권한 적용

**Given**

- 사용자에게 특정 기능에 필요한 Permission만 부여되어 있다.
- 추가적인 보호 기능에 필요한 Permission은 부여되어 있지 않다.

**When**

사용자가 시스템 기능에 접근한다.

**Then**

- 허용된 기능은 사용할 수 있어야 한다.
- 부여되지 않은 추가 권한이 필요한 기능은 사용할 수 없어야 한다.

**Pass Condition**

사용자는 부여된 Permission의 범위를 벗어나 보호된 기능에 접근할 수 없다.

## 4. Verification Matrix

| ID | Scenario | Expected Result |
|---|---|---|
| AC-RBAC-001 | User → Role 연결 | Role 정상 적용 |
| AC-RBAC-002 | Role → Permission 연결 | Permission 정상 적용 |
| AC-RBAC-003 | 권한 보유 사용자 | 접근 허용 |
| AC-RBAC-004 | 권한 없는 사용자 | 접근 거부 |
| AC-RBAC-005 | Role 변경 | 변경된 권한 적용 |
| AC-RBAC-006 | Permission 변경 | 변경된 권한 적용 |
| AC-RBAC-007 | 인증/인가 분리 | 인증만으로 접근 불가 |
| AC-RBAC-008 | 권한 없는 요청 | 일관된 거부 |
| AC-RBAC-009 | IAM 관리 접근 | 권한 없는 관리자 기능 접근 거부 |
| AC-RBAC-010 | 사용자 A 권한 변경 | 사용자 B 영향 없음 |
| AC-RBAC-011 | 최소 권한 | 허용 범위 외 접근 거부 |

## 5. Implementation Verification

현재 구현과의 주요 검증 대상은 다음과 같다.

```txt
AC-003-rbac
        ↓
User
        ↓
UserRoleService
        ↓
Role
        ↓
RolePermissionService
        ↓
Permission
        ↓
UserAuthorityService
        ↓
JwtAuthenticationFilter
        ↓
SecurityConfig
        ↓
Protected API
```