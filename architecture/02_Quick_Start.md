# [Guide] 퀵 스타트 및 개발 환경 가이드 (Quick Start Guide)

- **Version:** 1.0.0
- **Last Updated:** 2026-08-26
- **Status:** Active

---

## 1. 사전 요구사항 (Prerequisites)

로컬 개발 및 컨테이너 빌드를 위해 다음 도구들이 설치되어 있어야 합니다.

* **Java**: OpenJDK 17 LTS (Eclipse Temurin 권장)
* **Node.js**: v20.x LTS 이상 (npm v10 이상)
* **Docker**: Docker Engine 24.0+ & Docker Compose v2.20+
* **IDE**: IntelliJ IDEA (Backend), VS Code / Cursor (Frontend)

---

## 2. 환경변수 설정 (.env)

프로젝트 루트 디렉터리에 `.env` 파일을 생성하고 다음 환경변수를 설정합니다.

```env
# -------------------------------------------------------------
# Web & Server Ports
# -------------------------------------------------------------
NGINX_PORT=80
BACKEND_PORT=8080
GRAFANA_PORT=3000
PROMQL_PORT=9090

# -------------------------------------------------------------
# Database Configuration (MySQL)
# -------------------------------------------------------------
MYSQL_USER=root
MYSQL_PASSWORD=password
SPRING_DATASOURCE_URL=jdbc:mysql://mysql:3306/eventdb?serverTimezone=Asia/Seoul&useUnicode=true&characterEncoding=UTF-8

# -------------------------------------------------------------
# Monitoring Credentials (Grafana)
# -------------------------------------------------------------
GRAFANA_ID=admin
GRAFANA_PW=1234
```

---

## 3. 전체 인프라 원클릭 실행 (Docker Compose)

프로젝트 루트 디렉터리에서 다음 명령어를 실행하여 전체 스택(Nginx, React, Spring Boot, MySQL, Redis, Prometheus, Grafana)을 기동합니다.

### 1) 빌드 및 백그라운드 실행
```bash
# 전체 서비스 빌드 및 실행
docker compose up --build -d
```

### 2) 컨테이너 상태 확인
```bash
docker compose ps
```

### 3) 로그 실시간 모니터링
```bash
# 백엔드 로그 확인
docker compose logs -f backend

# Nginx 로그 확인
docker compose logs -f nginx
```

### 4) 서비스 중지 및 초기화
```bash
# 컨테이너 중지
docker compose down

# 볼륨(DB 데이터)까지 완전 초기화할 경우
docker compose down -v
```

---

## 4. 로컬 독립 개발 환경 실행 (Local Dev Mode)

Docker 없이 백엔드와 프론트엔드를 각각 로컬에서 직접 띄워 개발할 경우의 가이드입니다. (단, MySQL과 Redis는 Docker로 기동 필요)

### 1) 데이터베이스 & 레디스 기동
```bash
docker compose up -d mysql redis
```

### 2) Backend 로컬 실행 (Spring Boot)
```bash
cd backend

# Gradle 빌드 및 실행
./gradlew bootRun
```
* 로컬 API 서버: `http://localhost:8080`
* Actuator Health 체크: `http://localhost:8080/actuator/health`

### 3) Frontend 로컬 실행 (React + Vite)
```bash
cd frontend

# 의존성 패키지 설치
npm install

# Vite 개발 서버 실행
npm run dev
```
* 로컬 프론트엔드 서버: `http://localhost:5173` (또는 Vite 기본 포트)

---

## 5. 서비스 정상 구동 검증 (Health Check & Verification)

컨테이너가 정상적으로 실행되었는지 브라우저 또는 curl로 확인합니다.

| 서비스 | 접속 URL | 정상 응답 확인 기준 |
| :--- | :--- | :--- |
| **Web Application** | `http://localhost:80` | 메인 React SPA 화면 정상 출력 |
| **Backend API Health** | `http://localhost:8080/actuator/health` | `{"status":"UP"}` JSON 응답 |
| **Prometheus Metrics** | `http://localhost:8080/actuator/prometheus`| Micrometer JVM/HTTP 메트릭 텍스트 출력 |
| **Grafana Dashboard** | `http://localhost:3000` | 로그인 화면 (ID: admin / PW: 설정값) |
| **Prometheus UI** | `http://localhost:9090` | Prometheus 쿼리 콘솔 접속 |
