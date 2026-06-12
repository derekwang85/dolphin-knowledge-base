# ADR-010: Flyway DB 迁移策略

> **状态**: 🟢 Accepted | **日期**: 2026-06-12 | **关联**: ADR-003

---

## 背景

TradeOMS 使用 Flyway 管理数据库 Schema 版本。随着模块增多（KYC → CDP → Document → Quote → Order → VA → TT → LC），需要制定统一的迁移策略来保证数据库 Schema 的兼容性、可追溯性和性能。

核心挑战：
- 17 个迁移脚本需合理编排，避免冲突
- 批量迁移 vs 单次迁移的选择
- 索引策略 — 何时加、如何加
- 历史表/审计表的 append-only 约束
- 可重复迁移 vs 版本化迁移的选择

## 决策

### 1. 版本化迁移（Versioned）为主，可重复迁移（Repeatable）为辅

| 类型 | 用途 | 例子 |
|------|------|------|
| `V{version}__{description}.sql` | 所有 DDL/DML 变更 | `V1__create_kyc_tables.sql` |
| `R__{description}.sql` | 已废弃视图兼容（如旧版 contract_log） | `R__contract_log_deprecated.sql` |

**规则**：版本化迁移永不修改已发布的迁移内容。如需回退，创建新的 V+1 迁移进行逆向操作或标记废弃。

### 2. 批量迁移模式（非增量单表）

**决策**：按模块批量创建迁移，而非每个表一个迁移。

```
V1  → KYC 模块（buyer + kyc_application）
V2  → CDP 模块（company + department + user + region + role）
V3  → Document 模块（document + attachment）
V4  → Quote 模块（quote + price_book + product + incoterm）
V5  → Order 模块（order + contract + business_event）
V6  → VA 模块（va_account + va_transaction）
V7  → TT 模块（tt_payment + payment_voucher）
V8  → LC 模块（lc + lc_collaboration）
```

**理由**：
- 模块间依赖清晰（V1 → V2 → V3 → V4 → V5 → V6 → V7 → V8）
- 开发新模块时只需增加 V+1 迁移，不影响历史
- CI 环境可一次性快速重建所有表

### 3. 索引策略 — 两波分离

**第一波 (V9) — 性能驱动**：在 Batch 10 API 性能检查后，为高频查询添加缺失索引。

| 索引 | 目标 |
|------|------|
| `idx_fact_section(fact_id, section_code)` | KYC 章节查询 |
| `idx_expiry_date(expiry_date)` | LC 过期批量查询 |
| `idx_entity_status(entity_type, entity_id, status)` | 附件查询 |
| `idx_account_created(account_id, created_at)` | VA 交易流水 |
| `idx_updated_at(updated_at)` | quote/order 列表排序 |
| `idx_created_at(created_at)` | TT 付款列表排序 |
| `idx_payment_status(payment_id, voucher_status)` | 付款凭证查询 |

**第二波 (V11) — 审计全覆盖**：在 V10 软删除统一后，对所有 V1-V10 表做索引审计，补充遗漏：

- 所有事实表的 `updated_at` 排序索引
- 所有字典表/状态表的 `status` 过滤索引
- 所有复合 JOIN 字段的复合索引（`idx_fact_detail_type`, `idx_fact_role`）
- 所有历史表的 `history_at` 时间范围查询索引

### 4. 软删除统一 (V10)

**决策**：所有事实表统一添加 `is_deleted TINYINT(1) DEFAULT 0` 字段。

**适用范围**：仅事实表（order, buyer, quote, product, company, va_account, tt_payment, lc 等）

**不适用范围**：历史表、审计日志表、EAV 扩展字段（append-only，不需要软删除）

**实现方式**：ALTER TABLE ADD COLUMN，而非重建表。

### 5. 种子数据分离 (V13-V15)

业务数据（参考数据、演示数据、字段修正）放在 V13 之后，与 Schema 迁移分离，便于环境和数据独立管理。

| 迁移 | 内容 |
|------|------|
| V13 | `seed_reference_data.sql` — region/incoterm/currency 等参考数据 |
| V14 | `seed_business_data.sql` — 演示用业务数据 |
| V15 | `add_local_amount_column.sql` — 字段追加修正 |

## 影响

- **正向**：模块化清晰，CI 可全量重建，索引审计可复用
- **约束**：不可修改已有迁移，所有变更必须新迁移
- **代价**：本地开发重跑 Flyway clean + migrate 需要较长时间（V1-V15 全部执行）
- **重复迁移** 仅用于已废弃功能的视图兼容，不用于业务逻辑变更
