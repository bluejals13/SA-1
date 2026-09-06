
```txt

현재 구현과 관련된 주요 구성요소는 다음과 같다.

UserAuthorityService
UserRoleService
RoleAdminService
RolePermissionService
PermissionAdminService
UserAdminService
SecurityConfig
JwtAuthenticationFilter

각 요구사항의 실제 코드 위치와 테스트 증거는 구현 및 검증 단계에서 연결한다.

```


# PRD-SEC-003 — RBAC

# 1. 개요
## 1.1 목적

시스템은 사용자의 역할(Role)과 권한(Permission)을 기반으로 시스템 기능 및 보호된 자원에 대한 접근을 통제해야 한다.

본 문서는 역할 기반 접근 제어가 무엇을 보장해야 하는지를 정의하며, 특정 Framework, 데이터베이스 또는 권한 저장 구조를 요구하지 않는다.

## 1.2 범위

본 요구사항은 다음 접근 제어 기능을 대상으로 한다.
```txt
사용자와 Role의 연계
Role과 Permission의 연계
인증 사용자의 권한 구성
권한 기반 접근 제어
권한이 없는 요청의 거부
Role 변경에 따른 권한 변경
IAM 관리 기능에 대한 접근 제어
```
Role 및 Permission 관리 기능 자체는 PRD-FUNC-003-iam에서 정의한다.

JWT를 통한 사용자 식별 및 인증 상태 구성은 PRD-SEC-001-jwt에서 정의한다.

# 2. Security Requirements
## SEC-RBAC-001 — 사용자 Role 연계

시스템은 사용자에게 하나 이상의 Role을 연결할 수 있어야 한다.

사용자의 접근 권한은 해당 사용자에게 부여된 Role을 기반으로 결정할 수 있어야 한다.

## SEC-RBAC-002 — Role Permission 연계

시스템은 Role에 하나 이상의 Permission을 연결할 수 있어야 한다.

Role을 통해 사용자에게 적용되는 접근 권한을 결정할 수 있어야 한다.

## SEC-RBAC-003 — 권한 기반 접근 제어

시스템은 인증된 사용자의 현재 권한을 기준으로 보호된 기능 또는 자원에 대한 접근 가능 여부를 판단해야 한다.

사용자가 요구된 Permission을 보유하지 않은 경우 해당 기능 또는 자원에 접근할 수 없어야 한다.

## SEC-RBAC-004 — 인증과 인가의 분리

시스템은 사용자가 인증되었다는 사실과 해당 사용자가 특정 기능에 접근할 권한을 가지고 있다는 사실을 구분해야 한다.

인증에 성공한 사용자라고 해서 모든 보호된 기능에 자동으로 접근할 수 있도록 처리해서는 안 된다.

## SEC-RBAC-005 — 현재 권한 반영

시스템은 보호된 요청을 처리할 때 해당 사용자의 현재 권한 상태를 기준으로 접근을 판단해야 한다.

이전에 발급된 인증 정보만을 근거로 이미 변경된 권한 상태를 계속 유효한 것으로 취급해서는 안 된다.

## SEC-RBAC-006 — Role 변경의 권한 반영

사용자의 Role이 변경된 경우 이후 보호된 요청에서는 변경된 Role에 따라 권한을 적용할 수 있어야 한다.

Role 변경 이전에 부여되었던 권한을 변경 이후에도 무조건 유지해서는 안 된다.

## SEC-RBAC-007 — Permission 변경의 권한 반영

Role에 연결된 Permission이 변경된 경우 이후 보호된 요청에서는 변경된 Permission 상태를 기준으로 접근을 판단할 수 있어야 한다.

## SEC-RBAC-008 — 권한 없는 요청 거부

사용자가 요구된 Permission을 보유하지 않은 경우 시스템은 해당 요청을 정상적인 성공 요청으로 처리해서는 안 된다.

권한 부족으로 거부된 요청을 인증 성공과 동일하게 처리해서는 안 된다.

## SEC-RBAC-009 — IAM 관리 기능 보호

사용자, Role, Permission 및 Menu 등의 IAM 관리 기능은 해당 관리 기능에 접근할 수 있는 권한을 가진 사용자에게만 제공되어야 한다.

IAM 관리 기능의 존재 자체와 해당 기능에 대한 접근 권한은 구분하여 관리할 수 있어야 한다.

## SEC-RBAC-010 — 권한 경계 독립성

한 사용자의 Role 또는 Permission 변경이 다른 사용자의 권한에 의도하지 않은 영향을 주어서는 안 된다.

## SEC-RBAC-011 — 최소 권한 원칙

사용자에게 부여되는 접근 권한은 해당 사용자에게 실제로 필요한 범위로 제한될 수 있어야 한다.

사용자가 특정 기능에 접근하기 위해 불필요한 추가 권한을 자동으로 획득해서는 안 된다.

# 3. Access Control Boundaries

본 문서에서는 다음 구현 방법을 요구하지 않는다.
```txt
Spring Security
특정 Authorization Annotation
특정 Role/Permission Entity 구조
특정 Database
특정 Repository
특정 Cache
특정 API Gateway
특정 권한 검사 방식
```
구체적인 구현 및 아키텍처 선택은 Architecture 및 ADR에서 정의한다.

# 4. Functional Traceability

| Requirement | Related Functional PRD |
|---|---|
| SEC-RBAC-001 | PRD-FUNC-002-user, PRD-FUNC-003-iam |
| SEC-RBAC-002 | PRD-FUNC-003-iam |
| SEC-RBAC-003 | PRD-FUNC-001-authentication, PRD-FUNC-003-iam |
| SEC-RBAC-004 | PRD-FUNC-001-authentication |
| SEC-RBAC-005 | PRD-FUNC-001-authentication |
| SEC-RBAC-006 | PRD-FUNC-003-iam |
| SEC-RBAC-007 | PRD-FUNC-003-iam |
| SEC-RBAC-009 | PRD-FUNC-003-iam |

# 5. Related Security PRD

| Related PRD | Relationship |
|---|---|
| PRD-SEC-001-jwt | 인증된 사용자의 식별 및 인증 상태 구성 |
| PRD-SEC-002-refresh-token | 인증 상태 갱신 |
| PRD-SEC-004-token-invalidation | 인증 상태 종료 및 Token 무효화 |



# 6. Implementation Evidence

본 PRD는 특정 구현을 요구하지 않는다.

현재 구현과의 추적은 다음 단계에서 연결한다.
```txt
PRD-SEC-003-rbac
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

```
