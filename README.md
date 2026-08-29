# 📚 SA-1: Technical Knowledge & Architecture Repository

> **APMS.SR 시스템의 기술 지식, 아키텍처 의사결정, 개발 컨벤션 및 변경 이력(Changelog) 관리 저장소**

본 저장소(`SA-1`)는 `26-05adf`(실제 구현 저장소)의 개발 과정에서 도출된 **기술적 의사결정의 배경(Why), 아키텍처 구조, 변경 이력 및 AI 에이전트 거버넌스 규칙**을 지식 자산으로 체계화하여 보관하는 지식 계층(Knowledge Layer) 저장소입니다.

---

## 1. 저장소 역할 및 책임 (Role & Responsibilities)

- **Architecture:** 시스템 토폴로지, 보안 계층 설계, 포트 매핑 및 데이터 흐름 명세 (`architecture/`)
- **Changelogs:** 각 개발 Phase별 변경 내역 및 기술 도입 배경/대안 비교 기록 (`changelogs/`)
  - Phase 1: Backend DTO, JWT/Redis, Global Exception, JPA N+1 최적화
  - Phase 2: Frontend Zustand Auth, Single Flight 재발급 최적화
  - Phase 3: Infra 안정화 (진행 예정)
- **Conventions:** 백엔드/프론트엔드 코딩 규칙, Zero-Chatter 거버넌스, Agent 커맨드 규격 (`conventions/`)
- **PKM & Infra:** 시스템 운영 지식, Grafana 대시보드 템플릿 및 인프라 지식 (`pkm&infra/`)

---

## 2. 3-Repository 데이터 흐름 내 위치

```text
[ 26-05adf ]  ── (구현 팩트 및 실행 결과) ──►  [ SA-1 (현재 저장소) ]
 실제 소스/테스트/인프라                               기술 지식 / 아키텍처 / Changelog
                                                            │
                                                            ▼ (지식 구조화)
                                                   [ PR-1A1 ]
                                                    포트폴리오 / 템플릿 / HTML
```

---

## 3. 디렉터리 구성

```
SA-1/
├── architecture/               # 시스템 아키텍처 및 포트 구성 명세
│   ├── 01_Architecture_and_Ports.md
│   └── 02_Quick_Start.md
├── changelogs/                 # Phase별 엔지니어링 변경 로그 (Why & Trade-off 중심)
│   ├── phase1_backend/         # Phase 1 백엔드 최적화 로그 (1-1 ~ 1-5)
│   ├── phase2_frontend/        # Phase 2 프론트엔드 상태/인증 로그 (2-1)
│   └── phase3_infra/           # Phase 3 인프라 안정화 로그 (예정)
├── conventions/                # 코딩 및 AI 에이전트 협업 규약
│   ├── 03_Backend_Conventions.md
│   ├── 04_Agent_Commands.md
│   └── rules.md
└── pkm&infra/                  # 개인 지식 관리(PKM) 및 인프라 대시보드 자산
    ├── PKM/
    └── infra/                  # Grafana 대시보드 JSON 등
```

---

## 4. 원칙 (Governance)

1. **Why 중심 기록:** 단순히 "코드가 있다"가 아니라 "왜 이 기술/패턴을 선택했고 어떤 문제를 해결했는가"를 기록합니다.
2. **Source of Truth 준수:** 실제 구현 및 런타임의 사실(Fact)은 `26-05adf`를 기준으로 하며, 없는 기능을 추측하여 기록하지 않습니다.
3. **코드 중복 방지:** 전체 소스코드를 복사하지 않고, 핵심 스니펫과 아키텍처 설계 배경 위주로 작성합니다.
