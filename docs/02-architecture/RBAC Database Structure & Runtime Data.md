

# RBAC Database Structure & Runtime Data

## 1. Overview

APMS.SR의 인증·인가 영역은 **User → Role → Permission** 구조를 기반으로 한다.

Role은 사용자의 권한을 묶어 관리하기 위한 단위이며, 실제 API 및 기능 접근 제어는 Role 자체가 아니라 **Permission을 Authority로 사용하여 수행한다.**

```text
User
  │
  │ User ↔ Role
  ▼
Role
  │
  │ Role ↔ Permission
  ▼
Permission
  │
  │ Authority
  ▼
API / Service
```

권한 변경 및 주요 관리 작업은 `audit_logs`를 통해 Runtime에서 기록한다.

---

# 2. Database Tables

| Table                   | 역할                   | 관계                                   |
| ----------------------- | -------------------- | ------------------------------------ |
| `users`                 | 사용자 계정 및 상태 관리       | `user_roles`와 연결                     |
| `roles`                 | 사용자 역할 정의            | `user_roles`, `role_permissions`와 연결 |
| `permissions`           | 시스템 기능 접근 권한 정의      | `role_permissions`와 연결               |
| `user_roles`            | User와 Role의 연결       | User ↔ Role                          |
| `role_permissions`      | Role과 Permission의 연결 | Role ↔ Permission                    |
| `menu`                  | 권한 기반 메뉴 관리          | Menu Permission과 연결                  |
| `audit_logs`            | 주요 관리/권한 변경 행위 기록    | User → Target                        |
| `flyway_schema_history` | DB Migration 이력 관리   | Flyway 관리                            |

> `flyway_schema_history` 및 Migration SQL 자체에 대한 상세 설명은 별도 DB Migration 문서에서 관리한다.

---

# 3. `users`

사용자 계정과 계정 상태를 관리한다.

## 3.1 Schema

| Column                | Type           | Null | Key    | Default  | Description |
| --------------------- | -------------- | ---- | ------ | -------- | ----------- |
| `id`                  | `bigint`       | NO   | PK     | `NULL`   | 사용자 식별자     |
| `password`            | `varchar(255)` | YES  |        | `NULL`   | 암호화된 비밀번호   |
| `username`            | `varchar(255)` | YES  | UNIQUE | `NULL`   | 사용자 로그인 식별자 |
| `password_changed_at` | `datetime(6)`  | YES  |        | `NULL`   | 비밀번호 변경 시각  |
| `status`              | `enum`         | NO   |        | `ACTIVE` | 사용자 계정 상태   |
| `email`               | `varchar(255)` | YES  |        | `NULL`   | 사용자 이메일     |

### `status`

| Value            | 의미            |
| ---------------- | ------------- |
| `ACTIVE`         | 정상적으로 활성화된 계정 |
| `SUSPENDED`      | 이용이 제한된 계정    |
| `DELETE_PENDING` | 삭제 대기 상태      |
| `DELETED`        | 삭제 처리된 계정     |

## 3.2 Runtime Data

| ID | Username | Status   | Email               |
| -: | -------- | -------- | ------------------- |
|  1 | `fpfns`  | `ACTIVE` | `testuser@test.com` |
|  2 | `test`   | `ACTIVE` | `admin@test.com`    |
|  4 | `blue`   | `ACTIVE` | `blue@gmail.com`    |

`password`는 Runtime 조회 결과 BCrypt 형태의 해시값으로 저장되어 있다.

---

# 4. `roles`

사용자에게 부여할 역할을 정의한다.

Role은 직접적인 API 접근 권한이라기보다 여러 Permission을 묶어 사용자에게 제공하는 **권한 관리 단위**로 사용한다.

## 4.1 Schema

| Column        | Type           | Null | Key    | Default | Description |
| ------------- | -------------- | ---- | ------ | ------- | ----------- |
| `id`          | `bigint`       | NO   | PK     | `NULL`  | Role 식별자    |
| `name`        | `varchar(255)` | NO   | UNIQUE | `NULL`  | Role 이름     |
| `description` | `varchar(255)` | YES  |        | `NULL`  | Role 설명     |
| `level`       | `int`          | NO   |        | `10`    | Role Level  |
| `is_system`   | `tinyint(1)`   | NO   |        | `0`     | 시스템 Role 여부 |

