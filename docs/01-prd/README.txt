
# 3. 01-prd/ ← 가장 중요
```txt
여기가 요구사항의 원본이야.

01-prd/
├─ README.md
├─ functional/
├─ non-functional/
├─ security/
└─ acceptance-criteria/

예를 들어:

01-prd/
├─ functional/
│  ├─ PRD-FUNC-001-authentication.md
│  ├─ PRD-FUNC-002-user.md
│  └─ PRD-FUNC-003-iam.md
│
├─ non-functional/
│  ├─ PRD-NFR-001-performance.md
│  └─ PRD-NFR-002-availability.md
│
├─ security/
│  ├─ PRD-SEC-001-jwt.md
│  ├─ PRD-SEC-002-refresh-token.md
│  ├─ PRD-SEC-003-rbac.md
│  └─ PRD-SEC-004-redis-failure.md
│
└─ acceptance-criteria/
   ├─ AC-001-authentication.md
   ├─ AC-002-refresh-token.md
   └─ AC-003-rbac.md

여기서 중요한 건:

PRD에는 구현 방법을 너무 빨리 적지 않는다.

예를 들어:

❌

Redis를 사용해서 Refresh Token을 저장한다.

이건 설계 결정이야.

PRD에서는:

동일 Refresh Token의 재사용을 방지해야 한다.
동시 요청 상황에서도 보안 정책을 만족해야 한다.

정도가 먼저야.
```

