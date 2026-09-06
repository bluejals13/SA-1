# [Log] Phase 1-4: JPA N+1 쿼리 문제 점검 및 Fetch Join / @EntityGraph 최적화

- **Date:** 2026-08-26
- **Domain:** Backend, JPA, Database Performance Optimization
- **Author:** AI Agent (Antigravity) & bluejck10113

---

## 1. 목적 (Purpose)
* **JPA 연관관계 쿼리 병목 제거:** 엔티티 간 연관관계(`@ManyToMany`, `@OneToMany`) 조회 시 발생하는 불필요한 추가 `SELECT` 쿼리(N+1 문제 및 지연 로딩 단건 쿼리)를 사전에 차단.
* **데이터베이스 I/O 최소화:** Spring Data JPA의 `@EntityGraph` 및 `Fetch Join(JPQL)`을 전략적으로 적용하여 단 1회의 `JOIN` 쿼리로 연관된 엔티티 컬렉션을 일괄 조회(Eager Fetching).
* **시스템 확장성 및 보안 인가 성능 개선:** 매 API 요청마다 실행되는 `UserAuthorityService`(권한 검증) 및 IAM 관리자 기능(역할/권한/사용자 매핑)의 응답 속도 향상.

---

## 2. 변경/추가된 파일 (Modified & Added Files)
* `backend/src/main/java/com/example/demo/iam/role/repository/RoleRepository.java` (수정)
* `backend/src/main/java/com/example/demo/iam/role/service/RolePermissionService.java` (수정)
* `backend/src/main/java/com/example/demo/iam/user/repository/UserRepository.java` (수정)
* `backend/src/main/java/com/example/demo/iam/admin/service/UserRoleService.java` (수정)
* `task_progress.md` (진행 현황 업데이트)

---

## 3. 핵심 분석 및 변경 사항 (Key Changes)

### 1) 엔티티 연관관계 구조 분석 및 모범 사례 확인
* **컬렉션 타입 `Set<T>` 사용:** `User.roles` 및 `Role.permissions`가 모두 `Set`으로 매핑되어 있어, 2단계 이상의 페치 조인(`User -> Role -> Permission`) 시 발생할 수 있는 Hibernate `MultipleBagFetchException` 및 카테시안 곱(Cartesian Product) 중복 데이터 문제를 방지.
* **기본 FetchType `LAZY` 준수:** 모든 연관관계를 `LAZY`로 유지하여 무분별한 즉시 로딩을 방지하고, 필요한 엔드포인트에서만 명시적 패치 조인을 사용하도록 설정.

### 2) 발견된 지연 로딩(N+1/추가 쿼리) 병목 지점 및 해결

| 대상 서비스 및 기능 | 기존 방식 및 문제점 | 최적화 적용 방식 |
| :--- | :--- | :--- |
| **`RolePermissionService.assignPermissions`** | `roleRepository.findById(id)`로 Role 단건 조회 후 `role.getPermissions()` 호출 시 추가 `SELECT` 쿼리 1회 발생 | `RoleRepository`에 `@EntityGraph(attributePaths = "permissions") Optional<Role> findWithPermissionsById(Long id)` 추가 적용하여 1회 쿼리로 조회 |
| **`UserRoleService.assignRoles`** | `userRepository.findById(id)`로 User 단건 조회 후 `user.getRoles()` 호출 시 추가 `SELECT` 쿼리 1회 발생 | `UserRepository`에 `@EntityGraph(attributePaths = "roles") Optional<User> findWithRolesById(Long id)` 추가 적용하여 1회 쿼리로 조회 |
| **`UserRepository.findByPermissionId`** | 단순 `join`으로 조회하여 영속화된 User의 Role/Permission이 프록시로 남아있음 | `join fetch u.roles r join fetch r.permissions p`로 변경하여 즉시 그래프 적재 |
| **`UserRepository.findAllWithRoles`** | 향후 관리자 사용자 목록 조회 화면에서 Role 확장 시 N+1 발생 방지 대비 | `@EntityGraph(attributePaths = "roles")` 기반 `findAllWithRoles()` 메서드 사전 구축 |

### 3) 기존 최적화 검증 완료 지점
* **`UserRepository.findWithRolesAndPermissionsById(Long id)`**: `@EntityGraph(attributePaths = {"roles", "roles.permissions"})`를 통해 JWT 인증 및 인가(`UserAuthorityService`), 내 정보 조회(`UserService.getMe`) 시 `User -> Role -> Permission` 3계층 데이터를 단 1번의 쿼리로 안전하게 패치 조인 완료.
* **`RoleRepository.findAllWithPermissions()`**: `@EntityGraph(attributePaths = "permissions")`를 통해 역할 목록 조회 시 N+1 문제 없이 일괄 패치 조인 완료.

---

## 4. 성능 개선 및 검증 결과 (Verification)
* **쿼리 발생 횟수 비교:**
  - 역할 권한 부여(`assignPermissions`): 기존 2회(`Role 조회` + `permissions LAZY 조회`) ➡️ **최적화 후 1회(`Role LEFT JOIN Permission`)**
  - 사용자 역할 부여(`assignRoles`): 기존 2회(`User 조회` + `roles LAZY 조회`) ➡️ **최적화 후 1회(`User LEFT JOIN Role`)**
* **안정성:** 모든 Repository 메서드에서 지연 로딩 예외(`LazyInitializationException`) 발생 위험을 원천 차단하고 `Set` 기반의 중복 제거 정합성을 확보.