## 4.2 Runtime Data

| ID | Name    | Description        | Level | Is System |
| -: | ------- | ------------------ | ----: | --------: |
|  1 | `ADMIN` | `Administrator`    |    10 |         0 |
|  2 | `USER`  | `Normal User`      |    10 |         0 |
|  3 | Role 3  | Role 3 Description |    10 |         0 |

> Role 3의 실제 이름 및 설명은 Runtime 출력에서 인코딩 문제로 표시되지 않았으므로 문서에서는 식별 목적으로 `Role 3`으로 표기한다.

---

# 5. `permissions`

시스템에서 수행할 수 있는 세부 기능을 정의한다.

Permission은 실제 API 접근 제어에서 **Authority**로 사용된다.

## 5.1 Schema

| Column        | Type           | Null | Key    | Default | Description    |
| ------------- | -------------- | ---- | ------ | ------- | -------------- |
| `id`          | `bigint`       | NO   | PK     | `NULL`  | Permission 식별자 |
| `description` | `varchar(255)` | YES  |        | `NULL`  | Permission 설명  |
| `name`        | `varchar(100)` | NO   | UNIQUE | `NULL`  | Permission 이름  |

## 5.2 Permission List

현재 시스템에는 총 **14개의 Permission**이 존재한다.

| ID | Permission           | Domain            | Description         |
| -: | -------------------- | ----------------- | ------------------- |
|  1 | `USER_READ`          | User              | 사용자 조회              |
|  2 | `USER_STATUS_UPDATE` | User              | 사용자 상태 변경           |
|  3 | `USER_DELETE`        | User              | 사용자 삭제              |
|  4 | `ROLE_READ`          | Role              | Role 조회             |
|  5 | `ROLE_CREATE`        | Role              | Role 생성             |
|  6 | `ROLE_UPDATE`        | Role              | Role 수정             |
|  7 | `ROLE_DELETE`        | Role              | Role 삭제             |
|  8 | `ROLE_ASSIGN`        | Role / Permission | Role에 Permission 부여 |
|  9 | `MENU_READ`          | Menu              | Menu 조회             |
| 10 | `MENU_CREATE`        | Menu              | Menu 생성             |
| 11 | `MENU_UPDATE`        | Menu              | Menu 수정             |
| 12 | `MENU_DELETE`        | Menu              | Menu 삭제             |
| 13 | `PERMISSION_READ`    | Permission        | Permission 조회       |
| 14 | `USER_ROLE_MANAGE`   | User / Role       | 사용자 Role 관리         |

---

# 6. User ↔ Role

## `user_roles`

User와 Role의 관계를 관리한다.

하나의 User가 어떤 Role을 가지고 있는지를 연결 테이블을 통해 관리한다.

## 6.1 Schema

| Column    | Type     | Null | Key | Description |
| --------- | -------- | ---- | --- | ----------- |
| `user_id` | `bigint` | NO   | PK  | User ID     |
| `role_id` | `bigint` | NO   | PK  | Role ID     |

복합 Primary Key:

```text
(user_id, role_id)
```

## 6.2 Runtime Data

| User ID | Username | Role ID | Role    |
| ------: | -------- | ------: | ------- |
|       1 | `fpfns`  |       2 | `USER`  |
|       2 | `test`   |       1 | `ADMIN` |
|       4 | `blue`   |       3 | Role 3  |

### Current Relationship

```text
fpfns
 └── USER

test
 └── ADMIN

blue
 └── Role 3
```

사용자에게 Role을 부여하거나 변경하는 관리 작업은 `USER_ROLE_MANAGE` Authority를 통해 보호된다.

---

# 7. Role ↔ Permission

## `role_permissions`

Role과 Permission의 관계를 관리한다.

Role 자체를 API 접근 권한으로 사용하는 것이 아니라, Role에 연결된 Permission을 실제 Authority로 사용한다.

## 7.1 Schema

