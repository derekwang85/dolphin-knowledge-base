# 🐩 巨贵 Batch 11 涟漪审查报告 — 审计漏排全量排期

**审查时间**: 2026-06-12 11:30 CST  
**审查范围**: C.11.13 — 审计漏排涟漪审查  
**审查基准**: 
- 审计报告 `audit-0612-0645.md`（联调扫街全量问题）
- 巨贵审查#6 `review-report-codex-6.md`（4 CRITICAL + 8 MAJOR）
- 联调扫街报告 `integration-sweep-issues.md`
- i18n硬编码审计 `i18n-hardcode-audit.md`

---

## 一、问题清单总览

### 1.1 联调扫街全量问题（源自 integration-sweep-issues.md）

| # | 问题 | 级别 | 模块 | 现状 |
|:-:|------|:----:|:----:|:----:|
| IS-01 | **VA 6个单元测试全部失败** (vnode null) | 🔴 P1 | VA前端 | 推进中，大鱼派单 |
| IS-02 | **MySQL/Redis/MinIO 未启动** (需sudo/docker) | 🔴 P0 | 基础设施 | 审计已确认实际均运行✅ |
| IS-03 | **KYC POST buyer_id NOT NULL** 缺字段 | 🔴 P1 | KYC后端 | 未排期 |
| IS-04 | **KYC→CDP 未自动创建公司** (APPROVED后) | 🟡 P2 | KYC+CDP | 未排期 |
| IS-05 | **Quote→Order 未自动触发订单创建** | 🟡 P2 | Quote+Order | 未排期 |
| IS-06 | **Order→VA/TT/LC 支付联动不更新状态** | 🟢 P3 | 跨模块 | 未排期 |

### 1.2 巨贵审查#6 新增问题（4 CRITICAL + 8 MAJOR）

| # | 问题 | 级别 | 模块 | 对应审计 |
|:-:|------|:----:|:----:|:--------:|
| C01 | **VaTransactions.vue 全组件硬编码中文** | 🔴 CRITICAL | VA前端 | 审计C01 |
| C02 | **前端API调用缺失 companyId** (deposit/withdraw) | 🔴 CRITICAL | VA前后端 | — |
| C03 | **VaController 缺少 @PreAuthorize** (垂直越权) | 🔴 CRITICAL | VA后端 | — |
| C04 | **VaTransactions.vue Mock数据替代真实API** | 🔴 CRITICAL | VA前端 | — |
| M01 | **FreezeForm.vue 验证消息硬编码** | 🟠 MAJOR | VA前端 | — |
| M02 | **VaDashboard.vue 冗余computed+window.i18n** | 🟠 MAJOR | VA前端 | — |
| M03 | **Static Balance Row 硬编码数值** | 🟠 MAJOR | VA前端 | — |
| M04 | **API参数名 companyId vs accountId不一致** | 🟠 MAJOR | VA前后端 | — |
| M05 | **#5遗留 — @PreAuthorize全员未恢复** | 🟠 MAJOR | 全后端 | — |
| M06 | **#5遗留 — Order @Version 乐观锁缺失** | 🟠 MAJOR | Order后端 | — |
| M07 | **#5遗留 — CtrmSyncService Thread.sleep阻塞** | 🟠 MAJOR | Order后端 | — |
| M08 | **#5遗留 — CtrmSyncService 10%随机失败Stub** | 🟠 MAJOR | Order后端 | — |

### 1.3 i18n硬编码审计 + 审计报告额外问题

