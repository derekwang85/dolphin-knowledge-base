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

## 2026-06-22 — 图片自动分析规约

Derek 要求：发图片时自动调用 image/multimodal 工具解读内容，不等他开口说"读一下"或"你看看"。他发图片 = 需要我知道里面是什么。以后所有带图片的消息，第一件事就是分析图片内容。

## Promoted From Short-Term Memory (2026-06-24)

<!-- openclaw-memory-promotion:memory:memory/2026-06-19.md:11:14 -->
- 今日修复记录: **MySQL(Docker)**: ✅ Up 2天, 124 tables, Flyway v88; **后端重启**: 编译失败(import缺失+类型不匹配) → Spring启动失败(条件Bean链式爆炸); **修复**: 14 个文件改动, 含 import 补全、类型修正、@ConditionalOnProperty 移除; **构建门禁**: scripts/pre-compile-check.sh (stale import + conditional bean 检测) [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-19.md:11-14]
<!-- openclaw-memory-promotion:memory:memory/2026-06-19.md:17:19 -->
- 关键教训: DTO 重构后 import 不会自动更新 → stale import → "找不到符号"; @ConditionalOnProperty 只放在 Client 层但下游 Service/Controller 无条件 → 启动时链式崩溃; 经验已写入: docs/lessons/compile-bootstrap-lessons.md + 纳入 build gate (pre-commit + CI) [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-19.md:17-19]
<!-- openclaw-memory-promotion:memory:memory/2026-06-19.md:22:24 -->
- 06:45 — 经验沉淀完成: docs/lessons/compile-bootstrap-lessons.md → TradeOMS 项目知识库; dereknbstartup skill → 新增预检步骤 + 常见陷阱说明; MEMORY.md → 已汇总关键模式 [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-19.md:22-24]
<!-- openclaw-memory-promotion:memory:memory/2026-06-19.md:29:32 -->
- 项目状态: **DereInside** 是一个本地优先 AI 知识系统，从 gbrain 的教训中重写; **Phase 0** ✅: gbrain → derekinside 数据迁移（12 wings, 44 rooms, 555 pages, 2651 chunks）; **Phase 1** ✅: 分层索引 Wing/Room/Drawer + 混合搜索（向量+FTS+RRF）+ LLM 重排序; **Phase 2** ✅: 知识图（687 entities, 1427 links）+ 正则/LLM 混合实体提取 + PageRank 传播重排序 [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-19.md:29-32]
<!-- openclaw-memory-promotion:memory:memory/2026-06-19.md:33:33 -->
- 项目状态: **Phase 3** ✅: MCP 服务器（6 tools, JSON-RPC 2.0 over stdio）+ FastAPI HTTP 桥（6 端点）+ Agent 隔离命名空间 [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-19.md:33-33]

## Promoted From Short-Term Memory (2026-06-26)

<!-- openclaw-memory-promotion:memory:memory/2026-06-21.md:12:15 -->
- Issue Log 更新: **ISS-001~074**: 全部 ✅ 已验证 (70/75); **ISS-005**: 🔶 HITL (Email IMAP, Derek标记不急着修); **ISS-075~078**: ❌ 已派发 → **已更新为 ✅ 已验证**; ISS-075: TT UAT 201→200 对齐 + 请求体补充 [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-21.md:12-15]
<!-- openclaw-memory-promotion:memory:memory/2026-06-21.md:16:19 -->
- Issue Log 更新: ISS-076: Admin UAT 11/11 API路径对齐; ISS-077: KYC Python UAT 8/8 全部通过 (脚本v2重构); ISS-078: Document/LC/OE/Order 全模块UAT API路径对齐; **Total: 74/75 已验证 + 1 HITL** [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-21.md:16-19]
<!-- openclaw-memory-promotion:memory:memory/2026-06-21.md:22:25 -->
- TCR (Test Case Register): v19 → v20 (367条), 全部登记/已验证; passCount: 298, failCount: 59; fulltest baseline (R5): 1341 total, 1127 passed (84%); Controller 全部清零 ✅ (358/358, 100%) [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-21.md:22-25]
<!-- openclaw-memory-promotion:memory:memory/2026-06-21.md:26:26 -->
- TCR (Test Case Register): 应用层剩余214个待修 [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-21.md:26-26]
<!-- openclaw-memory-promotion:memory:memory/2026-06-21.md:29:32 -->
- UAT 结果 (已验证): KYC Python: 8/8 ✅; Quote: 13/13 ✅; TT: 修复闭环 ✅; Document/LC/OE/Order: API路径全部对齐 ✅ [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-21.md:29-32]
<!-- openclaw-memory-promotion:memory:memory/2026-06-21.md:6:9 -->
- Git (凌晨活跃 Chain): 20 commits 从 2026-06-20 16:19 → 06-21 00:19; 范围: 权限系统 Phase1~6 → 生产部署 → UAT全方位修复 → 文档规整 → wbs审计同步; **1336/0/0 fulltest** 出现在 2 个 commit 中 (9ea02c12, 5462b377); 最后一次 PR merge: 187682fc (feature/preproduction → main) [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-21.md:6-9]

## Promoted From Short-Term Memory (2026-06-27)