| Column          | Type     | Null | Key | Description   |
| --------------- | -------- | ---- | --- | ------------- |
| `role_id`       | `bigint` | NO   | PK  | Role ID       |
| `permission_id` | `bigint` | NO   | PK  | Permission ID |

복합 Primary Key:

```text
(role_id, permission_id)
```

## 7.2 Runtime Data

현재 총 20개의 Role-Permission 관계가 존재한다.

| Role    | Permission           |
| ------- | -------------------- |
| `ADMIN` | `USER_READ`          |
| `ADMIN` | `USER_STATUS_UPDATE` |
| `ADMIN` | `USER_DELETE`        |
| `ADMIN` | `ROLE_READ`          |
| `ADMIN` | `ROLE_CREATE`        |
| `ADMIN` | `ROLE_UPDATE`        |
| `ADMIN` | `ROLE_DELETE`        |
| `ADMIN` | `ROLE_ASSIGN`        |
| `ADMIN` | `MENU_READ`          |
| `ADMIN` | `MENU_CREATE`        |
| `ADMIN` | `MENU_UPDATE`        |
| `ADMIN` | `MENU_DELETE`        |
| `ADMIN` | `PERMISSION_READ`    |
| `ADMIN` | `USER_ROLE_MANAGE`   |
| `USER`  | `USER_READ`          |
| `USER`  | `MENU_READ`          |
| Role 3  | `USER_READ`          |
| Role 3  | `ROLE_READ`          |
| Role 3  | `MENU_READ`          |
| Role 3  | `PERMISSION_READ`    |

---

# 8. Current Role Permission Matrix

## ADMIN

| Permission           | Granted |
| -------------------- | ------- |
| `USER_READ`          | ✓       |
| `USER_STATUS_UPDATE` | ✓       |
| `USER_DELETE`        | ✓       |
| `ROLE_READ`          | ✓       |
| `ROLE_CREATE`        | ✓       |
| `ROLE_UPDATE`        | ✓       |
| `ROLE_DELETE`        | ✓       |
| `ROLE_ASSIGN`        | ✓       |
| `MENU_READ`          | ✓       |
| `MENU_CREATE`        | ✓       |
| `MENU_UPDATE`        | ✓       |
| `MENU_DELETE`        | ✓       |
| `PERMISSION_READ`    | ✓       |
| `USER_ROLE_MANAGE`   | ✓       |

## USER

| Permission    | Granted |
| ------------- | ------- |
| `USER_READ`   | ✓       |
| `MENU_READ`   | ✓       |
| 기타 Permission | -       |

## Role 3

| Permission        | Granted |
| ----------------- | ------- |
| `USER_READ`       | ✓       |
| `ROLE_READ`       | ✓       |
| `MENU_READ`       | ✓       |
| `PERMISSION_READ` | ✓       |

---

# 9. Permission Authority Model

현재 권한 검증의 핵심은 다음과 같다.

```text
Authentication
      │
      ▼
    User
      │
      ▼
    Role
      │
      ▼
 Permission
      │
      ▼
 Authority
      │
      ▼
 Controller / API
```

따라서 API 접근 제어는 다음과 같은 개념으로 이루어진다.

```text
Role 자체를 검사
        X

Permission Authority를 검사
        O
```

예를 들어 사용자 삭제 API의 접근 제어는 특정 Role 이름을 직접 검사하기보다 `USER_DELETE` Authority를 기준으로 판단한다.

이 구조를 통해 동일한 Permission을 여러 Role에 부여할 수 있으며, Role은 Permission 집합을 관리하는 단위로 동작한다.

---

# 10. Permission Management

Permission 자체도 일반 데이터가 아니라 권한 정책의 일부이므로 관리 기능 역시 Permission으로 보호된다.

## User → Role

사용자에게 Role을 부여하거나 변경한다.

```text
User
  │
  ▼
USER_ROLE_MANAGE
  │
  ▼
User ↔ Role
```

## Role → Permission

Role에 Permission을 부여하거나 변경한다.

```text
Role
  │
  ▼
ROLE_ASSIGN
  │
  ▼
Role ↔ Permission
```

따라서 권한 관리 구조는 다음과 같다.

