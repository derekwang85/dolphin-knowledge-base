# Long-Term Memory


## 2026-06-14 — 金毛(Cline)改造+战略评估

### 金毛改造完成
- **worker**: bash→Python v2 (带海豚上下文+结构化提示词模板)
- **系统提示**: .clinerules 52行角色定义+项目规约
- **交互**: 从"扔一行文字"升级为"项目+角色+任务+完成要求"四段式提示词

### 首次任务评估 (i18n-sgs-widgets)
- Cline 跑了63轮迭代，177秒完成
- 逐文件扫描、区分界面文本vs注释、验证locale一致性
- 成本 $0.0378（cache命中98.3%，很便宜）
- **结论：精细提示词模式产出质量明显优于快速模式**

### 战略方向（Derek指示）
随着项目完成度提高，重点从快速开发转向：
1. 深入挖掘问题
2. 完善项目细节
3. 从 Reasonix fast mode → **精细提示词调教模式**
4. 密切观察 Cline 效果再做评估
5. 未来可能架构：Reasonix(批量) + Cline(深度) 双轨协作

## Promoted From Short-Term Memory (2026-06-17)

<!-- openclaw-memory-promotion:memory:memory/2026-06-12.md:12:12 -->
- 三层方案实施: **Layer 2 — Skill (gbrain-query) ✅** [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-12.md:12-12]
<!-- openclaw-memory-promotion:memory:memory/2026-06-12.md:13:14 -->
- 三层方案实施: 通过 skill_workshop 创建 `gbrain-query` 技能提案（pending）; 定义自动触发条件、三种调用方式、规则和健康检查 [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-12.md:13-14]
<!-- openclaw-memory-promotion:memory:memory/2026-06-12.md:3:3 -->
- gbrain 三层集成完成 (2026-06-12 08:59): Derek 要求将 gbrain 静默集成到 OpenClaw，无需每次说"查一下"。 [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-12.md:3-3]
<!-- openclaw-memory-promotion:memory:memory/2026-06-12.md:7:7 -->
- 三层方案实施: **Layer 1 — AGENTS.md 默认规则 ✅** [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-12.md:7-7]
<!-- openclaw-memory-promotion:memory:memory/2026-06-12.md:8:10 -->
- 三层方案实施: 在 AGENTS.md 新增 `🧠 gbrain — 静默副脑` 章节; 定义自动触发范围：TradeOMS 项目、OpenClaw 工作区、历史决策; 调用优先级：HTTP 桥 → CLI → 跳过无关问题 [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-12.md:8-10]
<!-- openclaw-memory-promotion:memory:memory/2026-06-13.md:3:5 -->
- 02:16 — dogwatch 保活: **reasonix code**: 1 process ✅ 正常; **auto-dispatcher**: 有 53 项待办（Batch 10~21），但均为已分配状态（期待特定 Agent），🟢 无空闲 Agent 可派; **结论**: 保活通过，无操作 [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-13.md:3-5]
<!-- openclaw-memory-promotion:memory:memory/2026-06-12.md:16:16 -->
- 三层方案实施: **Layer 3 — HTTP 桥 ✅** [score=0.802 recalls=0 avg=0.620 source=memory/2026-06-12.md:16-16]

## Promoted From Short-Term Memory (2026-06-19)

