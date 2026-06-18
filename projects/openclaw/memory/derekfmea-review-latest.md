# FMEA 月度复审 — 基线 (2026-06-18)

> 首次全组件健康基线扫描，作为后续对比基准。

## 健康基线

| 组件 | 状态 | 说明 |
|:-----|:----:|:-----|
| memory-index | ✅ | Dirty: no, Embeddings: ready, 123 files, 1553 chunks |
| gateway-config | ✅ | Runtime: running (pid 3476292) |
| wbs-board | ✅ | project-wbs-full.md 存在 (568 lines) |
| cron-jobs | ✅ | enabled=True, 29 jobs |
| dolphin-gbrain | ✅ | HTTP bridge: 200 |
| media-server | ✅ | HTTP 200 (localhost) |

## 备注
- media-server 10.88.98.20 外网接口路由不通（疑似网络层问题），健康检查已改用 127.0.0.1
- memory-index 曾经发生过嵌入模型配置不匹配导致 memory_search 暂停，已通过重建解决
- 本次扫描用 recover.sh --mode check 完成
