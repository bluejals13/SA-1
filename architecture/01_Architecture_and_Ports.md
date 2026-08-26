# [Architecture] 시스템 아키텍처 및 포트 구성 (Architecture and Ports)

- **Version:** 1.0.0
- **Last Updated:** 2026-08-26
- **Status:** Active

---

## 1. 시스템 전체 아키텍처 개요 (System Overview)

`26-05adf-main` 프로젝트는 **Docker Compose** 기반의 컨테이너 오케스트레이션 환경에서 구동되는 엔터프라이즈급 풀스택 웹 애플리케이션 및 모니터링 시스템입니다.
리버스 프록시(Nginx)를 단일 진입점(Single Entry Point)으로 두어 클라이언트 요청을 프론트엔드 정적 파일 서빙과 백엔드 REST API로 라우팅하며, 통합 모니터링(Prometheus, Grafana, VictoriaMetrics) 스택을 내장하고 있습니다.

```
[ Client / Web Browser ]
           │
           ▼ (Port 80)
┌────────────────────────────────────────────────────────┐
│                      Nginx Web Server                  │
│  - Static SPA Hosting (/usr/share/nginx/html)          │
│  - Reverse Proxy (/api/* ──► Backend:8080)            │
│  - CORS / Security Headers Proxy Setting               │
└──────────────┬─────────────────────────┬───────────────┘
               │ (SPA Dist)              │ (API Proxy)
               ▼                         ▼
      ┌─────────────────┐       ┌────────────────────────┐
      │ React Frontend  │       │ Spring Boot Backend    │
      │ (Vite, TS, SPA) │       │ (Java 17, Spring 3.3)  │
      └─────────────────┘       └───────┬────────┬───────┘
                                        │        │
                     ┌──────────────────┘        └─────────────────┐
                     ▼                                             ▼
        ┌─────────────────────────┐                   ┌────────────────────────┐
        │       MySQL 8.0         │                   │        Redis 7         │
        │ - Database: eventdb     │                   │ - Refresh Token Store  │
        │ - Flyway Migration      │                   │ - JWT Blacklist (TTL)  │
        └─────────────────────────┘                   └────────────────────────┘
```

---

## 2. 서비스별 포트 매핑 (Port Allocation Table)

모든 컨테이너는 동일한 Docker Bridge 네트워크(`app-net`) 내부에서 컨테이너명(도메인)으로 서로 통신합니다.

| 서비스 (Service) | 컨테이너 명 | 내부 포트 (Container) | 외부 포트 (Host) | 프로토콜 | 용도 및 설명 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Nginx** | `nginx` | `80` | `80` (`${NGINX_PORT}`) | HTTP | 전체 단일 진입점, SPA 서빙 및 `/api/` 프록시 |
| **Backend** | `backend` | `8080` | `8080` (`${BACKEND_PORT}`) | HTTP | Spring Boot REST API 서버 & Actuator |
| **MySQL** | `mysql` | `3306` | `3307` | TCP/MySQL | 메인 RDBMS (DB명: `eventdb`) |
| **Redis** | `redis` | `6379` | - (내부 전용) | TCP/Redis | 세션, Refresh Token(JTI), JWT 블랙리스트 캐시 |
| **Prometheus** | `prometheus` | `9090` | `9090` (`${PROMQL_PORT}`) | HTTP | 시계열 메트릭 수집 및 PromQL 엔진 |
| **VictoriaMetrics** | `vm` | `8428` | `8428` | HTTP | 고성능 시계열 데이터 스토리지 |
| **Grafana** | `grafana` | `3000` | `3000` (`${GRAFANA_PORT}`) | HTTP | 인프라 및 애플리케이션 통합 모니터링 대시보드 |
| **Grafana Agent**| `grafana-agent` | `12345` | `12345` | HTTP | 메트릭/로그 수집 및 전송 에이전트 |
| **Node Exporter** | `node-exporter` | `9100` | - (내부 전용) | HTTP | 호스트 OS 하드웨어 및 커널 메트릭 수집 |
| **cAdvisor** | `cadvisor` | `8080` | - (내부 전용) | HTTP | 실행 중인 Docker 컨테이너 리소스 사용량 측정 |

---

## 3. 네트워크 및 트래픽 흐름 (Network & Traffic Flow)

### 1) 사용자 요청 처리 흐름
1. **정적 리소스 요청 (`/`)**: 
   - `Client` ➡️ `Nginx` (Port 80) ➡️ Nginx가 자체 서빙하는 React 빌드 산출물(`index.html`, `js`, `css`) 반환.
   - React Router 대응을 위해 `try_files $uri $uri/ /index.html;` 적용.
2. **API 요청 (`/api/*`)**:
   - `Client` ➡️ `Nginx` (Port 80) ➡️ `backend:8080`으로 리버스 프록시 패스.
   - `X-Real-IP`, `X-Forwarded-For`, CORS Preflight(`OPTIONS 204`) 헤더 자동 처리.

### 2) 백엔드 데이터 영속 계층 흐름
- **비즈니스 트랜잭션**: `Spring Boot` ➡️ `mysql:3306` (Flyway 스키마 버전 관리 및 커넥션 풀링).
- **보안 및 토큰 검증**: `Spring Boot` ➡️ `redis:6379` (JWT Access Token 블랙리스트 체크, Refresh Token 교체 검증).

### 3) 모니터링 메트릭 수집 흐름
- **Spring Actuator**: `backend:8080/actuator/prometheus` ➡️ `Prometheus` (15초 주기 스크랩).
- **시스템 & 컨테이너 메트릭**: `node-exporter:9100`, `cadvisor:8080` ➡️ `Prometheus` ➡️ `VictoriaMetrics` 원격 저장.
- **시각화**: `Grafana` ➡️ `Prometheus` / `VictoriaMetrics` 데이터 소스 연동을 통해 대시보드 표출.

---

## 4. Docker Volume 및 영속성 관리 (Volumes)

- `mysql_data`: MySQL 데이터베이스 파일 영속 저장 (`/var/lib/mysql`). 컨테이너 재생성 시에도 데이터 보존.
- `frontend_dist`: Multi-stage 빌드 시 생성되는 프론트엔드 정적 산출물 공유 볼륨.
