---
name: derekgitworkspace
description: >
  工作区 Git 自动同步技能。检测文件变更→自动提交→推送到 GitHub。
  支持心跳触发、手动触发、定时触发三种模式。
  防止虚拟机迁移或意外丢失导致工作区信息遗失。
---

# derekgitworkspace — 工作区自动 Git 同步

## 核心职责

1. **变更检测** — 每次触发时检查工作区未跟踪/已修改文件
2. **智能提交** — 按变更类型自动生成 commit message
3. **自动推送** — 提交后推送到 GitHub 远程仓库
4. **静默模式** — 无变更时安静退出，不产生噪音

---

## 触发方式

| 方式 | 说明 | 频率建议 |
|------|------|---------|
| **心跳（推荐）** | 每次 heartbeat 轮次中检查一次变更 | 每 30-60 分钟 |
| **定时任务** | 通过 cron 在固定时间点执行 | 每小时或每 2 小时 |
| **手动** | 主动调用 `derekgitworkspace` 或对应指令 | 按需 |
| **后置钩子** | 重大操作完成后自动触发（如规约更新后） | 事件驱动 |

---

## 工作流

```
触发（心跳/cron/手动）
  │
  ├── git status --porcelain
  │   ├── 无变更 → 安静退出
  │   └── 有变更 →
  │       ├── 分类变更文件
  │       ├── 生成 commit message
  │       ├── git add -A
  │       ├── git commit
  │       └── git push
  │
  └── 输出结果（仅在非静默模式下）
```

## 变更分类与 commit message 模板

脚本自动识别变更文件类型，生成对应的 commit 前缀：

| 变更类型 | 检测方式 | Commit 前缀 | 示例 |
|---------|---------|------------|------|
| 日常笔记 | `memory/2026-*.md` | `📝 daily:` | `📝 daily: 2026-06-09` |
| OMS 规约文档 | `oms/**/*.md` 含 ADR/模块 | `📐 oms:` | `📐 oms: update ADR-005 i18n gates` |
| 产品手册 | `oms/11-product-manual/**` | `📖 manual:` | `📖 manual: fill KYC operation steps` |
| 技能代码 | `skills/**/*.py` 或 `skills/**/*.md` | `🔧 skill:` | `🔧 skill: derekadmin feedback loop` |
| 脚本 | `scripts/**` | `⚙️ script:` | `⚙️ script: update healthcheck endpoint` |
| 配置/门禁 | `oms/10-i18n/**` 或 `oms/12-*/**` | `🚧 gate:` | `🚧 gate: add PR checklist i18n item` |
| 配置文件 | `.gitignore`, `*.json` 等 | `🔧 config:` | `🔧 config: update .gitignore` |
| 混合/多类型 | — | `🔄 sync:` | `🔄 sync: daily notes + OMS docs update` |
| 默认 | — | `🔨 chore:` | `🔨 chore: auto-sync workspace` |

### 生成逻辑

```bash
# 伪代码
if changes contains memory/*.md then prefix="📝 daily"
elif changes contains oms/11-product-manual/* then prefix="📖 manual"
elif changes contains oms/**/*.md then prefix="📐 oms"
elif changes contains skills/**/* then prefix="🔧 skill"
elif changes contains scripts/**/* then prefix="⚙️ script"
elif changes is mixed then prefix="🔄 sync"
else prefix="🔨 chore"
```

## 安全约束

| 规则 | 说明 |
|------|------|
| **仅跟踪已 gitignore 排除的文件** | `.openclaw/`、`.learnings/`、`DREAMS.md`、`memory/dreaming/` 等不会意外提交 |
| **不推送空提交** | 无变更时完全静默退出 |
| **不覆盖手动提交** | 检测到未 push 的本地 commit 时会先 push 再检查新变更 |
| **SSH key 已配** | 使用已配置的 `git@github.com:derekwang85/OpenClawWorkSpacePrivate.git` |

## 使用方式

### 手动触发

```
derekgitworkspace: 现在同步工作区
```

### 查看同步状态

```
derekgitworkspace: 查看Git状态
```

### 设置定时同步

建议通过 cron 设置每小时同步一次：

```bash
# 每天 9-22 点间每 2 小时同步一次
0 9-22/2 * * * cd ~/.openclaw/workspace && bash skills/derekgitworkspace/scripts/auto-sync.sh >> /tmp/derekgit-sync.log 2>&1
```

---

## 初始化检查

首次使用前检查 SSH 连接：

```bash
ssh -T git@github.com
# 预期: Hi derekwang85! You've successfully authenticated...
```

## 关联

- 远程仓库: `git@github.com:derekwang85/OpenClawWorkSpacePrivate.git`
- 自动同步脚本: `scripts/auto-sync.sh`
- 心跳集成: 在 HEARTBEAT.md 中注册检查项