| # | 问题 | 级别 | 模块 | 文件 |
|:-:|------|:----:|:----:|------|
| I18N-01 | **FreezeForm.vue 21处硬编码** | 🟥 HIGH | VA前端 | 用户可见UI文本全部硬编码 |
| I18N-02 | **DepositForm.vue 13处硬编码** | 🟥 HIGH | VA前端 | label/placeholder/confirm对话框 |
| I18N-03 | **VaApplicationService.java 14处硬编码** | 🟥 HIGH | VA后端 | 异常消息+txRemark硬编码 |
| I18N-04 | **QuoteApplicationService.java 7处硬编码** | 🟥 HIGH | Quote后端 | 状态检查异常 |
| I18N-05 | **CtrmSyncService.java 8处硬编码** | 🟥 HIGH | Order后端 | 异常+日志中文 |
| I18N-06 | **LcApplicationService.java 7处硬编码** | 🟥 HIGH | LC后端 | 异常消息中文 |
| I18N-07 | **TtApplicationService.java 5处硬编码** | 🟥 HIGH | TT后端 | 异常消息中文 |
| I18N-08 | **Order+Auth+KYC+Security 19处硬编码** | 🟥 HIGH | 多模块 | 散落于各Service |
| A-01 | **泰迪空闲6h+** (进程未启动) | 🔴 P0 | 调度 | 需边牧启动CodeBuddy |
| A-02 | **二哈OpenRouter离线>12h** | 🔴 P0 | 基础设施 | 需决策停机/换模型 |
| A-03 | **前端Vite port 5173不可达** | 🟡 P2 | 工具链 | 检查启动状态 |
| A-04 | **审计误判：MySQL/Redis/MinIO状态** | 🟢 MINOR | 审计质量 | 06-12 00:45 误报已修复 |

---

## 二、涟漪分析（三层波及面）

### 2.1 涟漪簇#1：VA前端三连阻塞（C01+C04+M01+M02+M03+I18N-01+I18N-02）

**核心问题**: VaTransactions页面硬编码/无数据 + VaDashboard冗余 + FreezeForm硬编码  
**涉及文件**: VaTransactions.vue, VaDashboard.vue, FreezeForm.vue, DepositForm.vue, va-api.js

**第一层（直接影响）**:
| 影响模块 | 具体影响 |
|---------|---------|
| VaTransactions 页面 | 全部标签始终中文，Mock数据无真实流水，分页失效 |
| VA Dashboard | 余额卡片静态数据与后端不联动，用户困惑 |
| FreezeForm/DepositForm 弹窗 | 验证消息不跟随语言切换，UI割裂 |
| VA前端E2E | 因Mock数据无法联调，交易流水无真实测试数据 |

**第二层（间接波及）**:
| 影响模块 | 具体影响 |
|---------|---------|
| Buyer端整体体验 | VA是Buyer核心支付模块，中英文展示不一致破坏信任 |
| 运营端VA管理 | 运营无法核实Buyer VA页面展示是否正确 |
| 单元测试 | 6个VA测试全部失败→CI不能全绿→Batch 11门禁不通过 |
| i18n门禁G2 | 连续新增硬编码，门禁形同虚设 |

**第三层（系统性根因）**:
| 根因 | 说明 |
|------|------|
| i18n执行断层 | Dev知道规范但交付时不检查（VaDashboard正确但VaTransactions遗漏） |
| 缺乏联调门禁 | 无强制"移除Mock数据→启用真实API"的checklist |
| 前端UIreview缺位 | VA前端无人做Figma对照审查，静态数据未发现 |

---

### 2.2 涟漪簇#2：VA安全防线缺失（C03+M04+M05）

**核心问题**: VaController无@PreAuthorize + companyId参数不一致 + 全员安全缺失  
**涉及文件**: VaController.java, va-api.js, 全体Controller

**第一层（直接影响）**:
| 影响模块 | 具体影响 |
|---------|---------|
| VaController 8端点 | 无任何安全控制→未登录用户可充值/扣款/冻结 |
| VA前端deposit/withdraw | companyId不传→400拒绝→功能不可用 |
| API参数一致性 | frontend传accountId, backend读companyId→400错误 |

**第二层（间接波及）**:
| 影响模块 | 具体影响 |
|---------|---------|
| 全系统安全 | 所有Controller @PreAuthorize缺失→Order/KYC/Doc/Quote 全线风险 |
| 资金安全 | 资金操作无角色约束→任意用户可操作他人账户 |
| 审计合规 | 无权限管控→无法通过合规审计 |

