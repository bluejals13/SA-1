# [Conventions] 백엔드 개발 표준 및 코딩 규칙 (Backend Conventions)

- **Version:** 1.0.0
- **Last Updated:** 2026-08-26
- **Status:** Active
- **Applied Tech Stack:** Java 17, Spring Boot 3.3.2, Spring Security, Spring Data JPA, Redis, JJWT

---

## 1. DTO & Entity 분리 규칙 (Record DTO)

### 1) DTO는 불변 Java Record 타입으로 선언
* 모든 요청(`Request`) 및 응답(`Response`) DTO는 Java 17의 `record` 타입을 사용합니다.
* 불필요한 보일러플레이트 코드(Getter, Setter, toString, equals)를 제거하고 불변성(Immutability)을 보장합니다.

```java
// ✅ 권장: record 타입 선언 및 정적 팩토리 메서드 제공
public record RoleResponse(
    Long id,
    String name,
    String description,
    List<PermissionResponse> permissions
) {
    public static RoleResponse from(Role role) {
        return new RoleResponse(
            role.getId(),
            role.getName(),
            role.getDescription(),
            role.getPermissions().stream()
                .map(PermissionResponse::from)
                .toList()
        );
    }
}
```

### 2) Entity를 Controller 응답으로 직접 노출 금지
* Controller는 절대 JPA Entity를 직접 반환해서는 안 되며, 반드시 DTO로 변환하여 반환합니다.
* 순환 참조 방지 및 엔티티 내부 구조 은닉을 철저히 지킵니다.

---

## 2. API 공통 응답 포맷 규약 (`ApiResponse<T>`)

모든 Controller 엔드포인트의 반환 타입은 `ResponseEntity<ApiResponse<T>>`로 통일합니다.

### 1) 응답 구조 (JSON Schema)
```json
{
  "status": "SUCCESS",
  "message": "요청이 성공적으로 처리되었습니다.",
  "data": { ... }
}
```

### 2) `ApiResponse` 팩토리 메서드 규약
* **데이터 반환 시**: `ApiResponse.success(T data)` ➡️ 데이터 객체 래핑 반환
* **메시지만 반환 시 (데이터 없음)**: `ApiResponse.success(String message)` ➡️ `ApiResponse<Void>` 반환
* **에러 응답 시**: `ApiResponse.error(String message)` ➡️ `GlobalExceptionHandler`에서 사용

```java
// Controller 예시
@GetMapping
public ResponseEntity<ApiResponse<List<RoleResponse>>> getRoles() {
    List<RoleResponse> roles = roleAdminService.getRoles();
    return ResponseEntity.ok(ApiResponse.success(roles));
}

@PostMapping
public ResponseEntity<ApiResponse<RoleResponse>> createRole(@Valid @RequestBody RoleRequest request) {
    RoleResponse response = roleAdminService.createRole(getAdminId(), request);
    return ResponseEntity.status(HttpStatus.CREATED).body(ApiResponse.success(response));
}

@DeleteMapping("/{id}")
public ResponseEntity<ApiResponse<Void>> deleteRole(@PathVariable Long id) {
    roleAdminService.deleteRole(getAdminId(), id);
    return ResponseEntity.ok(ApiResponse.success("역할이 성공적으로 삭제되었습니다."));
}
```

---

## 3. 전역 예외 처리 규약 (`GlobalExceptionHandler`)

모든 예외는 `@RestControllerAdvice`를 통해 중앙 집중식으로 제어되며, 클라이언트에게 정제된 일관된 JSON을 반환합니다.

### 1) HTTP 상태 코드 매핑 기준
| 예외 유형 | HTTP Status | 처리 방식 |
| :--- | :--- | :--- |
| **`MethodArgumentNotValidException`** | `400 Bad Request` | `@Valid` 필드 검증 실패 시 첫 번째 필드 에러 메시지 추출 반환 |
| **`IllegalArgumentException`** | `400 Bad Request` | 잘못된 파라미터 또는 비즈니스 유효성 검증 실패 메시지 반환 |
| **`BadCredentialsException`** | `401 Unauthorized` | 인증 실패 및 만료된 세션 처리 ("인증에 실패했습니다.") |
| **`AccessDeniedException`** | `403 Forbidden` | 인가 부족 및 권한 없음 처리 ("접근 권한이 없습니다.") |
| **`Exception` (기타 미제어 예외)** | `500 Internal Server Error`| 서버 시스템 로그(`log.error`) 기록 후 클라이언트에는 보안상 안전한 공통 메시지만 전달 |

---

## 4. JWT & Redis 인증/보안 아키텍처 규칙

```
[ Client ] ──(Access Token)──► [ JwtAuthenticationFilter ]
                                        │
                                        ▼ 1. Redis Blacklist 조회 (탈취 토큰 차단)
                                  [ Redis Store ]
                                        │
                                        ▼ 2. SecurityContext 인증 주입
                               [ UserAuthorityService ]
                                        │ (Fetch Join 권한 적재)
                                        ▼
                                  [ Controller ]
```

### 1) 이중 토큰 전략 (Dual Token Strategy)
* **Access Token**: 유효기간 1시간 (`3600000ms`), 클라이언트(Zustand/메모리) 보관, API 호출 시 `Authorization: Bearer <Token>` 전달.
* **Refresh Token**: 유효기간 7일 (`604800000ms`), `HttpOnly`, `SameSite=Strict`, `Secure` 쿠키에 보관.

### 2) Redis 기반 Rotation & Blacklist
* **Refresh Token Rotation (RTR)**: 토큰 재발급(`/api/auth/refresh`) 시마다 고유 `JTI(JWT ID)`를 갱신하여 1회용으로만 사용 (재사용 감지 시 즉시 세션 무효화).
* **Logout Blacklist**: 로그아웃(`/api/auth/logout`) 시 클라이언트의 Access Token 남은 유효시간(TTL)만큼 Redis에 `blacklist:<token>`으로 등록하여 즉시 무력화.

---

## 5. JPA 연관관계 및 N+1 최적화 규칙

### 1) `@ManyToMany` 및 `@OneToMany`는 반드시 `Set<T>` 사용
* 다대다/일대다 컬렉션 매핑 시 `List` 대신 `Set`을 사용하여 Hibernate `MultipleBagFetchException`을 방지하고 중복 튜플을 필터링합니다.

### 2) 기본 FetchType은 무조건 `LAZY` (지연 로딩)
* 엔티티 정의 시 모든 연관관계는 `FetchType.LAZY`로 선언합니다.

### 3) 조회 시 `@EntityGraph` 또는 `join fetch` 필수 적용
* 컬렉션 연관관계가 필요한 조회 쿼리는 Repository에 `@EntityGraph` 또는 JPQL `join fetch`를 명시하여 단 1회의 SQL 조인으로 가져옵니다.

```java
// UserRepository.java
@EntityGraph(attributePaths = {"roles", "roles.permissions"})
Optional<User> findWithRolesAndPermissionsById(Long id);

@EntityGraph(attributePaths = "roles")
Optional<User> findWithRolesById(Long id);
```
