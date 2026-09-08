

```txt

# 실제 User 구현:
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/iam/user/domain/User.java
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/iam/user/service/UserService.java
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/iam/user/controller/UserController.java
https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/java/com/example/demo/iam/user/repository/UserRepository.java

```

# PRD-FUNC-002 — User

# 1. 개요
## 1.1 목적

시스템 사용자의 계정을 생성하고, 자신의 사용자 정보를 조회하며, 자신의 비밀번호를 변경할 수 있도록 사용자 기능의 요구사항을 정의한다.

## 1.2 범위

본 요구사항은 다음 사용자 기능을 대상으로 한다.

* 사용자 계정 생성
* 사용자 식별 정보 관리
* 자신의 사용자 정보 조회
* 자신의 비밀번호 변경
* 사용자 상태 및 역할 정보와의 연계

관리자에 의한 사용자 관리와 역할 할당은 PRD-FUNC-003-iam에서 정의한다.

# 2. Functional Requirements
## FR-USER-001 — 사용자 계정 생성

시스템은 사용자가 필요한 사용자 정보를 제출하여 사용자 계정을 생성할 수 있도록 해야 한다.

정상적으로 생성된 계정은 시스템에서 식별 가능한 사용자 계정으로 등록되어야 한다.

## FR-USER-002 — 사용자 식별 정보 중복 방지

시스템은 중복된 사용자 식별 정보로 사용자 계정이 생성되지 않도록 해야 한다.

현재 사용자명과 이메일은 각각 고유하게 관리되어야 한다.

## FR-USER-003 — 신규 사용자 상태

정상적으로 생성된 사용자는 시스템이 정의한 초기 상태를 가져야 한다.

현재 신규 사용자는 활성 상태로 생성된다.

## FR-USER-004 — 자신의 사용자 정보 조회

인증된 사용자는 자신의 사용자 정보를 조회할 수 있어야 한다.

사용자는 자신의 정보 조회를 통해 다른 사용자의 정보를 조회할 수 있어서는 안 된다.

## FR-USER-005 — 사용자 역할 정보 연계

시스템은 사용자에게 연결된 역할 정보를 사용자 정보와 연계하여 관리해야 한다.

역할의 부여 및 변경 권한은 PRD-FUNC-003-iam 및 PRD-SEC-003-rbac에서 정의한다.

## FR-USER-006 — 비밀번호 변경

인증된 사용자는 자신의 비밀번호를 변경할 수 있어야 한다.

비밀번호 변경 요청은 시스템이 정의한 비밀번호 정책을 만족해야 한다.

현재 구현에서는 8자 미만의 비밀번호 변경을 허용하지 않는다.

## FR-USER-007 — 비밀번호 변경에 따른 인증 상태 처리

비밀번호 변경 이후 기존 인증 상태의 유효성은 시스템의 보안 정책에 따라 처리되어야 한다.

기존 Refresh Token 등의 인증 상태 무효화 정책은 PRD-SEC-004-token-invalidation에서 정의한다.

# 3. Functional Boundaries

본 문서에서는 다음 구현 세부사항을 요구하지 않는다.

특정 데이터베이스 기술
특정 ORM 기술
특정 비밀번호 해시 알고리즘
특정 API 프레임워크
특정 캐시 또는 세션 저장소
특정 인증 토큰 기술

구현 및 기술 선택은 Architecture 및 ADR 문서에서 정의한다.

# 4. Traceability

| Requirement | Related PRD |
|---|---|
| FR-USER-001 | PRD-FUNC-001-authentication |
| FR-USER-004 | PRD-FUNC-001-authentication |
| FR-USER-005 | PRD-FUNC-003-iam, PRD-SEC-003-rbac |
| FR-USER-006 | Security password policy |
| FR-USER-007 | PRD-SEC-004-token-invalidation |

# 5. Acceptance Criteria Traceability

| Requirement | Acceptance Criteria |
|---|---|
| FR-USER-001 | AC-004-user-signup |
| FR-USER-002 | AC-004-user-signup |
| FR-USER-003 | AC-004-user-signup |
| FR-USER-004 | AC-005-user-profile |
| FR-USER-005 | AC-005-user-profile |
| FR-USER-006 | AC-006-password-change |
| FR-USER-007 | AC-006-password-change |

# 6. Implementation Evidence
```txt
PRD-FUNC-002
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


사용자 기능의 요구사항부터 실제 구현 및 검증 결과까지 추적 가능하도록 관리한다.