**第三层（系统性根因）**:
| 根因 | 说明 |
|------|------|
| 安全底座未完工 | C.1.03 (API端点权限注解) 尚未排期执行 |
| 安全审查无门禁 | #5 C02已指出但连两轮未修复，无强制追踪 |
| API契约失步 | frontend与backend参数名无统一注册表校验 |

---

### 2.3 涟漪簇#3：跨模块集成断裂（IS-03/04/05/06）

**核心问题**: KYC→CDP→Quote→Order→VA/TT/LC 流水线不完整  
**涉及文件**: KycApplicationService, QuoteApplicationService, OrderApplicationService, VaController

**第一层（直接影响）**:
| 影响模块 | 具体影响 |
|---------|---------|
| KYC→CDP (IS-04) | KYC审批通过后CDP无数据→客户管理为空 |
| Quote→Order (IS-05) | 报价确认后需手动创建订单，无自动流转 |
| Order→VA/TT/LC (IS-06) | 支付后订单状态不联动更新 |
| KYC buyer_id (IS-03) | POST创建时buyer_id缺字段→SQL 500 |

**第二层（间接波及）**:
| 影响模块 | 具体影响 |
|---------|---------|
| E2E全链路 | 跨模块测试无法执行→E2E覆盖率=0 |
| Buyer体验 | Buyer完成KYC后看不到CDP公司→困惑 |
| 运营效率 | 报价→订单需运营手动操作→运营反对 |

**第三层（系统性根因）**:
| 根因 | 说明 |
|------|------|
| 模块独立开发无集成门禁 | 各模块各自为政，无人负责跨模块流水线 |
| E2E测试欠奉 | F.3跨模块验收全未开始 |
| API契约未落地 | E系列契约注册表KYC/CDP等均未定稿 |

---

### 2.4 涟漪簇#4：#5遗留技术债（M05/M06/M07/M08 + I18N-03~08）

**核心问题**: #5审查指出的问题跨两轮未修，技术债务加速积累  
**涉及文件**: 全体Controller, Order实体, CtrmSyncService, 各ApplicationService

**第一层（直接影响）**:
| 影响模块 | 具体影响 |
|---------|---------|
| @PreAuthorize全员缺失 | 全系统无方法级安全控制 |
| Order乐观锁缺失 | 并发下订单状态可能混乱 |
| CTRM同步Thread.sleep | 生产环境下可能耗尽Tomcat线程池 |
| CTRM 10%故意失败 | 生产环境不可接受的退化逻辑 |
| 后端113处硬编码中文 | Buyer端看到中文异常→体验割裂 |

**第二层（间接波及）**:
| 影响模块 | 具体影响 |
|---------|---------|
| 上线准备 | 这些P1/P2问题是上线的拦路石 |
| 团队信任 | 同一问题跨轮未修→审查权威性受损 |
| 代码质量 | 硬编码+Stub+阻塞→技术债年化利率最高 |

**第三层（系统性根因）**:
| 根因 | 说明 |
|------|------|
| 修复无强制排期 | #5问题标注"未修复"但未强制入WBS排期 |
| 审查→修复闭环断裂 | 审计发现→入WBS→排期→修复→验证 流程未打通 |
| 无验收红线 | 未定义"某批次/某模块上线前必须修复的债务底限" |

---

## 三、建议排期（批次 + Agent）

### 3.1 排期总表

