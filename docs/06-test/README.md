# 06-ai-engineering (AI Collaboration & Engineering Process)

본 디렉토리는 시스템 개발 과정에서 AI 에이전트(LLM)와 협업한 **엔지니어링 프로세스, 상호작용 사이클, 그리고 AI 오류 및 반박·교정 사례(Failure Cases)**를 관리하는 공간입니다.

> **운영 핵심 원칙:**  
> "AI가 처음 제안한 설계의 결함을 사람이 발견하고, 반박하여 설계를 바로잡은 **실패·교정 사례(Failure Case) 하나가 수십 개의 단순 코드 생성 기록보다 가치가 있습니다.**"  
> AI의 결과를 맹신하지 않고, 인간 개발자의 비판적 검증(Critique)과 실증(Verification)을 거치는 엔지니어링 과정을 자산화합니다.

---

## 1. 문서 계층 간 책임 경계 (Separation of Concerns)

`06-ai-engineering`은 타 문서 계층과 다음과 같이 역할을 명확히 분리합니다:

| 문서 계층 | 주요 역할 | 06-ai-engineering과의 차이점 |
| :--- | :--- | :--- |
| **`01~03` (PRD/Arch/ADR)** | 시스템 스펙, 구조, 아키텍처 결정 | 시스템 자체의 산출물이며, AI와의 대화/교정 과정을 기록하지 않음 |
| **`04-task-progress`** | 작업 단위(TASK) 실행 및 테스트 증거 | 작업 완료 여부를 추적하며, 프롬프트 엔지니어링 자체를 다루지 않음 |
| **`05-knowledge`** | 시스템 버그 트러블슈팅 및 기술 지식 | **코드/인프라 자체의 결함**을 다루며, AI의 설계 환각을 다루지 않음 |
| **`06-ai-engineering`** | **AI 협업 프로세스 및 Failure Cases** | **AI 에이전트의 오답/환각 제안을 사람이 논리적으로 반박하여 올바른 설계로 유도한 엔지니어링 과정의 단일 저장소** |
| **`07-operations`** | 시스템 기동, 런타임 환경, 운영 가이드 | 런타임 환경을 다루며, AI 협업을 다루지 않음 |
| **`conventions`** | 코딩 표준 및 에이전트 표준 명령어 | 표준 프롬프트 문구(`04_Agent_Commands.md`)는 `conventions/`의 단일 소스에서 관리 |

---

## 2. AI 협업 표준 사이클

AI 에이전트와의 협업은 다음 4단계 검증 루프를 엄격히 준수합니다:

```text
[ 1. Prompt ] ────► [ 2. AI Generation ]
                            │
                            ▼
                    [ 3. Human Critique & Refutation ] ──(반박/설계 수정)──┐
                            │                                             │
                            ▼ (검증 통과 시)                               │
                    [ 4. Grounding & Verification ]                       ▼
                            │                                     [ failure-cases/ ]
                            ▼                                     (오류/반박 사례 문서화)
                    (실제 코드/테스트 반영)
```

1. **Prompt (지시):** `conventions/04_Agent_Commands.md` 표준 프롬프트 규약에 따라 컨텍스트와 목표를 명확히 제시.
2. **AI Generation (제안):** AI가 코드, 아키텍처, 또는 작업 계획을 생성.
3. **Human Critique & Refutation (비판적 검증):** 사람이 AI의 제안을 정밀 검토하여 환각(Hallucination), 과도한 복잡성(Over-engineering), 또는 보안 취약점을 발견 시 논리적으로 반박.
4. **Grounding & Verification (실증):** 반박을 통해 수정된 최종 설계를 실제 테스트와 Evidence로 증명.

---

## 3. `failure-cases/` 작성 기준 및 템플릿

AI가 잘못된 설계를 제안하여 사람이 이를 논리적으로 지적하고 설계를 전면 변경한 유의미한 사례가 발생할 때, `failure-cases/FC-{번호}-{주제}.md` 파일로 작성합니다.

```markdown
# [Failure Case] FC-001: {사례 제목}

- **일자:** YYYY-MM-DD
- **관련 영역:** Security / Architecture / Concurrency
- **관련 TASK / ADR:** {TASK-ID 또는 ADR-ID}

### 1. AI Initial Proposal (AI의 초기 제안)
* AI가 최초로 제시한 접근 방식이나 코드 구조 요약

### 2. Flaw Detection (문제점 발견)
* 해당 제안에 내포된 기술적 결함, 보안 취약점, 레이스 컨디션, 또는 잘못된 가정

### 3. Human Refutation & Counter-argument (사람의 반박 및 대안 제시)
* 개발자가 AI에게 제시한 반박 논리와 정정 요구 사항

### 4. Revised Solution & Decision (수정된 최종 결정)
* 반박 후 AI와 함께 재도출한 올바른 구조 및 최종 구현/검증 결과

### 5. Lessons Learned (교훈)
* 향후 유사 프롬프트나 설계 시 주의해야 할 점
```
