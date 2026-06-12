# ADR-003: 数据库设计原则

> **状态**: 已接受 | **日期**: 2026-06-09

---

## 背景

原方案采用 OntOS 本体建模（RDF/OWL/Neo4j），但实际落地需要关系型数据库。

## 决策

**采用以下六项数据库设计原则**：

### 原则 1: 三表结构

每个业务实体由三张表组成：

| 表 | 用途 | 示例 oms_quote |
|----|------|---------------|
| 事实表 (fact) | 当前有效状态 | id, quote_no, status, price, buyer_id... |
| 明细表 (detail) | 一对多行项目 | fact_id, field_name, field_value... |
| 历史表 (history) | 全量变更审计 | history_type, snapshot_of_all_fact_fields |

### 原则 2: 深度仅两层

禁止孙表（主表→明细表→子明细表）。遇到三层需求：
- 用 JSON 字段存储结构化小数据
- 或用独立实体 + 明细表二级关联

### 原则 3: 无存储过程/触发器

所有业务逻辑在 Java Service 层实现。数据库只负责：
- 存储
- 索引
- 完整性约束（主键/唯一键/外键定义）

### 原则 4: 历史表全量镜像

历史表不存 diff，而是存储变更时的全部字段快照。这样：
- 无需回溯计算历史版本
- DELETE 操作可追溯
- 一条 SQL 即可还原任意时间点的数据

### 原则 5: 枚举用 VARCHAR

所有状态/类型字段使用 VARCHAR，配合 Java Enum：
- 数据库不约束枚举值（避免频繁 DDL）
- Java 层校验和转换
- `oms_data_dictionary` 表作为枚举参考

### 原则 6: 计算列仅限纯算术

只允许 MySQL `GENERATED ALWAYS AS (...)` 做纯四则运算（如 `available_balance = total_limit - used_amount`），不做 IF/ELSE/业务判断。

## 影响

- 每套实体 3 张表带来约 40% 的额外存储（历史表）
- 但换来：无需审计模块、回滚天生支持、调试成本降低
- 三表模式在 MyBatis Plus 中通过通用 Service 模板实现，非重复劳动
