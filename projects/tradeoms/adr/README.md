# ADR 索引

> 架构决策记录索引，追踪每个 ADR 的状态、关联关系和落地进度。

---

## ADR 状态定义

| 状态 | 含义 |
|------|------|
| 🟢 Accepted | 决策已接受，实施中或已完成 |
| 🟡 Proposed | 提案中，待评审 |
| 🔴 Deprecated | 已被替代或废弃 |

---

## ADR 一览

| # | 标题 | 状态 | 关联 ADR | 实施进度 | 关键决策 |
|---|------|------|---------|---------|---------|
| ADR-001 | 双轨并行架构 (Dual-track) | 🟢 Accepted | — | ✅ 已完成 | Track A (Web) + Track B (OpenClaw) 共享同一后端和数据库 |
| ADR-002 | 技术栈选型 | 🟢 Accepted | ADR-001 | ✅ 已完成 | Java 17 + Spring Boot 3 + MySQL + Redis + Qdrant + MinIO |
| ADR-003 | 数据库设计 | 🟢 Accepted | ADR-002 | ✅ 已完成 | 3 张核心表（订单/报价/客户）+ 扩展字段 |
| ADR-004 | OpenClaw 集成 | 🟢 Accepted | ADR-001 | ✅ 已完成 | Agent 通过标准 API 接入，sourceChannel=OPENCLAW |
| ADR-005 | 国际化 (i18n) | 🟢 Accepted | ADR-002 | ✅ 已完成 | 中英双语 Buyer/运营端，6 道门禁 |
| ADR-006 | 涟漪分析机制 | 🟢 Accepted | — | ✅ 已完成 | 元方法论：新增规约时的系统性检查 |
| ADR-010 | Flyway DB 迁移策略 | 🟢 Accepted | ADR-003 | ✅ 已完成 | 17迁移+批量模式+两波索引策略 |
| ADR-011 | JWT 认证架构 | 🟢 Accepted | ADR-002 | ✅ 已完成 | HMAC-SHA256 + Redis TokenStore + Filter |
| ADR-012 | 前端 API 拦截器 | 🟢 Accepted | ADR-011 | ✅ 已完成 | Axios 双拦截器 + 401 防循环 |
| ADR-013 | Batch 11/12/13 规划 | 🟢 Accepted | ADR-001 | ✅ 已完成 | 三批次交付边界+Agent分工决策 |
| — | derekcoding-phase5.md | — | — | ⚠️ 参考 | Phase 5 开源整合与内化 |

---

## ADR 关系图谱

```
ADR-001 (双轨并行)
  ├── ADR-002 (技术栈) ──┬── ADR-003 (数据库) ── ADR-010 (Flyway迁移)
  │                      │                      └── ADR-013 (批次规划)
  │                      ├── ADR-005 (i18n)
  │                      ├── ADR-011 (JWT认证) ── ADR-012 (前端拦截器)
  │                      └── ADR-013 (批次规划)
  └── ADR-004 (OpenClaw 集成)

ADR-006 (涟漪分析) — 元方法论，独立于各 ADR
ADR-013 (批次规划) — 元规划，关联所有 ADR
```

---

## 待补充 ADR (候选)

根据 aITMS Best Practice 和 TradeOMS 演进需要，以下主题可能需要 ADR：

| 候选主题 | 触发条件 | 优先级 |
|---------|---------|--------|
| 部署拓扑与运行时隔离 | 首次部署时 | 🟢 中 |
| 前端技术选型与拓扑 | 开始前端开发时 | 🟡 中 |
| 部署拓扑与运行时隔离 | 首次部署时 | 🟢 中 |
| 数据备份与灾备策略 | 数据持久化上线前 | 🟢 中 |
| API 契约版本化策略 | 出现外部 API 消费者时 | 🟢 低 |
| LLM 开放策略与审计设计 | AI 功能模块开始开发时 | 🟡 中 |
| 产品手册交付规约 | 手册体系需要标准化时 | 🟢 中 |