<!-- openclaw-memory-promotion:memory:memory/2026-06-22.md:12:12 -->
- 问题: embedder.py / entity.py / reranker.py 都硬编码 Ollama，切换模型要改 3 个文件，不支持 vLLM/OpenAI/FreeCode，无 GPU 加速，无自动降级。 [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-22.md:12-12]
<!-- openclaw-memory-promotion:memory:memory/2026-06-22.md:15:17 -->
- 设计演化（Derek 指导核心决策）: **第一次设计**（被 Derek 否决）：按能力分裂的 3 个 Provider 接口（EmbeddingProvider / ExtractionProvider / RerankProvider）→ 太僵化，Provider 一次实现 3 个接口; **第二次设计**（被 Derek 指出不够灵活）：Model 作为配置单元 + Pipeline 策略（fallback/quality/speed）→ 缺少认知能力维度; **最终设计**：Model Registry（四维属性：intelligence/cost_tier/speed_tier/quality）+ Pipeline 约束求解器（min_intelligence / max_cost / max_latency_ms 约束 + objective 优化）+ ModelProfiler（自动探测模型位置） [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-22.md:15-17]
<!-- openclaw-memory-promotion:memory:memory/2026-06-22.md:20:23 -->
- 实现代码: **engine/model.py**（240行）— ModelEndpoint 统一接口 + ModelProfile 四维属性 + satisfies 约束检查; **engine/registry.py**（200行）— ModelRegistry（注册/发现/自动 profile/磁盘缓存）; **engine/pipeline.py**（200行）— PipelineResolver 约束求解（严格→relax→rank）; **engine/profiler.py**（425行）— ModelProfiler（boot probe 3 samples 2s + deep probe 15 samples 10s）+ PassiveObserver（零成本侧效应监控）+ 震荡检测 [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-22.md:20-23]
<!-- openclaw-memory-promotion:memory:memory/2026-06-22.md:24:27 -->
- 实现代码: **engine/engine.py**（170行）— Engine facade（统一 embed/extract/rerank/generate API）; **drivers/ollama.py**（230行）— OllamaModel（embed/extract/rerank/generate 四合一）; **drivers/vllm.py**（110行）— VLLMModel（embed only，GPU）; **drivers/openai.py**（180行）— OpenAIModel（兼容 OpenAI/FreeCode/MiniMax） [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-22.md:24-27]
<!-- openclaw-memory-promotion:memory:memory/2026-06-22.md:28:28 -->
- 实现代码: **config.yaml**（195行）— 新格式（models + pipeline 取代旧三段） [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-22.md:28-28]
<!-- openclaw-memory-promotion:memory:memory/2026-06-22.md:5:5 -->
- 会话摘要: 全天重构 DereInside 模型调用架构，从硬编码 Ollama 三件套（embedder/entity/reranker）升级为 Model Registry + Pipeline 约束求解 + 自动探测。同时完成轨道 C（实体→图谱改进）+ 轨道 B 半程（智能调度 + 跨 Wing 融合）。 [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-22.md:5-5]

## Promoted From Short-Term Memory (2026-06-29)

<!-- openclaw-memory-promotion:memory:memory/2026-06-24.md:10:13 -->
- P0 #1 — Spec 设计用例 → 集成测试脚本: `scripts/spec-to-test.py` — spec markdown 测试表格解析 → 标准化 bash 集成测试脚本; 29 个 spec → 29 个脚本, 365 条自动化集成测试用例; 1,671 条 spec 设计用例中识别出 365 条可自动化的; 其余 1,306 条标记为 UT/E2E/手动 [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-24.md:10-13]
<!-- openclaw-memory-promotion:memory:memory/2026-06-24.md:16:19 -->
- P0 #2 — Register.json 执行绑定门禁: `scripts/test-register-updater.py` — 测试执行结果→register.json 自动同步; `scripts/tc-pipeline.sh` — 统一测试执行管道（run/validate/report/cleanup/install-gate）; 发现并清理 **145 个假 PASS**（标记 PASS 但 `real_verified=false`）; pre-commit hook 门禁 — 禁止手动修改 register.json 状态 [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-24.md:16-19]
<!-- openclaw-memory-promotion:memory:memory/2026-06-24.md:22:25 -->
- P1 #3 — Jacoco 覆盖率缺口分析: 跑 `mvn test + jacoco:report` — 1,410 tests, 46.1% 指令覆盖率; 核心发现: OrderController 等 6 个 REST Controller 覆盖率为 0%; 25 条 TC-COV-* 覆盖率缺口注册到 register.json; `docs/testing/jacoco-gap-analysis-2026-06-24.md` [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-24.md:22-25]
<!-- openclaw-memory-promotion:memory:memory/2026-06-24.md:28:31 -->
- P2 #5 — PIT Mutation Testing: 加 PIT 1.17.1 + junit5-plugin 到 pom.xml; 跑 5,647 个变异, 502 杀死 → **82.6% 域层变异杀死率**; 发现 2 个脆皮测试（baseline 不通过）; `OrderEntityContractTest.t8` — 合字段用 `isSynthetic()` 过滤修复 [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-24.md:28-31]
<!-- openclaw-memory-promotion:memory:memory/2026-06-24.md:32:35 -->
- P2 #5 — PIT Mutation Testing: `sync_AlreadySynced` — Mockito/PIT 兼容性问题, 排除; 发现 1 个真实 bug: `CrossModuleDataFlowTest` 缺 `@Transactional` → 加 `@Transactional` 修复; 但 `generateOrderNo()` 用 `SELECT COUNT+1` 生成序号 → 多次运行必冲突; 实际根因: 集成测试数据清理机制缺失 [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-24.md:32-35]
<!-- openclaw-memory-promotion:memory:memory/2026-06-24.md:36:37 -->
- P2 #5 — PIT Mutation Testing: 临时方案: `mysql -e "DELETE FROM oms_order WHERE ..."` 清理数据库; 这个问题的持久修复需要提给 Derek [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-24.md:36-37]
<!-- openclaw-memory-promotion:memory:memory/2026-06-24.md:5:5 -->
- 概况: Derek 给出大方向：让测试资产真正起作用。我提出 P0-P3 七项计划，Derek全额批准，逐项推进，6小时内全线完成。 [score=0.812 recalls=0 avg=0.620 source=memory/2026-06-24.md:5-5]
