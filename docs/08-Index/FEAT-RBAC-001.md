


# FEAT-RBAC-001 APMS.SR IAM / Data Evidence Index

 # 1\. SA-1 의 각 md 문서

 | 저장소 | 로그 분류 코드 | 해당 연결 주소 |
| --- | --- | --- |
| SA-1 | [AC-003-rbac](https://github.com/bluejals13/SA-1/blob/main/docs/01-prd/acceptance-criteria/AC-003-rbac.md) | "\[AC-003-rbac\]"`(docs/01-prd/acceptance-criteria/AC-003-rbac.md)` |
| SA-1 | [PRD-FUNC-003-iam](https://github.com/bluejals13/SA-1/blob/main/docs/01-prd/functional/PRD-FUNC-003-iam.md) | "\[PRD-FUNC-003-iam\]"`(docs/01-prd/functional/PRD-FUNC-003-iam.md)` |
| SA-1 | [0006-rbac](https://github.com/bluejals13/SA-1/blob/main/docs/03-adr/0006-rbac.md) | "\[0006-rbac\]"`(docs/03-adr/0006-rbac.md)` |

# 2\. apms-sr 의 각 도메인 단위 분리 폴더

 | 저장소 | 브런치 | 도메인 분류 폴더 | 해당 연결 주소 |
| --- | --- | --- | --- |
| apms-sr | feature/auth@0603@1401 | [iam/admin/](<https://github.com/bluejals13/apms-sr/tree/feature/auth@0603@1401/backend/src/main/java/com/example/demo/iam/admin>) | "\[iam/admin/\]"`(https://github.com/bluejals13/apms-sr/tree/feature/auth@0603@1401/backend/src/main/java/com/example/demo/iam/admin)` |
| apms-sr | feature/auth@0603@1401 | [iam/menu/](<https://github.com/bluejals13/apms-sr/tree/feature/auth@0603@1401/backend/src/main/java/com/example/demo/iam/menu>) | "\[iam/menu/\]"`(https://github.com/bluejals13/apms-sr/tree/feature/auth@0603@1401/backend/src/main/java/com/example/demo/iam/menu)` |
| apms-sr | feature/auth@0603@1401 | [iam/role/](<https://github.com/bluejals13/apms-sr/tree/feature/auth@0603@1401/backend/src/main/java/com/example/demo/iam/role>) | "\[iam/menu/\]"`(https://github.com/bluejals13/apms-sr/tree/feature/auth@0603@1401/backend/src/main/java/com/example/demo/iam/role)` |
| apms-sr | feature/auth@0603@1401 | [iam/permission/](<https://github.com/bluejals13/apms-sr/tree/feature/auth@0603@1401/backend/src/main/java/com/example/demo/iam/permission>) | "\[iam/menu/\]"`(https://github.com/bluejals13/apms-sr/tree/feature/auth@0603@1401/backend/src/main/java/com/example/demo/iam/permission)` |
| apms-sr | feature/auth@0603@1401 | [iam/user/](<https://github.com/bluejals13/apms-sr/tree/feature/auth@0603@1401/backend/src/main/java/com/example/demo/iam/user>) | "\[iam/menu/\]"`(https://github.com/bluejals13/apms-sr/tree/feature/auth@0603@1401/backend/src/main/java/com/example/demo/iam/user)` |

# 3\. apms-sr 의 실제 진행되는 해당 테스트 들

 | 저장소 | 브런치 | 도메인 분류 폴더 | 해당 연결 주소 |
| --- | --- | --- | --- |
| apms-sr | feature/auth@0603@1401 | [permission/PermissionAdminServiceTest.java](<https://github.com/bluejals13/apms-sr/tree/feature/auth@0603@1401/backend/src/test/java/com/example/demo/iam/permission/PermissionAdminServiceTest.java>) | "\[permission/PermissionAdminServiceTest.java\]"`(https://github.com/bluejals13/apms-sr/tree/feature/auth@0603@1401/backend/src/test/java/com/example/demo/iam/permission/PermissionAdminServiceTest.java)` |
| apms-sr | feature/auth@0603@1401 | [permission/PermissionIntegrationTest.java](<https://github.com/bluejals13/apms-sr/tree/feature/auth@0603@1401/backend/src/test/java/com/example/demo/iam/permission/PermissionIntegrationTest.java>) | "\[permission/PermissionIntegrationTest.java\]"`(https://github.com/bluejals13/apms-sr/tree/feature/auth@0603@1401/backend/src/test/java/com/example/demo/iam/permission/PermissionIntegrationTest.java)` |
| apms-sr | feature/auth@0603@1401 | [permission/PermissionRepositoryTest.java](<https://github.com/bluejals13/apms-sr/tree/feature/auth@0603@1401/backend/src/test/java/com/example/demo/iam/permission/PermissionRepositoryTest.java>) | "\[permission/PermissionRepositoryTest.java\]"`(https://github.com/bluejals13/apms-sr/tree/feature/auth@0603@1401/backend/src/test/java/com/example/demo/iam/permission/PermissionRepositoryTest.java)` |
| apms-sr | feature/auth@0603@1401 | [permission/RbacSecurityIntegrationTest.java](<https://github.com/bluejals13/apms-sr/tree/feature/auth@0603@1401/backend/src/test/java/com/example/demo/iam/permission/RbacSecurityIntegrationTest.java>) | "\[permission/RbacSecurityIntegrationTest.java\]"`(https://github.com/bluejals13/apms-sr/tree/feature/auth@0603@1401/backend/src/test/java/com/example/demo/iam/permission/RbacSecurityIntegrationTest.java)` |

 | 저장소 | 브런치 | 도메인 분류 폴더 | 해당 연결 주소 |
| --- | --- | --- | --- |
| apms-sr | feature/auth@0603@1401 | [permission/RoleAdminServiceTest.java](<https://github.com/bluejals13/apms-sr/tree/feature/auth@0603@1401/backend/src/test/java/com/example/demo/iam/permission/RoleAdminServiceTest.java>) | "\[permission/RoleAdminServiceTest.java\]"`(https://github.com/bluejals13/apms-sr/tree/feature/auth@0603@1401/backend/src/test/java/com/example/demo/iam/permission/RoleAdminServiceTest.java)` |
| apms-sr | feature/auth@0603@1401 | [permission/RolePermissionIntegrationTest](<https://github.com/bluejals13/apms-sr/tree/feature/auth@0603@1401/backend/src/test/java/com/example/demo/iam/permission/RolePermissionIntegrationTest>) | "\[permission/RolePermissionIntegrationTest\]"`(https://github.com/bluejals13/apms-sr/tree/feature/auth@0603@1401/backend/src/test/java/com/example/demo/iam/permission/RolePermissionIntegrationTest)` |
| apms-sr | feature/auth@0603@1401 | [permission/RolePermissionServiceTest.java](<https://github.com/bluejals13/apms-sr/tree/feature/auth@0603@1401/backend/src/test/java/com/example/demo/iam/permission/RolePermissionServiceTest.java>) | "\[permission/RolePermissionServiceTest.java\]"`(https://github.com/bluejals13/apms-sr/tree/feature/auth@0603@1401/backend/src/test/java/com/example/demo/iam/permission/RolePermissionServiceTest.java)` |
| apms-sr | feature/auth@0603@1401 | [permission/RoleRepositoryTest.java](<https://github.com/bluejals13/apms-sr/tree/feature/auth@0603@1401/backend/src/test/java/com/example/demo/iam/permission/RoleRepositoryTest.java>) | "\[permission/RoleRepositoryTest.java\]"`(https://github.com/bluejals13/apms-sr/tree/feature/auth@0603@1401/backend/src/test/java/com/example/demo/iam/permission/RoleRepositoryTest.java)` |
| apms-sr | feature/auth@0603@1401 | [menu/MenuSecurityIntegrationTest.java](<https://github.com/bluejals13/apms-sr/tree/feature/auth@0603@1401/backend/src/test/java/com/example/demo/iam/menu/MenuSecurityIntegrationTest.java>) | "\[menu/MenuSecurityIntegrationTest.java\]"`(https://github.com/bluejals13/apms-sr/tree/feature/auth@0603@1401/backend/src/test/java/com/example/demo/iam/menu/MenuSecurityIntegrationTest.java)` |

# 4\. apms-sr 의 위 실제 작동 확인 검증 진행 중

 | 저장소 | 브런치 | 도메인 분류 폴더 | 해당 연결 주소 |
| --- | --- | --- | --- |
| apms-sr | feature/auth@0603@1401 | [verification/ADR-0A-RBAC](<https://github.com/bluejals13/apms-sr/tree/feature/auth%400603%401401/verification/ADR-0A-RBAC>) | "\[verification/ADR-0A-RBAC\]"`(https://github.com/bluejals13/apms-sr/tree/feature/auth%400603%401401/verification/ADR-0A-RBAC)` |

# 5\. PR-1A1 의 위 부분에 대한 ppt 혹은 발표 및 전달 자료 아직 계획 중

 | 저장소 | 도메인 분류 폴더 | 해당 연결 주소 |
| --- | --- | --- |
| SA-1 | [PRD-PO/claim](<https://github.com/bluejals13/PR-1A1/tree/main/PRD-PO/claim>) | "\[PRD-PO/claim\]"`(https://github.com/bluejals13/PR-1A1/tree/main/PRD-PO/claim)` |


