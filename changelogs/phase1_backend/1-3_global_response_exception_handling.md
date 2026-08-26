# [Log] Phase 1-3: API 공통 응답 포맷 및 전역 예외 처리(Global Exception Handler) 구축

- **Date:** 2026-08-26
- **Domain:** Common, Exception Handling, API Response
- **Author:** AI Agent (Antigravity) & bluejck10113

## 1. 목적 (Purpose)
* **프론트엔드 연동 최적화:** API의 응답 형식을 `status`, `message`, `data` 구조로 통일하여 React(Frontend)에서 `Axios Interceptor`를 통한 공통 에러 처리가 용이하도록 개선.
* **에러 메시지 캡슐화:** Spring Boot의 기본 에러 응답 트레이스가 클라이언트(브라우저)에 노출되는 보안 취약점을 차단.
* **유지보수성 증대:** 비즈니스 예외 및 `@Valid` 예외를 `@RestControllerAdvice`에서 중앙 집중식으로 핸들링.

## 2. 변경/추가된 파일 (Modified & Added Files)
* `backend/src/main/java/com/example/demo/common/dto/ApiResponse.java` (신규)
* `backend/src/main/java/com/example/demo/common/exception/GlobalExceptionHandler.java` (신규)
* `backend/src/main/java/com/example/demo/iam/role/RoleAdminController.java` 외 전체 Controller 클래스

## 3. 핵심 변경 사항 (Key Changes)

### 1) `ApiResponse.java` (공통 응답 래퍼 도입)
* 성공(`success`)과 실패(`error`) 팩토리 메서드를 포함한 Record 타입의 공통 DTO 생성.
* 모든 API 응답이 `{ "status": "SUCCESS", "message": "...", "data": {...} }` 형태로 반환되도록 규격화.
* 제네릭 타입(`<T>`)을 활용하여 데이터가 있는 응답과 메시지만 있는 응답(`Void`)을 안전하게 구분.

### 2) `GlobalExceptionHandler.java` (전역 예외 처리기)
* `@RestControllerAdvice`를 활용해 어플리케이션 전역의 예외를 캐치.
* `@Valid` 실패 시 발생하는 `MethodArgumentNotValidException`을 캐치해, 400 Bad Request와 함께 첫 번째 필드의 정제된 에러 메시지(예: "역할명은 필수입니다.") 반환.
* 비즈니스 및 인가 예외(`IllegalArgumentException`, `AccessDeniedException`, `BadCredentialsException`)를 캐치하여 각각 400, 403, 401 상태 코드와 명확한 메시지로 응답하도록 설정.
* 제어되지 않은 서버 예외(`Exception.class`) 발생 시 시스템 로그(`log.error`)를 남기고, 클라이언트에는 "서버 내부 오류가 발생했습니다"라는 안전한 메시지만 반환.

### 3) Controller 일괄 리팩토링 및 타입 불일치 해결
* 각 도메인(Auth, IAM, Audit 등)의 Controller 반환 타입을 `ResponseEntity<ApiResponse<T>>` 형식으로 전면 교체.
* 반환 객체(Data)가 있는 경우 `ApiResponse.success(문자열)`을 사용하여 발생하던 컴파일 에러(타입 불일치 및 데이터 누락)를 `ApiResponse.success(실제객체)`로 수정하여 데이터 유실 방지.

## 4. 테스트 및 검증 결과 (Verification)
* 잘못된 DTO 요청 시 `GlobalExceptionHandler`가 작동하여 일관된 JSON 에러 메시지를 반환함을 검증.
* Controller의 반환 타입(`ApiResponse<T>`)과 실제 주입되는 데이터 간의 제네릭 타입 일치를 확인하고 컴파일 에러 해결 및 빌드 성공.