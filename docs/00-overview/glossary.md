# Glossary

이 문서는 apms-sr 프로젝트에서 반복적으로 사용하는 핵심 용어와 프로젝트 내 의미를 정의한다.

## Authentication \& Authorization

|용어|의미|
|-|-|
|**IAM**|Identity and Access Management. 사용자 식별과 접근 권한을 관리하는 영역|
|**JWT**|JSON Web Token. 본 프로젝트에서 인증 정보를 전달하기 위해 사용하는 토큰 형식|
|**Access Token**|보호된 API 요청에 사용되는 단기 인증 토큰|
|**Refresh Token**|Access Token 갱신을 위해 사용하는 장기 인증 토큰|
|**RTR**|Refresh Token Rotation. Refresh Token 사용 시 기존 토큰을 폐기하고 새로운 Refresh Token을 발급하는 방식|
|**RBAC**|Role-Based Access Control. Role을 기반으로 사용자의 접근 권한을 제어하는 방식|
|**Role**|사용자의 권한 집합을 표현하는 역할|
|**Permission**|특정 리소스 또는 기능에 대한 접근 권한|
|**Replay Attack**|이미 사용되거나 폐기되어야 하는 인증 토큰을 다시 사용하는 공격|
|**Blacklist**|더 이상 유효하지 않은 토큰 또는 식별자를 별도로 관리하여 재사용을 차단하는 방식|

## Infrastructure \& Data

|용어|의미|
|-|-|
|**Redis**|본 프로젝트에서 Refresh Token 및 인증 관련 상태 관리에 사용하는 인메모리 데이터 저장소|
|**MySQL**|애플리케이션의 영속 데이터를 저장하는 관계형 데이터베이스|
|**JPA**|Java Persistence API. 애플리케이션과 관계형 데이터베이스 사이의 객체 영속성을 관리하기 위한 표준|
|**N+1 Query**|하나의 조회 후 연관 데이터를 개별 조회하여 불필요하게 다수의 SQL이 발생하는 문제|
|**Nginx**|클라이언트 요청을 Frontend/Backend로 전달하는 Reverse Proxy|

## Testing \& Verification

|용어|의미|
|-|-|
|**Unit Test**|개별 컴포넌트 또는 단위 동작을 검증하는 테스트|
|**Integration Test**|여러 컴포넌트의 상호작용을 검증하는 테스트|
|**E2E**|End-to-End. 사용자 흐름의 시작부터 종료까지 전체 경로를 검증하는 테스트|
|**Security Test**|인증·인가·공격 시나리오 및 보안 정책을 검증하는 테스트|
|**Execution**|테스트나 측정 계획을 실제 환경에서 실행한 행위와 그 결과|
|**Evidence**|실행 결과를 객관적으로 확인할 수 있도록 보존한 증거|
|**Claim**|프로젝트에서 검증되었다고 주장하는 기술적 사실|

## Performance

|용어|의미|
|-|-|
|**k6**|HTTP 부하 및 성능 테스트를 실행하기 위한 도구|
|**RPS**|Requests Per Second. 초당 처리 요청 수|
|**p50**|전체 요청 중 50%가 이 시간 이하로 완료되는 응답시간|
|**p95**|전체 요청 중 95%가 이 시간 이하로 완료되는 응답시간|
|**p99**|전체 요청 중 99%가 이 시간 이하로 완료되는 응답시간|
|**Error Rate**|전체 요청 중 오류가 발생한 요청의 비율|
|**Baseline**|최적화나 변경 전 비교 기준이 되는 측정 결과|

## Operations \& Reliability

|용어|의미|
|-|-|
|**Health Check**|애플리케이션 또는 의존 서비스의 정상 상태를 확인하는 검사|
|**Observability**|시스템 내부 상태를 Metrics, Logs 등의 외부 신호를 통해 파악할 수 있는 능력|
|**SLO**|Service Level Objective. 서비스가 목표로 유지해야 하는 신뢰성 수준|
|**Incident**|정상적인 서비스 동작에 영향을 주는 장애 또는 운영 사건|
|**Rollback**|문제가 발생한 배포나 변경을 이전 정상 상태로 되돌리는 작업|

## Repository Roles

|용어|의미|
|-|-|
|**SA-1**|요구사항, 설계, 의사결정, 작업 및 기술 지식을 관리하는 PROCESS 저장소|
|**apms-sr**|실제 애플리케이션 코드와 테스트 및 인프라 구현을 관리하는 BUILD 저장소|
|**PR-1A1**|실행 결과, 검증 증거 및 포트폴리오 산출물을 관리하는 PROOF 저장소|
|**Source of Truth**|해당 정보의 최종적인 기준이 되는 저장소 또는 문서|

## Traceability

|용어|의미|
|-|-|
|**PRD**|Product Requirement Document. 무엇을 해결해야 하는지를 정의하는 문서|
|**ADR**|Architecture Decision Record. 중요한 설계 의사결정과 그 근거를 기록하는 문서|
|**TASK**|실제 구현 또는 검증해야 할 작업을 정의하는 문서|
|**Traceability**|요구사항부터 구현·테스트·증거까지의 연결 관계를 추적할 수 있는 상태|



