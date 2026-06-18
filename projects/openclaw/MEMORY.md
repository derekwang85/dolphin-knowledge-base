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

## Promoted From Short-Term Memory (2026-06-15)

<!-- openclaw-memory-promotion:memory:memory/2026-06-10-tradeoms-summary.md:18:21 -->
- 上午 (09:00~12:00): 团队ABCD岗制度确立; OE_V1 52张Figma截图全分析 → 发现CDP缺失业务域; Document Management模块设计+PRD; 系统全景架构图 system-map-complete.md [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-10-tradeoms-summary.md:18-21]
<!-- openclaw-memory-promotion:memory:memory/2026-06-10-tradeoms-summary.md:24:27 -->
- 下午 (14:00~18:00): **大鱼** → KYC前端组件填充 → CDP前端精修 → Document前端; **斗牛** → Document后端 → Quote后端全部8API+状态机+限价规则; **巨贵** → Document前端框架(UI还原) → 代码审查; WBS补充: 横切关注点C/A/B(C.1~C.12) + D文档专项 + E API契约 [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-10-tradeoms-summary.md:24-27]
<!-- openclaw-memory-promotion:memory:memory/2026-06-10-tradeoms-summary.md:30:33 -->
- 晚上 (19:00~23:45): **大鱼** → Order前端1,466行 → VA前端1,219行 → TT前端 → LC前端1,532行 → 6个联调脚本; **斗牛** → Order后端1,069行 → VA后端431行 → TT后端853行 → LC后端1,050行 → JWT安全底座; **柯基** 🆕 → aider安装+MiniMax M2.7配置 → 纳入团队建制; **巨贵** → 代码审查报告+编译检查脚本 [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-10-tradeoms-summary.md:30-33]
<!-- openclaw-memory-promotion:memory:memory/2026-06-10-tradeoms-summary.md:34:36 -->
- 晚上 (19:00~23:45): **二哈** → 改名+明早08:00 cron; 全员昵称体系建立 (边牧/大鱼/斗牛/巨贵/柯基/二哈); WBS F-K-U 全量扩充 (UAT/UX/平台/运维/安全/集成/UI精修) [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-10-tradeoms-summary.md:34-36]
<!-- openclaw-memory-promotion:memory:memory/2026-06-10-tradeoms-summary.md:5:8 -->
- 📊 核心数据: | 指标 | 数值 | |------|:----:| | Commits | 26 | | 代码新增 | 30,133 lines | [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-10-tradeoms-summary.md:5-8]
<!-- openclaw-memory-promotion:memory:memory/2026-06-10-tradeoms-summary.md:9:11 -->
- 📊 核心数据: | 文件变更 | 386 files | | WBS任务数 | 243 → 600+ | | Agent工时 | 6 名成员全时段 | [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-10-tradeoms-summary.md:9-11]
<!-- openclaw-memory-promotion:memory:memory/2026-06-10.md:12:15 -->
- 整体产出（跨双通道）: | §3 CDP | 全量实现 | 10 组件 | ✅ | | §4 Document | 全量实现 | 6 组件 | ✅ | | §5 Quote | 全量 + 限价规则引擎 | 精修中 | ✅ | | §6 Order | 全量 (Reasonix) | 已完成 | ✅ | [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-10.md:12-15]
<!-- openclaw-memory-promotion:memory:memory/2026-06-10.md:16:19 -->
- 整体产出（跨双通道）: | §7 VA | 全量实现 | 全量 (大鱼) | ✅ | | §8 TT | 全量实现 | 前端完成 | ✅ | | §9 LC | 全量实现 | 前端+后端全部完成 | ✅ | | §10 OE | — | — | 🔲 | [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-10.md:16-19]
<!-- openclaw-memory-promotion:memory:memory/2026-06-10.md:20:21 -->
- 整体产出（跨双通道）: | §11 Buyer | JWT+RBAC 安全底座 | — | ⏳ | | §12 集成测试 | 12项联调测试 | — | ⏳ | [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-10.md:20-21]
<!-- openclaw-memory-promotion:memory:memory/2026-06-10.md:25:28 -->
- 本通道关键交付: **架构治理**: 单一事实源 6 个软链接、derekcoding 原则 9 条落地; **DDL 修正**: 新增 `oms_role` / `oms_audit_log` / `oms_business_event`，总 68 表; **质量门禁**: G1-G11 + D1-D12 延迟触发器; **配置兜底**: MinioService @Value 修复 + pre-submit 门禁 [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-10.md:25-28]