```text
USER_ROLE_MANAGE
    │
    └── User → Role

ROLE_ASSIGN
    │
    └── Role → Permission
```

---

# 11. `menu`

Menu는 시스템 UI의 메뉴 정보를 관리하며, 현재 Permission 체계에는 Menu 관리를 위한 4개의 Permission이 존재한다.

| Permission    | 기능      |
| ------------- | ------- |
| `MENU_READ`   | Menu 조회 |
| `MENU_CREATE` | Menu 생성 |
| `MENU_UPDATE` | Menu 수정 |
| `MENU_DELETE` | Menu 삭제 |

현재 Runtime 조회에서는 Menu 데이터가 존재하지 않는다.

```text
SELECT * FROM menu;

Empty set
```

따라서 현재 Permission 구조에서는 Menu에 대한 CRUD 관리 권한이 정의되어 있으며, Menu 기능 자체는 해당 Permission 체계와 연결되는 구조로 동작한다.

현재 범위의 Menu 권한은 **Menu 전체에 대한 CRUD 수준의 권한 관리**를 중심으로 한다.

---

# 12. `audit_logs`

주요 사용자 및 권한 관리 작업의 변경 이력을 기록한다.

## 12.1 Runtime Structure

| Column         | Description   |
| -------------- | ------------- |
| `id`           | Audit Log 식별자 |
| `action`       | 수행된 작업        |
| `created_at`   | 작업 발생 시각      |
| `result`       | 작업 결과         |
| `target_id`    | 대상 객체 ID      |
| `target_type`  | 대상 객체 종류      |
| `user_id`      | 작업 수행 사용자     |
| `before_value` | 변경 전 값        |
| `after_value`  | 변경 후 값        |

## 12.2 Runtime Data

| ID | Action                   | Target | User | Before           | After                                              |
| -: | ------------------------ | ------ | ---: | ---------------- | -------------------------------------------------- |
|  1 | `USER_STATUS_CHANGE`     | User 3 |    2 | `ACTIVE`         | `DELETE_PENDING`                                   |
|  2 | `USER_DELETE`            | User 3 |    2 | `DELETE_PENDING` | `DELETED`                                          |
|  3 | `ROLE_CREATE`            | Role 3 |    2 | `NULL`           | Role 생성 정보                                         |
|  4 | `ROLE_PERMISSION_MANAGE` | Role 3 |    2 | `[]`             | `MENU_READ, PERMISSION_READ, ROLE_READ, USER_READ` |
|  5 | `USER_ROLE_UPDATE`       | User 4 |    2 | `[]`             | Role 3                                             |

## 12.3 Audit Flow

### User Status Change

```text
ACTIVE
  ↓
USER_STATUS_CHANGE
  ↓
DELETE_PENDING
```

### User Delete

```text
DELETE_PENDING
  ↓
USER_DELETE
  ↓
DELETED
```

### Role Permission Assignment

```text
Role 3
  ↓
ROLE_PERMISSION_MANAGE
  ↓
MENU_READ
PERMISSION_READ
ROLE_READ
USER_READ
```

### User Role Assignment

```text
User 4
  ↓
USER_ROLE_UPDATE
  ↓
Role 3
```

Audit Log를 통해 단순히 현재 DB 상태뿐 아니라 **권한 및 사용자 관리 작업이 실제로 수행된 이력**을 확인할 수 있다.

---

# 13. Overall ER Relationship

현재 RBAC 영역의 전체적인 관계는 다음과 같다.

```text
┌──────────────┐
│    users     │
├──────────────┤
│ id           │
│ username     │
│ password     │
│ status       │
│ email        │
└──────┬───────┘
       │
       │
       ▼
┌──────────────┐
│  user_roles  │
├──────────────┤
│ user_id      │
│ role_id      │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│    roles     │
├──────────────┤
│ id           │
│ name         │
│ description  │
│ level        │
│ is_system    │
└──────┬───────┘
       │
       ▼
┌──────────────────┐
│ role_permissions │
├──────────────────┤
│ role_id          │
│ permission_id    │
└──────┬───────────┘
       │
       ▼
┌──────────────────┐
│   permissions    │
├──────────────────┤
│ id               │
│ name             │
│ description      │
└──────────────────┘
```

