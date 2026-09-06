# Project Overview

## 1. Project

**APMS.SR**는 인증·인가를 중심으로 한 풀스택 웹 서비스를 대상으로, 기능 구현뿐 아니라 보안·성능·운영·검증까지 연결하는 엔지니어링 구조를 구축하는 프로젝트다.

이 프로젝트의 핵심 관심사는 특정 프레임워크의 사용 자체가 아니라 다음의 개발 흐름을 하나의 체계로 연결하는 것이다.

```text
Requirement
    ↓
Architecture / Design Decision
    ↓
Implementation
    ↓
Test
    ↓
Execution / Measurement
    ↓
Evidence
    ↓
Verification
```

## 2. Problem

일반적인 기능 개발에서는 기능이 구현되었는지를 확인하는 것에 집중하기 쉽다.

이 프로젝트에서는 다음 질문까지 검증 대상으로 포함한다.

* 요구사항이 명확하게 정의되었는가?
* 설계 결정에 근거가 있는가?
* 실제 구현이 설계와 일치하는가?
* 보안 요구사항이 공격 시나리오에서도 유지되는가?
* 장애 상황에서 시스템의 동작이 정의되어 있는가?
* 성능을 수치로 측정할 수 있는가?
* 테스트가 실제로 실행되었는가?
* 실행 결과를 재현 가능한 증거로 남길 수 있는가?

따라서 프로젝트의 목표는 단순히 "동작하는 애플리케이션"을 만드는 것이 아니라 **구현된 시스템을 설명하고, 검증하고, 증명할 수 있는 엔지니어링 구조를 만드는 것**이다.

## 3. Current Implementation

실제 애플리케이션 구현은 `26-05adf` 저장소를 기준으로 한다.

현재 구현의 주요 기술 구성은 다음과 같다.

| 영역                  | 기술                                            |
| ------------------- | --------------------------------------------- |
| Backend             | Spring Boot, Spring Security, Spring Data JPA |
| Authentication      | JWT                                           |
| Authorization       | RBAC                                          |
| Frontend            | React, Vite                                   |
| Database            | MySQL                                         |
| Token / State Store | Redis                                         |
| Reverse Proxy       | Nginx                                         |
| Container           | Docker, Docker Compose                        |
| CI/CD               | GitHub Actions                                |
| Monitoring          | Prometheus, Grafana                           |

`26-05adf`의 README에 명시된 현재 시스템 흐름은 다음과 같다.

```text
Browser
   ↓
Cloudflare
   ↓
Nginx
   ├──→ React
   └──→ Spring Boot
             ├──→ MySQL
             └──→ Redis

Spring Boot
   ↓
Prometheus
   ↓
Grafana
```

구현의 세부사항과 현재 동작 여부에 대한 최종 판단은 항상 `26-05adf`의 실제 소스 코드, 테스트 및 실행 결과를 기준으로 한다.

## 4. Three-Repository Model

이 프로젝트는 세 개의 저장소를 하나의 엔지니어링 흐름으로 사용한다.

### 4.1 `SA-1` — PROCESS

기술적 요구사항, 아키텍처, 설계 의사결정, 작업 계획, 기술 지식을 관리한다.

```text
Why?
What?
How?
What should be done?
```

### 4.2 `26-05adf` — BUILD

실제 애플리케이션과 테스트, 인프라 설정을 구현한다.

```text
What was actually implemented?
```

### 4.3 `PR-1A1` — PROOF

테스트 실행 결과, 성능 측정, 검증 결과, 증거와 발표 자료를 관리한다.

```text
Does it actually work?
How was it verified?
What evidence supports the claim?
```

전체 관계는 다음과 같다.

```text
             SA-1
        PROCESS / DESIGN
               │
               ▼
        26-05adf
       BUILD / TEST
               │
               ▼
          PR-1A1
        PROOF / EVIDENCE
```

## 5. Engineering Traceability

주요 작업은 다음 관계를 유지한다.

```text
PRD
 ↓
ADR
 ↓
TASK
 ↓
Implementation
 ↓
Test
 ↓
Execution
 ↓
Evidence
 ↓
Claim
```

각 단계의 역할은 다음과 같다.

| 단계             | 질문                   |
| -------------- | -------------------- |
| PRD            | 무엇을 해결해야 하는가?        |
| ADR            | 왜 이 설계를 선택했는가?       |
| TASK           | 무엇을 구현·검증해야 하는가?     |
| Implementation | 실제로 무엇을 만들었는가?       |
| Test           | 어떤 조건을 검증하는가?        |
| Execution      | 실제 실행 결과는 무엇인가?      |
| Evidence       | 결과를 무엇으로 증명하는가?      |
| Claim          | 최종적으로 무엇을 주장할 수 있는가? |

## 6. Source of Truth

문서에 기록된 내용은 실제 구현과 일치해야 한다.

각 영역의 기준은 다음과 같다.

| 정보       | Source of Truth               |
| -------- | ----------------------------- |
| 실제 코드    | `26-05adf`                    |
| 실제 테스트   | `26-05adf`                    |
| 실제 실행 결과 | `PR-1A1` Evidence / Execution |
| 설계 의사결정  | `SA-1` ADR                    |
| 요구사항     | `SA-1` PRD                    |
| 작업 상태    | `SA-1` Task Progress          |
| 기술 지식    | `SA-1` Knowledge              |

문서에 존재하지만 실제 구현되지 않은 기능을 현재 기능처럼 기록하지 않는다.

## 7. Engineering Principle

이 프로젝트에서 사용하는 핵심 원칙은 다음과 같다.

> **Claim ≠ Implementation ≠ Test ≠ Execution ≠ Evidence**

기능이 구현되어 있다는 사실만으로 해당 기능이 검증되었다고 판단하지 않는다.

예를 들어 보안 기능은 다음 단계를 모두 구분한다.

```text
Security Requirement
      ↓
Security Design
      ↓
Implementation
      ↓
Security Test
      ↓
Actual Execution
      ↓
Execution Result
      ↓
Evidence
```

## 8. Scope of This Repository

`SA-1`은 실제 애플리케이션 코드를 저장하는 저장소가 아니다.

이 저장소의 책임은 다음과 같다.

* 프로젝트 요구사항 정의
* 아키텍처 문서화
* 설계 의사결정 기록
* 개발 작업 관리
* 기술 지식 정리
* AI-assisted engineering workflow 기록
* 운영 및 검증을 위한 프로세스 정의

실제 구현은 `26-05adf`, 최종 검증 증거는 `PR-1A1`에서 관리한다.
