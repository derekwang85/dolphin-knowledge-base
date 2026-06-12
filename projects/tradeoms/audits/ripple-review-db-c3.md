# 🌊 涟漪分析: C.3.x Flyway 数据库迁移

> **日期**: 2026-06-12 | **WBS**: C.3.01~C.3.10 | **Agent**: 🐶 斗牛

---

## 一、变更总览

| WBS | 变更 | 类型 | 影响模块 |
|:---:|------|:----:|---------|
| C.3.01 | Flyway 启用 + Baseline V8 | 配置 | 全局启动 |
| C.3.02~3.08 | V1~V8 审核 (已有,baseline) | DDL | 全部 8 个模块 |
| C.3.09 | V9 性能索引 / V10 软删除 / V11 缺失索引 | DDL+索引 | 全部模块 |
| C.3.10 | V12 审计日志表兼容修复 | DDL+索引 | 审计模块 |

---

## 二、基线分析 (Baseline V8)

### 2.1 已有表 (V1~V8, 来自 init schema)

| 模块 | 表数量 | 来源文件 | 状态 |
|:----:|:------:|---------|:----:|
| KYC | 14 | V1__create_kyc_tables.sql | ✅ Baseline |
| CDP | 20 | V2__create_cdp_tables.sql | ✅ Baseline |
| Document | 7 | V3__create_document_tables.sql | ✅ Baseline |
| Quote | 14 | V4__create_quote_tables.sql | ✅ Baseline |
| Order | 6 | V5__create_order_tables.sql | ✅ Baseline |
| VA | 8 | V6__create_va_tables.sql | ✅ Baseline |
| TT | 9 | V7__create_tt_tables.sql | ✅ Baseline |
| LC | 11 | V8__create_lc_tables.sql | ✅ Baseline |

### 2.2 基线前提

Docker init (`03-database/oms-database-schema.sql`) 已创建 V1~V8 全部表结构。
Flyway baseline-version=8 标记这些表为"基线"，V9+ 才会被 Flyway 管理。

---

## 三、V9~V12 涟漪分析

### 3.1 V9: 性能索引

| 索引 | 表 | 类型 | 影响 |
|:----:|:--|:----:|:----:|
| `idx_fact_section` | `oms_kyc_application_detail` | 复合 | KYC 章节查询加速 |
| `idx_expiry_date` | `oms_lc` | 单列 | LC 过期批量扫描 |
| `idx_entity_status` | `oms_attachment` | 复合 | 附件实体查询(含状态) |
| `idx_account_created` | `oms_va_transaction` | 复合 | VA 交易流水查询排序 |
| `idx_updated_at` | `oms_quote` | 单列 | 报价列表排序 |
| `idx_updated_at` | `oms_order` | 单列 | 订单列表排序 |
| `idx_created_at` | `oms_tt_payment` | 单列 | 付款单列表排序 |
| `idx_payment_status` | `oms_payment_voucher` | 复合 | 付款凭证状态查询 |

**影响**: ✅ 仅索引新增，无结构变更。SQL EXPLAIN 计划改善。

### 3.2 V10: 软删除字段

| 模块 | 受影响表 | 影响类型 |
|:----:|:--------:|:--------:|
| Order | `oms_order`, `oms_order_detail`, `oms_contract`, `oms_business_event` | 新增列 |
| KYC | `oms_buyer`, `oms_buyer_detail`, `oms_kyc_application`, `oms_kyc_application_detail` | 新增列 |
| Quote | `oms_quote`, `oms_quote_detail`, `oms_quote_negotiation`, `oms_product`, `oms_product_detail`, `oms_product_category`, `oms_price_book`, `oms_price_book_detail`, `oms_incoterm`, `oms_incoterm_detail` | 新增列 |
| CDP | `oms_company`, `oms_company_detail`, `oms_department`, `oms_department_detail`, `oms_user`, `oms_user_detail`, `oms_region`, `oms_region_detail`, `oms_role`, `oms_data_dictionary` | 新增列 |
| Document | `oms_transaction_document`, `oms_attachment`, `oms_attachment_detail` | 新增列 |
| VA | `oms_va_account`, `oms_va_account_detail`, `oms_va_transaction`, `oms_va_transaction_detail` | 新增列 |
| TT | `oms_tt_payment`, `oms_tt_payment_detail`, `oms_payment_voucher`, `oms_payment_voucher_detail` | 新增列 |
| LC | `oms_lc`, `oms_lc_detail`, `oms_lc_collaboration`, `oms_lc_collaboration_detail` | 新增列 |

