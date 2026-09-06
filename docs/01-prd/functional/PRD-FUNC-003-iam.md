
```

https://github.com/bluejals13/26-05adf/tree/feature/auth%400603%401401/backend/src/main/java/com/example/demo/iam/admin
https://github.com/bluejals13/26-05adf/tree/feature/auth%400603%401401/backend/src/main/java/com/example/demo/iam/menu

https://github.com/bluejals13/26-05adf/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/iam/role/RoleAdminController.java
https://github.com/bluejals13/26-05adf/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/iam/role/service/RoleAdminService.java
https://github.com/bluejals13/26-05adf/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/iam/role/service/RolePermissionService.java

https://github.com/bluejals13/26-05adf/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/iam/permission/PermissionAdminController.java
https://github.com/bluejals13/26-05adf/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/iam/permission/service/PermissionAdminService.java

https://github.com/bluejals13/26-05adf/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/iam/menu/MenuAdminController.java
https://github.com/bluejals13/26-05adf/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/iam/menu/service/MenuAdminService.java

https://github.com/bluejals13/26-05adf/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/iam/admin/UserAdminController.java
https://github.com/bluejals13/26-05adf/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/iam/admin/service/UserRoleService.java
https://github.com/bluejals13/26-05adf/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/iam/admin/service/UserAdminService.java

```


# PRD-FUNC-003 — IAM

# 1. 개요
## 1.1 목적

시스템 관리자가 사용자, 역할, 권한 및 메뉴 정보를 관리할 수 있도록 IAM 기능의 요구사항을 정의한다.

본 문서는 IAM 관리 기능이 제공해야 하는 행위를 정의하며, 접근 제어의 구체적인 정책은 PRD-SEC-003-rbac에서 정의한다.

## 1.2 범위

본 요구사항은 다음 IAM 관리 기능을 대상으로 한다.

사용자 관리
사용자 Role 관리
Role 관리
Role과 Permission의 관계 관리
Permission 조회
Menu 관리

# 2. User Administration
## FR-IAM-USER-001 — 사용자 목록 조회

관리자는 시스템에 등록된 사용자 목록을 조회할 수 있어야 한다.

## FR-IAM-USER-002 — 사용자 상태 변경

관리자는 사용자의 상태를 변경할 수 있어야 한다.

시스템은 허용되지 않은 상태 변경을 수행해서는 안 된다.

## FR-IAM-USER-003 — 사용자 삭제 대기 처리

관리자는 사용자를 삭제 대기 상태로 전환할 수 있어야 한다.

## FR-IAM-USER-004 — 사용자 영구 삭제

삭제 대기 상태의 사용자는 영구 삭제할 수 있어야 한다.

삭제 대기 상태가 아닌 사용자를 동일한 삭제 절차로 즉시 영구 삭제해서는 안 된다.

## FR-IAM-USER-005 — 사용자 Role 집합 관리

관리자는 사용자의 Role 집합을 관리할 수 있어야 한다.

Role 변경 요청에 포함된 Role이 유효하지 않은 경우 변경을 적용해서는 안 된다.

# 3. Role Management
## FR-IAM-ROLE-001 — Role 목록 조회

관리자는 Role 목록을 조회할 수 있어야 한다.

## FR-IAM-ROLE-002 — Role 생성

관리자는 새로운 Role을 생성할 수 있어야 한다.

## FR-IAM-ROLE-003 — Role 수정

관리자는 기존 Role 정보를 수정할 수 있어야 한다.

## FR-IAM-ROLE-004 — Role 삭제

관리자는 기존 Role을 삭제할 수 있어야 한다.

## FR-IAM-ROLE-005 — Role Permission 집합 관리

관리자는 Role에 연결된 Permission 집합을 관리할 수 있어야 한다.

Permission 변경 요청에 포함된 Permission이 유효하지 않은 경우 변경을 적용해서는 안 된다.

# 4. Permission Management
## FR-IAM-PERM-001 — Permission 목록 조회

관리자는 등록된 Permission 목록을 조회할 수 있어야 한다.

## FR-IAM-PERM-002 — Permission 상세 조회

관리자는 특정 Permission의 상세 정보를 조회할 수 있어야 한다.

상세 정보에는 해당 Permission과 연결된 Role 정보를 포함할 수 있어야 한다.

본 기능 범위에서는 Permission 생성, 수정 및 삭제를 요구하지 않는다.

# 5. Menu Management
## FR-IAM-MENU-001 — Menu 목록 조회

관리자는 등록된 Menu 목록을 조회할 수 있어야 한다.

## FR-IAM-MENU-002 — Menu 상세 조회

관리자는 특정 Menu의 상세 정보를 조회할 수 있어야 한다.

## FR-IAM-MENU-003 — Menu 생성

관리자는 새로운 Menu를 생성할 수 있어야 한다.

## FR-IAM-MENU-004 — Menu 수정

관리자는 기존 Menu 정보를 수정할 수 있어야 한다.

## FR-IAM-MENU-005 — Menu 삭제

관리자는 기존 Menu를 삭제할 수 있어야 한다.

# 6. Access Control Boundary

IAM 관리 기능에 대한 접근 권한은 기능 자체와 분리하여 관리한다.

본 문서는 특정 Permission 이름이나 구현 방식을 기능 요구사항으로 강제하지 않는다.

IAM 관리 기능에 대한 역할 및 권한 기반 접근 통제는 PRD-SEC-003-rbac에서 정의한다.

# 7. Functional Boundaries

본 문서에서는 다음 구현 세부사항을 요구하지 않는다.

특정 데이터베이스 기술
특정 ORM 기술
특정 캐시 기술
특정 API 프레임워크
특정 인증/인가 라이브러리
특정 인프라 구성

# 8. Traceability
Requirement Area	Related PRD
User Administration	PRD-FUNC-002-user, PRD-SEC-003-rbac
User Role Management	PRD-SEC-003-rbac
Role Management	PRD-SEC-003-rbac
Permission Management	PRD-SEC-003-rbac
Menu Management	PRD-SEC-003-rbac

# 9. Acceptance Criteria Traceability
Requirement Area	Acceptance Criteria
User Administration	AC-007-user-administration
User Role Management	AC-008-user-role-management
Role Management	AC-009-role-management
Role Permission Management	AC-010-role-permission-management
Permission Management	AC-011-permission-read
Menu Management	AC-012-menu-management

Acceptance Criteria는 실제 Controller/Service의 동작과 테스트 가능성을 기준으로 세부 조건을 정의한다.

# 10. Implementation Evidence
```txt
PRD-FUNC-003
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

IAM 관리 기능의 요구사항부터 실제 구현 및 검증 결과까지 추적 가능하도록 관리한다.