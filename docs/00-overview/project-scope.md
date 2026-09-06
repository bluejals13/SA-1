# Project Scope

## 1. Purpose

이 문서는 APMS.SR 프로젝트의 현재 범위를 정의한다.

Scope는 프로젝트가 해결해야 하는 문제와 검증해야 하는 영역을 명확하게 하기 위한 기준이며, 기술 스택 자체를 늘리기 위한 목록이 아니다.

범위가 변경될 경우 관련 PRD, ADR 및 TASK를 함께 갱신한다.

---

## 2. In Scope

### 2.1 Application

* Full-stack Web Application
* React 기반 Frontend
* Spring Boot 기반 Backend
* REST API
* 사용자 인증 및 인가

### 2.2 Identity & Access

* IAM
* User / Role / Permission
* JWT 기반 Authentication
* Access Token
* Refresh Token
* Refresh Token Rotation
* Token Replay 방어
* RBAC
* Logout 및 Token 무효화 정책

### 2.3 Data

* MySQL
* Spring Data JPA
* Database Migration
* Entity / DTO 분리
* Query 성능 및 N+1 검증

### 2.4 Infrastructure

* Redis
* Docker
* Docker Compose
* Nginx Reverse Proxy
* GitHub Actions CI/CD

### 2.5 Security Verification

다음과 같은 보안 요구사항을 실제 테스트와 실행 결과로 검증하는 것을 범위에 포함한다.

* Refresh Token Replay
* Concurrent Refresh
* Logout 이후 Token 재사용
* RBAC 권한 분기
* Redis 장애 상황
* CORS 정책
* Secret / Configuration 관리

구체적인 보안 정책과 검증 조건은 `01-prd/security/` 및 관련 ADR/TASK에서 정의한다.

### 2.6 Performance Verification

* k6 기반 부하 테스트
* Baseline 측정
* 최적화 전후 비교
* RPS
* p50
* p95
* p99
* Error Rate
* CPU / Memory
* DB / Redis 영향 분석

단순히 부하 테스트를 실행하는 것이 아니라 동일한 조건에서 재현 가능한 측정을 목표로 한다.

### 2.7 Observability

* Prometheus
* Grafana
* Application Metrics
* Health Check
* 로그 및 실행 결과 기반 문제 분석

운영 수준의 Alerting과 SLO는 실제 요구사항이 확정되는 범위에서 단계적으로 추가한다.

---

## 3. Out of Scope

현재 프로젝트의 핵심 목표와 직접적인 관련이 없는 다음 항목은 기본 범위에서 제외한다.

### 3.1 Distributed Architecture Expansion

* Kubernetes
* MSA
* Service Mesh
* Kafka
* Event-driven distributed architecture

### 3.2 Infrastructure Scaling

* Redis Cluster
* Kubernetes-based autoscaling
* Multi-region deployment
* 대규모 분산 시스템 전환

### 3.3 Additional AI Infrastructure

* Multi-agent platform
* 별도의 AI Agent orchestration system
* AI 모델 serving infrastructure

AI는 개발 과정의 보조 도구와 의사결정 검토 프로세스로 활용하며, AI 자체를 별도의 제품으로 구축하지 않는다.

---

## 4. Scope Boundary

다음 기준으로 범위를 판단한다.

### 포함

현재 시스템의 신뢰성, 보안성, 성능 및 운영 가능성을 높이기 위해 필요한 작업.

### 조건부 포함

현재 시스템에서 실제 문제가 발견되었거나 명확한 요구사항이 발생했을 때 검토한다.

예:

* 새로운 인프라 도입
* 추가 모니터링 시스템
* 배포 전략 변경
* 확장 아키텍처

이 경우 먼저 문제와 도입 조건을 정의하고 ADR을 작성한다.

### 제외

기술 스택의 다양성을 보여주기 위한 목적만으로 추가하는 기술.

---

## 5. Change Policy

Scope 변경이 필요한 경우 다음 절차를 따른다.

```text
Problem
  ↓
Requirement
  ↓
Scope Review
  ↓
ADR
  ↓
TASK
  ↓
Implementation
  ↓
Verification
```

단순히 "새 기술을 사용해보고 싶다"는 이유만으로 Scope를 확장하지 않는다.

---

## 6. Current Boundary

현재 프로젝트의 우선 목표는 다음과 같다.

> **현재 구현된 시스템을 더 많은 기술로 확장하는 것이 아니라, 현재 시스템의 보안·성능·장애 대응·운영 가능성을 실제 검증하고 그 결과를 재현 가능한 증거로 남기는 것.**

따라서 프로젝트의 완성도를 판단할 때 기술 스택의 개수보다 요구사항 → 구현 → 테스트 → 실행 → 증거의 연결성을 우선한다.
