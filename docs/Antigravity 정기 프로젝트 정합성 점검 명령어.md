# Antigravity 정기 프로젝트 정합성 점검 명령어

## 목적

이 명령어는 프로젝트의 문서, 구현, 테스트, 실행 결과, 증거 사이의 정합성을 주기적으로 점검하기 위한 것이다.

목적은 문서를 많이 만드는 것이 아니다.

다음 흐름이 실제로 유지되고 있는지 검증한다.

`Requirement → PRD → ADR → TASK → Implementation → Test → Execution → Evidence → Claim`

특히 다음 원칙을 지킨다.

`CLAIM ≠ IMPLEMENTATION ≠ TEST ≠ EXECUTION ≠ EVIDENCE`

---

# 1. 기본 정기 점검 명령어

다음 명령어를 프로젝트 루트에서 실행한다.

```text
현재 프로젝트 전체 상태를 정기 감사하라.

대상:
- SA-1
- 26-05adf
- PR-1A1

목적:
새로운 기능이나 문서를 임의로 추가하지 말고,
현재 프로젝트의 요구사항 → 설계 → 구현 → 테스트 → 실행 → 증거 → 주장 사이의 정합성을 점검한다.

다음 순서로 검사하라.

1. Git 상태 확인
   - 각 repository의 현재 branch
   - uncommitted 변경
   - 최근 commit
   - 최근 변경 파일
   - 의도하지 않은 변경 여부

2. SA-1 문서 구조 확인
   - 00-overview
   - 01-prd
   - 02-architecture
   - 03-adr
   - 04-task-progress
   - 05-knowledge
   - 06-ai-engineering
   - 07-operations

3. PRD → ADR 정합성 검사
   - 각 핵심 PRD가 어떤 ADR과 연결되는지 확인
   - ADR에서 결정한 내용이 PRD 요구사항을 충족하는 방향인지 확인
   - PRD에 없는 요구사항을 ADR이 임의로 만들어내고 있지 않은지 확인
   - ADR이 구현 세부사항을 불필요하게 과도하게 고정하고 있지 않은지 확인

4. ADR → Implementation 정합성 검사
   - ADR-0001~현재 ADR의 Decision을 확인
   - 실제 26-05adf 구현과 비교
   - 문서에는 존재하지만 코드에 없는 결정
   - 코드에는 존재하지만 ADR에 반영되지 않은 중요한 결정
   - 문서와 코드가 서로 다른 동작을 설명하는 부분
   를 분리해서 보고하라.

5. Implementation → Test 정합성 검사
   - 핵심 보안/기능/성능 결정이 실제 테스트로 검증되는지 확인
   - 테스트가 존재한다는 사실만으로 검증 완료라고 판단하지 말 것
   - 테스트가 실제 요구사항과 ADR의 Verification 조건을 검증하는지 확인

6. Test → Execution 정합성 검사
   - 테스트 코드가 실제 실행되었는지 확인
   - 최근 실행 결과 확인
   - 성공/실패 여부 확인
   - 실패한 테스트를 성공으로 간주하지 말 것
   - 실행하지 않은 테스트는 검증 완료로 표시하지 말 것

7. Execution → Evidence 정합성 검사
   - 실제 실행 결과가 PR-1A1에 증거로 남아 있는지 확인
   - 실행 결과와 Evidence의 내용이 일치하는지 확인
   - 캡처/로그/수치/리포트의 출처를 확인
   - 실행하지 않은 결과를 Evidence로 만들지 말 것

8. Evidence → Claim 정합성 검사
   - PR-1A1의 주장(Claim)을 확인
   - 각 Claim에 실제 Evidence가 연결되는지 확인
   - Evidence보다 강한 표현을 Claim에서 사용하고 있지 않은지 확인
   - "구현했다", "테스트했다", "실행했다", "검증했다", "성능이 개선됐다"를 서로 동일한 의미로 취급하지 말 것

9. 보안 검증 상태 확인
   특히 다음 항목을 별도로 확인한다.

   - JWT
   - Access Token
   - Refresh Token
   - Refresh Token Rotation
   - Refresh Token Replay
   - Concurrent Refresh
   - Logout Token Reuse
   - RBAC
   - Permission
   - Redis Failure
   - CORS
   - Secret / Configuration

10. 성능 검증 상태 확인
   - N+1
   - Query count
   - 응답시간
   - p50
   - p95
   - p99
   - RPS
   - CPU / Memory
   - 개선 전/후 비교 가능 여부

11. 운영/장애 검증 상태 확인
   - Redis 장애
   - DB 장애
   - 서비스 재시작
   - 로그
   - correlation ID
   - health check
   - backup / restore
   - rollback
   - monitoring / alerting

12. 과잉 개발 여부 확인
   다음 항목을 단순히 포트폴리오를 위해 추가할 필요가 있는지 검토하지 말고,
   실제 요구사항/병목/장애/운영 문제의 존재 여부만 판단한다.

   - Kubernetes
   - Kafka
   - Microservices
   - Redis Cluster
   - Service Mesh
   - GitOps
   - 추가 AI Agent
   - 중복 문서
   - 의미 없는 추가 테스트

   실제 Trigger가 없다면 "추가하지 않음"으로 판단한다.

13. 문서 중복/불일치 확인
   - 동일한 사실을 여러 문서에서 서로 다르게 설명하는지 확인
   - Source of Truth가 존재하는데 복사본을 유지하고 있는지 확인
   - Current State 문서에 미래 계획이나 추측이 섞여 있는지 확인
   - ADR의 역사적 맥락과 현재 상태를 혼동하고 있는지 확인

14. 변경 필요성 판단
   발견된 문제를 다음 4가지로 분류하라.

   A. 즉시 수정 필요
   - 실제 구현과 문서가 다름
   - 보안 정책과 구현이 다름
   - 테스트 결과가 잘못 기록됨
   - Evidence가 Claim을 뒷받침하지 못함

   B. 다음 작업에서 수정
   - 추적성 부족
   - 테스트/Evidence 연결 부족
   - 문서 표현의 불명확성

   C. 선택적 개선
   - 문서 가독성
   - 표현 개선
   - 구조 개선

   D. 수정하지 않음
   - 현재 동작에 문제가 없음
   - 실제 요구사항이 없음
   - 단순히 기술을 추가하기 위한 제안

15. 최종 보고서를 다음 형식으로 출력하라.

## Project Health

- Overall:
- Critical Issues:
- Traceability:
- Security:
- Performance:
- Operations:
- Documentation Consistency:

## A. 즉시 수정 필요

각 항목:
- 문제
- 근거
- 영향
- 관련 파일
- 권장 조치

## B. 다음 작업에서 수정

동일 형식

## C. 선택적 개선

동일 형식

## D. 수정하지 않음

왜 수정하지 않는지 설명

## Traceability Gaps

다음 형식으로 표시한다.

PRD
→ ADR
→ TASK
→ Implementation
→ Test
→ Execution
→ Evidence
→ Claim

각 단계가 존재하는지 [PASS / GAP]로 표시한다.

## Final Recommendation

마지막에는 반드시 다음 중 하나를 선택한다.

- CONTINUE
- FIX THEN CONTINUE
- STOP AND INVESTIGATE

그리고 그 이유를 간결하게 설명한다.

중요:

- 문제를 발견했다고 해서 임의로 코드를 수정하지 말 것.
- 문제를 발견했다고 해서 임의로 ADR/PRD를 추가하지 말 것.
- 실제 파일과 실행 결과를 확인하지 않고 "완료"라고 판단하지 말 것.
- 구현되지 않은 내용을 구현된 것으로 기록하지 말 것.
- 테스트가 존재하는 것과 테스트가 실행된 것을 구분할 것.
- 실행된 것과 증거가 존재하는 것을 구분할 것.
- 증거가 존재하는 것과 주장을 증명하는 것을 구분할 것.
- 기술 추가를 성과로 간주하지 말 것.
- 현재 프로젝트의 범위를 불필요하게 확대하지 말 것.
```

