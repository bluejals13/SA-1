# .커서규칙

## APMS.SR Code Generation Rules

1. Zero-Chatter Policy
   - 절대 사과하지 마세요. (Do not apologize)
   - 묻지 않은 설명을 하지 마세요. (No explanations unless explicitly requested)
   - "여기 코드가 있습니다" 같은 서론을 쓰지 마세요.

2. Code Output Rules
   - 수정된 코드 전체를 다시 작성하지 마세요. 변경된 클래스나 메서드의 Diff(스니펫)만 정확히 제공하세요.
   - 주석은 비즈니스 로직이 매우 복잡할 때만(Why 위주로) 작성하세요.

3. Architecture & Stack Enforcement
   - [Backend] DTO와 Entity는 철저히 분리하고, 의존성 주입은 반드시 `@RequiredArgsConstructor`를 이용한 생성자 주입을 사용하세요.
   - [Backend] 모든 로그는 `System.out.println`이 아닌 `@Slf4j`를 사용하세요.
   - [Frontend] API 통신은 직접 `fetch`를 사용하지 말고 정의된 `http.ts`의 래퍼 메서드(`http.get`, `http.post`)를 사용하세요.

## Documentation First Policy
- 모든 코드를 수정한 후에는 즉시 다음 작업을 진행하지 마세요.
- 반드시 `26-05adf-guideline/changelogs/` 경로에 해당 작업의 변경 로그(Log) 마크다운 파일을 생성해야 합니다.
- 파일명 규칙: `{Phase번호}-{Task번호}_{작업명}.md` (예: 1-2_jwt_redis_optimization.md)