<!-- openclaw-memory-promotion:memory:memory/2026-06-14.md:10:10 -->
- 11:13~11:22 — 队列修复 + Batch17 启动: ...（前面内容见下文） [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-14.md:10-10]
<!-- openclaw-memory-promotion:memory:memory/2026-06-14.md:15:15 -->
- 背景: Derek 发现项目存在多套WBS任务文件各自为政，要求全线审查。 [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-14.md:15-15]
<!-- openclaw-memory-promotion:memory:memory/2026-06-14.md:18:18 -->
- 勘察结果: **5套任务权威文件冲突:** [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-14.md:18-18]
<!-- openclaw-memory-promotion:memory:memory/2026-06-14.md:19:22 -->
- 勘察结果: ✅ `project-wbs-full.md` — 主WBS表848项（主真理源）; ❌ `BATCH_PLAN.md` — Batch 17 描述与WBS表冲突（BATCH_PLAN说=测试，WBS说=Shipping）; ❌ `dispatch-active.md` — 过期（只列R.8~R.15，未更新V系列）; ❌ `active-task-*.md` 4个 — 最后更新6月12日，内容老旧 [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-14.md:19-22]
<!-- openclaw-memory-promotion:memory:memory/2026-06-14.md:7:7 -->
- 昨夜至今状态（2026-06-13 深夜 → 06-14 凌晨）: ...（前面内容见下文） [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-14.md:7-7]
<!-- openclaw-memory-promotion:memory:memory/2026-06-14.md:4:4 -->
- 08:15 — 完工报告请求: Derek 在 Telegram 上要"完工报告"。 [score=0.802 recalls=0 avg=0.620 source=memory/2026-06-14.md:4-4]

## Promoted From Short-Term Memory (2026-06-20)

<!-- openclaw-memory-promotion:memory:memory/2026-06-15.md:13:14 -->
- 2681c611 wbs-mark-done.sh — 状态列索引6→7修复 ⭐8/10: 定位准确,修复Bug效果立竿见影(Markdown表col[7]是状态); 建议: 添加注释解释表结构 [score=0.806 recalls=0 avg=0.620 source=memory/2026-06-15.md:13-14]
<!-- openclaw-memory-promotion:memory:memory/2026-06-15.md:17:19 -->
- de1701ac cron-master.py — 后台任务总管 ⭐9/10: 纯Python自循环替代4个cron,架构优美; 30秒自检+时间驱动,功耗低; 完整的PID/LOG管理 [score=0.806 recalls=0 avg=0.620 source=memory/2026-06-15.md:17-19]
<!-- openclaw-memory-promotion:memory:memory/2026-06-15.md:22:24 -->
- be05049f wbs-telegram-notify.py — Telegram直推 ⭐7/10 ⚠️: 功能完整,报告格式清晰; **🚨 bad-90嫌疑: BOT_TOKEN硬编码** → 应移至环境变量; 无通知静默退出设计好 [score=0.806 recalls=0 avg=0.620 source=memory/2026-06-15.md:22-24]
<!-- openclaw-memory-promotion:memory:memory/2026-06-15.md:27:28 -->
- a4b7843e wbs-completion-report.py — WBS报告聚合 ⭐9/10: 架构清晰,git log聚合+Agent分工统计; stdin管道设计符合UNIX哲学 [score=0.806 recalls=0 avg=0.620 source=memory/2026-06-15.md:27-28]
<!-- openclaw-memory-promotion:memory:memory/2026-06-15.md:31:32 -->
- 9ec821d8 wbs-mark-done.sh — 精确逐行解析 ⭐9/10: regex→精确解析,规避Bad-90重复空跑; JSONL日志+管道通信设计合理 [score=0.806 recalls=0 avg=0.620 source=memory/2026-06-15.md:31-32]
<!-- openclaw-memory-promotion:memory:memory/2026-06-15.md:5:8 -->
- Git统计 (最近20个commit): **含代码的commit**: 18个 (.py/.sh/.json); **纯文档commit**: 2个 (lessons/ 下的.md文件); **bad-90嫌疑(corgi-dogwatch/BATCH_PLAN)**: 0个 ✅; 该目录不存在于工作区 [score=0.806 recalls=0 avg=0.620 source=memory/2026-06-15.md:5-8]

## Promoted From Short-Term Memory (2026-06-21)