## Promoted From Short-Term Memory (2026-06-16)

<!-- openclaw-memory-promotion:memory:memory/2026-06-10.md:6:6 -->
- 整体产出（跨双通道）: **30,133 行代码变更，386 个文件**，7 个 Agent 参与。 [score=0.834 recalls=0 avg=0.620 source=memory/2026-06-10.md:6-6]
<!-- openclaw-memory-promotion:memory:memory/2026-06-10.md:8:11 -->
- 整体产出（跨双通道）: | 模块 | 后端 | 前端 | 状态 | |------|------|------|------| | §1 项目基础 | 规约体系 80 文档 | — | ✅ | | §2 KYC | 全量 API (7端点) 编译通过 | Vue 组件就绪 | ✅ | [score=0.834 recalls=0 avg=0.620 source=memory/2026-06-10.md:8-11]
<!-- openclaw-memory-promotion:memory:memory/2026-06-11.md:14:17 -->
- 🐕 巡检 11:20: **HEAD (workspace)**: ea6158c 扩展工程规范 ✅; **HEAD (TradeOMS)**: 2f00fe2 🐩 巨贵 产品手册完整性审计 Batch 4 ✅; **Free模型**: `openrouter/free` ✅ (已验证可用); **🐶 斗牛**: Java TradeOMS 进程运行中 (自10:54) ✅ — 之前10:40的🔴已解除 [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-11.md:14-17]
<!-- openclaw-memory-promotion:memory:memory/2026-06-11.md:18:21 -->
- 🐕 巡检 11:20: **🧸 泰迪**: CodeBuddy 运行 1h43min — 仍在处理VA前端首单任务 ⏳; **🐋 大鱼**: 今日有产出 (i18n, 审计日志AOP, login修复) ✅; **🐩 巨贵**: 活跃 — Batch8/9审查 + 产品手册完整性审计 Batch4 ✅; **🐺 二哈**: freecode 模型可用 (openrouter/free) ✅ [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-11.md:18-21]
<!-- openclaw-memory-promotion:memory:memory/2026-06-11.md:22:23 -->
- 🐕 巡检 11:20: **🐕 柯基**: 上次提交 i18n修复 (昨夜) — 暂无新任务; **🟢 整体**: 无红灯，全员正常运作 [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-11.md:22-23]
<!-- openclaw-memory-promotion:memory:memory/2026-06-11.md:26:28 -->
- 🐕 巡检 13:04 — 夜巡派活: **HEAD (workspace)**: 4c7ad71 docs: ops-manual 二哈模型红线规则 ✅; **HEAD (TradeOMS)**: ed72cc5 Figma截图对照表+缺口分析 ✅; **CPU**: 3个reasonix所有 0.0% >1h → 🟢 空闲，立刻派活 [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-11.md:26-28]
<!-- openclaw-memory-promotion:memory:memory/2026-06-11.md:4:7 -->
- 🐕 巡检 10:40: **HEAD**: 2044d0b 🐩 巨贵 审查报告#5 (10:39, 1分钟前) ✅; **Free模型**: `openrouter/free` → gpt-oss-20b ✅ WORKING; **🐶 斗牛**: 无运行中的后端进程 ⚠️; **🧸 泰迪**: CodeBuddy 运行中 (09:37起) — VA前端首单任务 [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-11.md:4-7]
<!-- openclaw-memory-promotion:memory:memory/2026-06-11.md:8:11 -->
- 🐕 巡检 10:40: **🐩 巨贵**: 刚完成审查报告#5 (448行); **🐺 二哈**: freecode 存在，模型可用 ✅; **🐋 大鱼**: 未见活跃; **🐕 柯基**: 上次提交为i18n修复 [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-11.md:8-11]

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