| 批次 | 模块 | 问题 | 子问题数 | Agent | 优先级 |
|:----:|------|------|:--------:|:-----:|:------:|
| **11.1** | VA前端 | 三连阻塞: i18n+Mock数据+companyId | 7 | 🐋 大鱼 | 🔴 P0 |
| **11.2** | VA后端 | @PreAuthorize+硬编码异常 | 2 | 🐶 斗牛 | 🔴 P0 |
| **11.3** | 全后端 | @PreAuthorize批量补全+乐观锁 | 2 | 🐶 斗牛 | 🔴 P0 |
| **11.4** | Order后端 | CTRM修复(异步+移除Stub) | 2 | 🐶 斗牛 | 🔴 P0 |
| **11.5** | KYC+CDP | buyer_id+CDP自动创建 | 2 | 🐶 斗牛 | 🟡 P1 |
| **11.6** | 跨模块 | Quote→Order+Order→VA/TT/LC联动 | 2 | 🐶 斗牛 | 🟡 P1 |
| **11.7** | 后端全模块 | 后端硬编码中文→MessageUtil统一 | 36 | 🐶 斗牛 | 🟡 P1 |
| **11.8** | VA前端 | FreezeForm/DepositForm i18n修复 | 2 | 🐋 大鱼 | 🟡 P1 |
| **11.9** | VA前端 | VaDashboard冗余computed清理+静态数据联动 | 2 | 🐋 大鱼 | 🟡 P1 |
| **11.10** | 后端 | API参数名对齐(accountId→companyId) | 1 | 🐶 斗牛 | 🟡 P1 |
| **11.11** | Quote后端 | QuoteApplicationService i18n | 1 | 🐶 斗牛 | 🟢 P2 |
| **11.12** | LC/TT后端 | 硬编码中文修复 | 2 | 🐶 斗牛 | 🟢 P2 |
| **11.13** | Cross | Order=KYC=Auth最小残留硬编码 | 3 | 🐶 斗牛 | 🟢 P2 |

### 3.2 基础设施调度项

| 批次 | 问题 | 负责人 | 说明 |
|:----:|------|:------:|------|
| C.11.08 | MySQL/Redis/MinIO 重启+验证 | 🐶 斗牛 | 审计已确认运行✅ 按需重启 |
| C.11.09 | 后端编译重启 | 🐶 斗牛 | mvn clean package + SpringBoot |
| C.11.10 | 前端Vite启动 | 🐋 大鱼 | cd frontend && npm run dev |
| C.11.11 | 二哈OpenRouter排查 | 🐕 边牧 | 标注停机/切换模型 |
| C.11.12 | 泰迪进程启动 | 🐕 边牧 | Quote UI精修+手动启动CodeBuddy |

---

## 四、问题→WBS 映射表

| WBS | 批次 | 问题ID | 描述 | 级别 | Agent | 状态 | 排期 |
|:---:|:----:|:------:|------|:----:|:-----:|:----:|:----:|
| **C.11.13.1** | 11.1 | C01, C04, I18N-01, I18N-02 | VA前端三连阻塞: VaTransactions全硬编码+Mock数据未解除+FreezeForm/DepositForm i18n | 🔴 | 🐋 大鱼 | 🔲 | 本周 |
| **C.11.13.2** | 11.1 | M01~M04, M02 | VaDashboard冗余computed+静态数据联动+accountId对齐+window.i18n | 🟠 | 🐋 大鱼 | 🔲 | 本周 |
| **C.11.13.3** | 11.2 | C02, C03 | VA前后端: companyId缺失+VaController @PreAuthorize | 🔴 | 🐶 斗牛 | 🔲 | 本周 |
| **C.11.13.4** | 11.3 | M05 | 全员Controller @PreAuthorize批量补全 | 🔴 | 🐶 斗牛 | 🔲 | 本周 |
| **C.11.13.5** | 11.3 | M06 | Order实体 @Version乐观锁 | 🔴 | 🐶 斗牛 | 🔲 | 本周 |
| **C.11.13.6** | 11.4 | M07, M08 | CtrmSyncService: Thread.sleep→@Async + 移除10%Stub | 🔴 | 🐶 斗牛 | 🔲 | 本周 |
| **C.11.13.7** | 11.5 | IS-03, IS-04 | KYC: buyer_id DTO补全 + APPROVED后自动创建CDP公司 | 🟡 | 🐶 斗牛 | 🔲 | 本周 |
| **C.11.13.8** | 11.6 | IS-05, IS-06 | 跨模块: Quote确认→自动Order + VA/TT/LC支付→Order状态联动 | 🟡 | 🐶 斗牛 | 🔲 | 下周 |
| **C.11.13.9** | 11.7 | I18N-03~08 | 后端Service 36处硬编码中文→MessageUtil统一切换 | 🟡 | 🐶 斗牛 | 🔲 | 下周 |
| **C.11.13.10** | 11.10 | M04, I18N-03 | VA前后端: accountId→companyId + VaApplicationService硬编码 | 🟡 | 🐶 斗牛 | 🔲 | 下周 |
| **C.11.13.11** | 11.11~13 | I18N-04~08 | Quote/LC/TT/Order后端残留硬编码 | 🟢 | 🐶 斗牛 | 🔲 | 迭代 |
| **C.11.13.12** | — | A-01 | 泰迪进程未启动→Quote UI精修停滞 | 🔴 | 🐕 边牧 | 🔲 | 立即 |
| **C.11.13.13** | — | A-02 | 二哈OpenRouter离线>12h→标注停机或切换 | 🔴 | 🐕 边牧 | 🔲 | 立即 |
| **C.11.13.14** | — | A-03 | 前端Vite port 5173不可达→检查启动 | 🟡 | 🐋 大鱼 | 🔲 | 本周 |