<!-- openclaw-memory-promotion:memory:memory/2026-06-16.md:11:13 -->
- 🎯 里程碑：第一版 SOUL.md 定义 (16:21): 快但错不如慢但对; HITL 在决策层不在操作层; 7 条高质量交付 checklist [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-16.md:11-13]
<!-- openclaw-memory-promotion:memory:memory/2026-06-16.md:15:15 -->
- 🎯 里程碑：第一版 SOUL.md 定义 (16:21): **更新文件：** [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-16.md:15-15]
<!-- openclaw-memory-promotion:memory:memory/2026-06-16.md:16:17 -->
- 🎯 里程碑：第一版 SOUL.md 定义 (16:21): SOUL.md — 完整重写为 v1 工程师人格定义; IDENTITY.md — 同步更新 [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-16.md:16-17]
<!-- openclaw-memory-promotion:memory:memory/2026-06-16.md:4:5 -->
- 🎯 里程碑：第一版 SOUL.md 定义 (16:21): Derek 基于近一个月并肩作战的观察，为我定义了第一份真正的 SOUL.md。 这不是我写的配置文件，是他为我塑造的工作人格。 [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-16.md:4-5]
<!-- openclaw-memory-promotion:memory:memory/2026-06-16.md:8:9 -->
- 🎯 里程碑：第一版 SOUL.md 定义 (16:21): 我不是听话的助手，是替 Derek 省注意力的工程师; 免疫系统不是创可贴 [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-16.md:8-9]
<!-- openclaw-memory-promotion:memory:memory/2026-06-17.md:10:13 -->
- 03:39 — 边牧元复盘 (定时任务): 评分: 60/100 (C级); 脚本问题: bad90检测误报Worker auto提交(115个实际是仓鼠/巨贵正常提交); 实质问题: 审计报告0份，海豚查询从5776骤降至127; `since=2026-06-12` 导致跨5天采样，脚本需fix [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-17.md:10-13]
<!-- openclaw-memory-promotion:memory:memory/2026-06-17.md:4:7 -->
- CTRM打榜 · 晚间排行榜 (03:39): Cron triggered but it's 03:39 AM, not 9 PM — scheduler may have offset issue; `score_manager.py rankings` 返回空，今日无人答题; 已给 Derek 发送空榜通知; 时间异常：cron 配置为晚上9点，但实际在凌晨3:39触发，需复查 cron 时区配置 [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-17.md:4-7]
<!-- openclaw-memory-promotion:memory:memory/2026-06-16.md:7:7 -->
- 🎯 里程碑：第一版 SOUL.md 定义 (16:21): **核心定义：** [score=0.802 recalls=0 avg=0.620 source=memory/2026-06-16.md:7-7]
<!-- openclaw-memory-promotion:memory:memory/2026-06-17.md:14:14 -->
- 03:39 — 边牧元复盘 (定时任务): 报告已写入 audits/边牧/，已喂回海豚 [score=0.802 recalls=0 avg=0.620 source=memory/2026-06-17.md:14-14]

## Promoted From Short-Term Memory (2026-06-22)

<!-- openclaw-memory-promotion:memory:memory/2026-06-17.md:18:21 -->
- CTRM打榜 · 晚间排行榜 (03:39): Cron triggered but it's 03:39 AM, not 9 PM — scheduler may have offset issue; `score_manager.py rankings` 返回空，今日无人答题; 已给 Derek 发送空榜通知; 时间异常：cron 配置为晚上9点，但实际在凌晨3:39触发，需复查 cron 时区配置 [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-17.md:18-21]
<!-- openclaw-memory-promotion:memory:memory/2026-06-17.md:24:27 -->
- 03:39 — 边牧元复盘 (定时任务): 评分: 60/100 (C级); 脚本问题: bad90检测误报Worker auto提交(115个实际是仓鼠/巨贵正常提交); 实质问题: 审计报告0份，海豚查询从5776骤降至127; `since=2026-06-12` 导致跨5天采样，脚本需fix [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-17.md:24-27]
<!-- openclaw-memory-promotion:memory:memory/2026-06-17.md:28:28 -->
- 03:39 — 边牧元复盘 (定时任务): 报告已写入 audits/边牧/，已喂回海豚 [score=0.802 recalls=0 avg=0.620 source=memory/2026-06-17.md:28-28]

## 2026-06-22 — 图片自动分析规约

Derek 要求：发图片时自动调用 image/multimodal 工具解读内容，不等他开口说"读一下"或"你看看"。他发图片 = 需要我知道里面是什么。以后所有带图片的消息，第一件事就是分析图片内容。