**影响**: 🔴 高 — 所有 Mapper XML 和 MyBatis-Plus 实体类的 `isDeleted` 字段必须对齐。
- MyBatis-Plus 已配置 `logic-delete-field: isDeleted` (见 application.yml)
- 所有查询会自动过滤 `is_deleted=0`
- 需检查: 自定义 SQL (非 MyBatis-Plus) 是否需要加 `AND is_deleted=0`

### 3.3 V11: 缺失索引

| 类别 | 索引数量 | 影响表 |
|:----:|:--------:|:------:|
| `updated_at` 排序索引 | 10 | `oms_buyer`, `oms_company`, `oms_department`, `oms_user`, `oms_product`, `oms_price_book`, `oms_va_account`, `oms_lc`, `oms_tt_payment`, `oms_transaction_document` |
| `status` 过滤索引 | 6 | `oms_department`, `oms_region`, `oms_role`, `oms_data_dictionary`, `oms_product_category`, `oms_incoterm` |
| JOIN 复合索引 | 4 | `oms_kyc_application(company_id)`, `oms_buyer_detail(fact_id,detail_type)`, `oms_company_detail(fact_id,detail_type)`, `oms_user_detail(fact_id,role_code)` |
| 历史表时间索引 | 16 | 全部 16 张历史表的 `history_at` |

**影响**: ✅ 仅索引新增。`ORDER BY updated_at DESC` 的列表查询消除文件排序。

### 3.4 V12: 审计日志兼容

| 变更 | 说明 |
|:----:|:----:|
| `CREATE TABLE IF NOT EXISTS` | 兼容 init schema 已建表场景 |
| `idx_result` 条件新建 | 仅当索引不存在时才 ALTER TABLE |

**影响**: ✅ 幂等兼容。`oms_audit_log` append-only，无历史表。

---

## 四、代码涟漪 (C.1.x 安全底座关联)

C.1.x 安全底座已实现，依赖用户表 (`oms_user`)：

| 文件 | 依赖 | 影响 |
|:----:|:----:|:----:|
| `UserDetailsServiceImpl.java` | `userMapper.findByUsername()` | 读 `oms_user.user_id`, `password_hash`, `user_type`, `status` |
| `JwtTokenProvider.java` | 无 DB 依赖 | 纯 JWT 逻辑 |
| `JwtTokenStore.java` | Redis `jwt:*` | 无 DB 表依赖 |
| `SecurityConfig.java` | 配置 | 无 DB 表依赖 |
| `AuthService.java` | `UserMapper` + `PasswordEncoder` | 读 `oms_user` |

**结论**: C.1.x 无新增 DB 表依赖，V10 软删除不影响认证逻辑 (`oms_user` 已有 `is_deleted` 但认证时需精准查账号)。

---

## 五、回滚方案

| 场景 | 操作 |
|:----:|:----:|
| Flyway 迁移失败 | 修复 SQL → `mvn flyway:repair` → 重跑 |
| V10 软删除需回退 | `ALTER TABLE xxx DROP COLUMN is_deleted` (逐表) |
| 索引需回退 | `ALTER TABLE xxx DROP INDEX idx_xxx` (逐索引) |
| V12 兼容问题 | 回退至旧版 V12 + `flyway:repair` |

---

## 六、验证清单

- [x] V1~V12 SQL 语法检查
- [x] V12 `IF NOT EXISTS` 幂等修复
- [x] V12 `idx_result` 条件索引兼容
- [x] Flyway baseline-version=8 (匹配 init schema)
- [x] MyBatis-Plus 软删除配置已对齐 `is_deleted`
- [x] C.1.x 安全底座无冲突依赖
- [ ] 后端重启后 Flyway 自动执行 V9~V12
- [ ] 验证 `flyway_schema_history` 表正确记录
- [ ] curl 测试 auth 端点: `POST /api/v1/auth/login`

---

*生成: 2026-06-12 13:03 | 签名: 🐶 斗牛*