## 2. 점검 주기

매번 전체 감사를 할 필요는 없다.

### 작은 변경 후

**PR/기능/ADR 1개 정도가 변경됐을 때**

```text
변경된 영역을 중심으로
PRD → ADR → Implementation → Test
정합성만 점검하라.
```

### 기능 단위 완료 시

**보안 기능, 인증 기능, RBAC, 성능 개선 등이 하나 끝났을 때**

```text
해당 기능의
PRD → ADR → TASK → Implementation → Test → Execution → Evidence
전체 추적성을 점검하라.
```

### 주기적 전체 감사

**큰 작업 묶음이 끝날 때마다**

```text
SA-1 / 26-05adf / PR-1A1 전체를 대상으로
프로젝트 정합성 감사를 수행하라.

문서를 추가하는 것이 목적이 아니다.
현재 상태를 검증하고 GAP만 찾아라.
```

### 릴리스/포트폴리오 발표 전

이때는 가장 강하게 검사한다.

```text
현재 프로젝트를 최종 포트폴리오 검증 대상으로 간주하라.

모든 주요 Claim에 대해
실제 Implementation
→ Test
→ Execution
→ Evidence
가 존재하는지 확인하라.

증거가 없는 Claim은 "검증되지 않음"으로 분류하라.

구현만 존재하는 경우:
IMPLEMENTED

테스트만 존재하는 경우:
TESTED

테스트가 실제 실행된 경우:
EXECUTED

실행 결과가 증거로 보존된 경우:
EVIDENCED

Evidence가 Claim을 직접 뒷받침하는 경우:
VERIFIED

각 상태를 구분하여 보고하라.
```

---

# 3. 지금 프로젝트에 특히 중요한 운영 원칙

앞으로 안티그래비티에게 **이 한 문장을 항상 기억시키는 게 좋다.**

```text
문서를 늘리는 것보다 현재 시스템의 결정이 실제 구현되고,
그 구현이 테스트되고,
실제로 실행되고,
실행 결과가 증거로 남고,
그 증거가 주장을 뒷받침하는지를 우선하라.
```

그리고 네가 지금 `ADR-0001~0006`까지 끝낸 상황에서는 **이 정기 감사 명령어를 바로 적용하기보다**, 이제 `01-prd → 03-adr → 26-05adf` 연결을 먼저 만드는 게 맞아.

즉 다음 작업부터는 안티그래비티에게:

```text
"ADR을 더 만들어라"
```

가 아니라

```text
"현재 PRD와 ADR을 실제 구현과 대조하고
추적성 GAP을 찾아라."
```

라고 시키는 단계로 넘어간 거야.