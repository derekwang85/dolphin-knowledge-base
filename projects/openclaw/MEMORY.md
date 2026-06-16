# Long-Term Memory


## Promoted From Short-Term Memory (2026-06-13)

<!-- openclaw-memory-promotion:memory:memory/2026-06-09.md:12:15 -->
- 产出: `04-adr/ADR-005-i18n.md` — i18n 架构决策; `10-i18n/` — 策略 + 6 道门禁 + 术语表 + CI 检查脚本; `11-product-manual/` — 产品手册框架（buyer/operations/openclaw 三端共 32 个功能骨架）; `06-runbook/` 3 个文件 — 补充 i18n 部署/运维/环境检查 [score=0.806 recalls=0 avg=0.620 source=memory/2026-06-09.md:12-15]
<!-- openclaw-memory-promotion:memory:memory/2026-06-09.md:16:18 -->
- 产出: `09-migrations/phase-plan.md` — 各 Phase 标注 i18n + 手册交付节点; `12-development-guide/` — 开发规约全套（工作流、PR checklist、合规速查）; `12-development-guide/spec-ripple-analysis.md` — 元方法论：规约涟漪分析 [score=0.806 recalls=0 avg=0.620 source=memory/2026-06-09.md:16-18]
<!-- openclaw-memory-promotion:memory:memory/2026-06-09.md:23:26 -->
- 做得好的 ✅: 任务理解准确，从 ADR 决策到落地执行全链路覆盖; 不满足于「加个 ADR + 两个文件」，而是系统地构建了一整层规约体系; quick check 矩阵的设计让未来新增规约时有章可循; 与现有 Phase 5 自然衔接形成闭环 [score=0.806 recalls=0 avg=0.620 source=memory/2026-06-09.md:23-26]
<!-- openclaw-memory-promotion:memory:memory/2026-06-09.md:29:32 -->
- 小瑕疵 ⚠️: **第一轮遗漏 runbook/phase-plan 的联动更新** — Derek 指出后才补上; root cause：只看到了「i18n 门禁 = 开发阶段的事」，没意识到「新规约的引入会涟漪扩散到部署/运维/规划文档」; fix：直接沉淀为 spec-ripple-analysis.md 元方法论，纳入 #0 核心规约; **README 中漏了 ADR-005 的目录入口标记** — Derek 没提但我自检发现 [score=0.806 recalls=0 avg=0.620 source=memory/2026-06-09.md:29-32]
<!-- openclaw-memory-promotion:memory:memory/2026-06-09.md:33:33 -->
- 小瑕疵 ⚠️: 没有在一开始就创建 `12-development-guide/`，而是补 runbook 时才意识到需要把门禁时序串起来 [score=0.806 recalls=0 avg=0.620 source=memory/2026-06-09.md:33-33]
<!-- openclaw-memory-promotion:memory:memory/2026-06-09.md:6:6 -->
- 任务: Derek 要求完善 derekcoding，增加： [score=0.806 recalls=0 avg=0.620 source=memory/2026-06-09.md:6-6]
<!-- openclaw-memory-promotion:memory:memory/2026-06-09.md:7:9 -->
- 任务: i18n 语言需求规划和相关门禁（含 ADR）; 产品手册（逐字段、逐功能开发）; 对应规约文件放到 OMS 合适文件夹 [score=0.806 recalls=0 avg=0.620 source=memory/2026-06-09.md:7-9]

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
