
# 5. 03-adr/
```txt
여기가 네가 말한 ADR.

03-adr/
├─ README.md
├─ 0001-architecture.md
├─ 0002-database.md
├─ 0003-redis.md
├─ 0004-jwt.md
├─ 0005-refresh-token-rotation.md
├─ 0006-rbac.md
└─ ...

ADR 하나는 결정 하나라고 생각해.

예를 들어:

0005-refresh-token-rotation.md

내용은 대략:

# ADR-0005 Refresh Token Rotation

## Status
Accepted

## Context

Refresh Token Replay 공격을 방어해야 한다.

## Options

1. Stateless JWT
2. DB 저장
3. Redis 저장
4. Redis + Rotation

## Decision

Redis + Refresh Token Rotation

## Consequences

장점:
...

단점:
...

## Related

PRD:
PRD-SEC-002

Task:
TASK-SEC-014

ADR에 모든 기술 지식을 넣지 마.

"왜 이 결정을 했는가"만 남겨.
```