### 映射规则说明

```
C.11.13 = 审计漏排涟漪审查（本次任务）
  ├── .1  VA前端三连阻塞（i18n + Mock + companyId）
  ├── .2  VA前端冗余+静态数据
  ├── .3  VA后端安全+companyId阻断
  ├── .4  全员@PreAuthorize安全补全
  ├── .5  Order乐观锁
  ├── .6  CTRM异步+移除Stub
  ├── .7  KYC修复(buyer_id + CDP自动创建)
  ├── .8  跨模块联动(Quote→Order + Order→VA/TT/LC)
  ├── .9  后端36处硬编码中文
  ├── .10 VA前后端参数对齐
  ├── .11 后端残留硬编码(Quote/LC/TT/Order)
  ├── .12 泰迪调度
  ├── .13 二哈调度
  └── .14 Vite启动
```

---

## 五、汇总统计

| 指标 | 数值 |
|------|:----:|
| 排查问题总数 | **30项**（联调扫街6 + 审查#6新增12 + i18n 8 + 审计额外4） |
| 🔴 CRITICAL/P0 | **8项**（VA三连阻塞3 + 安全3 + 调度2） |
| 🟠🟡 MAJOR/P1 | **12项**（i18n+跨模块+参数对齐） |
| 🟢 MINOR/P2+ | **10项**（售后+迭代） |
| 去重后实际修复项 | **14个WBS子条目**（含调度3项） |

### 当前进度总结

| 指标 | 状态 |
|------|:----:|
| 已入WBS排期 | ✅ C.11.13.1~C.11.13.14 全部排期 |
| 已指定Agent | ✅ 🐋🐶🐕 均已标注 |
| 涟漪分析 | ✅ 4簇涟漪已完成三层波及分析 |
| 映射表 | ✅ 问题→WBS→批次→Agent→状态 四维清晰 |
| 遗留验证 | ⚠️ C01/C02/C03分析完成待验证（大鱼+斗牛修复中） |

---

## 六、柯基交叉验证兜底

柯基 `ci/audit-wbs-crosscheck.sh` 应新增对本报告映射表的交叉验证：
1. 检查 `C.11.13.*` 各条目是否已入 `project-wbs-full.md`
2. 检查各问题在下轮审计中是否被提及为"修复中"或"已修复"
3. 🔴 任何已排期问题连续2轮审计无进展→触发升级

---

*报告生成: 2026-06-12 11:30 CST* | *审查官: 🐩 巨贵*