Audit Log는 주요 변경 작업을 별도로 기록한다.

```text
User / Role / Permission
          │
          ▼
     audit_logs
```

---

# 14. Complete Authorization Flow

전체 서비스 관점에서는 다음과 같이 연결된다.

```text
HTTP Request
      │
      ▼
Authentication
      │
      ▼
Authenticated User
      │
      ▼
User → Role
      │
      ▼
Role → Permission
      │
      ▼
Permission Authority
      │
      ▼
Controller
      │
      ▼
Service
      │
      ▼
Business Operation
      │
      ├──────────────► DB
      │
      └──────────────► Audit Log
```

권한 변경 자체도 다시 권한으로 보호된다.

```text
User
 │
 ├── USER_ROLE_MANAGE ──► User → Role
 │
 └── ROLE_ASSIGN ───────► Role → Permission
```

---

# 15. Runtime Data Summary

| Category                  | Current Data |
| ------------------------- | -----------: |
| Users                     |            3 |
| Roles                     |            3 |
| Permissions               |           14 |
| User-Role Relations       |            3 |
| Role-Permission Relations |           20 |
| Menu Records              |            0 |
| Audit Logs                |            5 |

### Users

```text
fpfns → USER
test  → ADMIN
blue  → Role 3
```

### Roles

```text
ADMIN
USER
Role 3
```

### Permissions

```text
14 Permissions
```

### Role-Permission

```text
ADMIN → 14 Permissions
USER  → 2 Permissions
Role 3 → 4 Permissions
```

---

# 16. Data Model Summary

현재 APMS.SR의 권한 모델은 다음 세 가지 계층으로 정리할 수 있다.

| Layer     | Entity        | 역할               |
| --------- | ------------- | ---------------- |
| Identity  | `users`       | 누가 요청하는가         |
| Role      | `roles`       | 어떤 권한 집합을 갖는가    |
| Authority | `permissions` | 어떤 기능을 수행할 수 있는가 |

관계 테이블은 각 계층을 연결한다.

```text
users
  │
  └── user_roles
          │
          ▼
        roles
          │
          └── role_permissions
                  │
                  ▼
             permissions
```

실제 서비스 접근 제어에서는 최종적으로 Permission이 Authority로 사용된다.

---

# 17. Current Scope

현재 구현 범위에서 권한 모델은 다음을 제공한다.

* 사용자 계정 및 상태 관리
* User ↔ Role 관계 관리
* Role ↔ Permission 관계 관리
* Permission 기반 API Authority 검증
* User에게 Role을 부여하는 권한 관리
* Role에 Permission을 부여하는 권한 관리
* Menu CRUD 권한 관리
* 주요 관리 작업의 Audit Log 기록
* User 상태 변경 및 삭제 lifecycle 기록

현재 모델은 **Role을 직접 API 권한으로 판단하기보다 Permission을 실제 Authority로 사용하는 구조**를 중심으로 한다.

Migration SQL 자체의 설계 및 변경 이력은 별도의 Flyway 문서에서 관리한다.

이 문서는 지금 단계에서는 **"DB가 왜 이렇게 설계됐는가"보다 "현재 서비스의 DB 모델이 무엇이며 실제 Runtime에서 어떤 상태인가"**에 초점을 맞추는 게 맞아.

그리고 중요한 건 내가 여기서 `User는 1개의 Role만 가진다` 같은 **확인되지 않은 정책을 임의로 문서에 넣지 않았다는 것**이야. 현재 실제 `user_roles` 데이터는 3건이고, 구조상 User–Role 관계를 별도 테이블로 관리한다는 사실까지만 적었다. 이 정도가 지금 자료로 가장 정확한 문서화다.

[1]: https://github.com/bluejals13/apms-sr/blob/feature/auth%400603%401401/backend/src/main/resources/db/migration/V1__init_schema.sql "apms-sr/backend/src/main/resources/db/migration/V1__init_schema.sql at feature/auth@0603@1401 · bluejals13/apms-sr · GitHub"
