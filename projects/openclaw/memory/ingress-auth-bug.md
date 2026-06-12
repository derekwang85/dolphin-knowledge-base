# Ingress API Auth Bug (2026-05-27)

## 现象
向 `POST /api/v1/wecom/ingress` 发送 JSON 请求时，同时包含以下 4 个字段会导致 500 "Authentication is required":

- `source` (string, non-empty)
- `conversationId` (string, non-empty)
- `content` (string, non-empty)
- `senderId` (string, non-empty)

## 已知正常的情况
- 缺少 senderId → 400 "senderId is required" (auth OK)
- senderId 为空字符串或 null → 400 "senderId is required" (auth OK)
- 3 个字段 (source + conversationId + content 除去 senderId) → 400 validation only (auth OK)
- 3 个字段 (source + conversationId + senderId 除去 content) → 400 "content required" (auth OK)
- senderId 为 number → 500 "raw.trim is not a function" (服务器JS错误)
- senderId 为 boolean → 500 "raw.trim is not a function"

## 根因推测
aITMS Ingress 服务端的 auth 中间件在处理 senderId 字段值时，对其调用了 `.trim()` 后执行了某种查找/验证操作。当 senderId 为非空字符串且 content 同时存在时，该验证失败返回 "Authentication is required"。

## 影响
无法通过 Ingress API 推送带有内容信息的工单。

## 临时方案
待 aITMS 团队修复后，文件 `ctrm-ticket-pending.json` 可重新推送。
