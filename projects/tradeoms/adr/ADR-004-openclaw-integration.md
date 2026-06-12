# ADR-004: OpenClaw 集成方式

> **状态**: 已接受 | **日期**: 2026-06-09

---

## 背景

OpenClaw 作为已有的 AI 交互引擎（aITMS01），可接入 OMS 作为 Buyer 端和运营端的智能交互入口。

## 决策

**OpenClaw 作为 Track B 的消息管理 + AI 推理层，不修改 OpenClaw 核心，通过 API 对接**。

### 集成架构

```
OpenClaw aITMS01
  │
  ├── 消息接收层（WeCom 消息 → 意图识别）
  │     ├── 查询意图 → 调 Java API → 格式化返回
  │     ├── 操作意图 → 调 Java API → 确认 → 执行
  │     └── 磋商意图 → LLM 推理 → 调 Java API → 格式化返回
  │
  ├── 消息推送层（Java API 回调 → 推送到 WeCom）
  │     ├── 报价卡片推送
  │     ├── KYC 审核结果通知
  │     ├── LC 变更通知
  │     └── 付款到账通知
  │
  └── AI 推理层（LLM 调用管理）
        ├── 报价磋商助手
        ├── KYC 预审辅助
        ├── 付款回单解析
        └── 驾驶舱报告生成
```

### API 对接方式

```java
// Java 端暴露 REST API，OpenClaw 调用
// 示例: OpenClaw 查询 VA 余额
GET /api/v1/va/balance?buyerId=xxx

// 示例: OpenClaw 推送报价确认
PUT /api/v1/quote/confirm
{
    "quoteNo": "Q2026-001",
    "confirmedBy": "wecom:2207"
}

// 示例: Java 端触发 OpenClaw 推送
POST /api/v1/message/push
{
    "recipient": "buyer_001",
    "channel": "WECOM",
    "template": "QUOTE_CARD",
    "data": { ... }
}
```

## 数据持久化

OpenClaw 不需要独立数据库，仅使用两张辅助表：

1. `oms_conversation_context` — 会话上下文持久化（重启不丢上下文）
2. `oms_llm_audit_log` — LLM 调用全量日志（审计+回退）

## 影响

- Buyer 端零客户端开发（复用企微）
- AI 能力逐步开放，不冲击传统流程
- LLM 所有输出可审计、可回退、可人工干预
