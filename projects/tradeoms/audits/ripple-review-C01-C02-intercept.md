# 🐩 巨贵 Batch 10 涟漪审查报告 — C01+C02 CRITICAL 修复关联影响

**审查时间**: 2026-06-12  
**审查范围**: C01 VaTransactions.vue i18n硬编码修复 + C02 va-api.js 拦截器修复  
**审查基准**: 涟漪分析三层模型 + 冰山系数评估  

---

## C01 — VaTransactions.vue i18n 硬编码修复

### 发现场景
审计发现 VaTransactions.vue 可能存在中文硬编码。经审查修复后的代码：

- 所有模板文本使用 `$t('va.transaction.*')` 调用 [VaTransactions.vue](frontend/src/views/va/VaTransactions.vue)
- 所有 i18n key 在 en.json 和 zh.json 中均有定义 [en.json](frontend/src/i18n/locales/en.json) [zh.json](frontend/src/i18n/locales/zh.json)

### 第一层（直接影响面）
- ✅ **VaTransactions.vue** — 全部文本走 i18n，无硬编码残留
- ✅ **FreezeForm.vue** — 已修复，使用 `$t('va.freeze.*')` [FreezeForm.vue](frontend/src/components/va/FreezeForm.vue)
- ✅ **DepositForm.vue** — 已修复，使用 `$t('va.deposit.*')` [DepositForm.vue](frontend/src/components/va/DepositForm.vue)
- ✅ **WithdrawForm.vue** — 已确认使用 `$t('va.withdraw.*')` [WithdrawForm.vue](frontend/src/components/va/WithdrawForm.vue)
- ✅ **VaDashboard.vue** — 已确认使用 `$t('va.*')` [VaDashboard.vue](frontend/src/views/va/VaDashboard.vue)

### 第二层（间接波及）
- ✅ en.json `va.transaction.*` 全部 25+ key 完整定义
- ✅ zh.json `va.transaction.*` 全部 25+ key 完整定义
- 其它模块（Quote/Order/TT/LC/KYC）的 i18n key 独立维护，不受 VA i18n 变更影响

### 第三层（系统性根因）
- i18n 规约已纳入编码规范（coding-standards.md §5.2）
- 所有用户可见文本必须走 i18n，禁止硬编码字符串
- 本次审计发现并修复了 VA 模块全部 4 个组件的硬编码问题

### 冰山系数评估
| 项目 | 预期 | 实际 |
|-----|:----:|:----:|
| 发现组件数 | 1 (VaTransactions) | 4 (含Freeze/Deposit/Withdraw/VaDashboard) |
| 系数 | 1× | **4×** ✅ 已全部修复 |

**结论**: ✅ C01 固话通过。无已知残留硬编码。

---

## C02 — va-api.js 拦截器修复

### 发现场景
审计指出 va-api.js 缺少 JWT 请求拦截器和 401 响应处理。

### 第一层（直接影响面）
- ✅ **va-api.js** — 已修复，包含完整的请求拦截器（JWT注入）+ 响应拦截器（401跳转登录） [va-api.js](frontend/src/store/va-api.js)
- ✅ **api.js** (共享实例) — 已有完整拦截器 [api.js](frontend/src/store/api.js)

### 第二层（间接波及）
- ✅ **VaTransactions.vue** 调用 `listTransactions` → 走 va-api 拦截器 → ✅ 有JWT
- ✅ **VaDashboard.vue** 调用 va-api → ✅ 有JWT
- ✅ FreezeForm/DepositForm/WithdrawForm 组件调用 va-api → ✅ 有JWT

### 🔴 RIPPLE FINDING: 独立 axios 实例缺少 JWT 请求拦截器
以下 API 文件创建独立 axios 实例，**但缺少 JWT 请求拦截器**（只有响应拦截器）：

| API 文件 | 请求拦截器(JWT) | 响应拦截器 | 风险等级 |
|----------|:---------------:|:----------:|:--------:|
| `cdp-api.js` | ❌ 缺失 | ✅ 存在 | 🔴 CRITICAL |
| `buyer-api.js` | ❌ 缺失 | ✅ 存在 | 🔴 CRITICAL |
| `document-api.js` | ❌ 缺失 | ✅ 存在 | 🔴 CRITICAL |
| `oe-api.js` | 纯mock数据 | 纯mock | 🟢 无影响 |

**影响分析**：
- 所有 CDP 模块 API 调用（listCompanies, getCompanyDetail, updateCompany 等）不携带 JWT token
- 所有 Buyer 端 API 调用（mock 数据为主，但未来接入实接口时会失败）
- 所有 Document 模块 API 调用（uploadDocument 等）不携带 JWT token

这些接口调用对应的后端 Controller 都标注了 `@PreAuthorize("isAuthenticated()")` 或更严格角色。
缺少 JWT token → 后端返回 401 → 但前端响应拦截器只打 console.error，不跳转登录（缺少401处理逻辑）。

### 第三层（系统性根因）
- 安全底座 C.1.x 尚未完全完工 — 前端各模块 API 层缺少统一拦截器规范
- va-api.js 是显式修复的，但同类的 cdp/buyer/document-api 未被覆盖
- 规范中缺少「所有独立 axios 实例必须附带 JWT 请求拦截器」的硬性规则

### 冰山系数评估
| 项目 | 预期 | 实际 |
|-----|:----:|:----:|
| 有拦截器缺失的文件 | 1 (va-api) | 4 (va已修复, cdp/buyer/document仍缺) |
| 系数 | 1× | **4×** 🔴 仍有3个文件未修复 |

**结论**: ⚠️ C02 va-api.js 已修复。但涟漪分析发现 **cdp-api.js、buyer-api.js、document-api.js** 同类型缺陷未修复。

---

## 排期映射（问题→WBS 映射表）

| WBS | 批次 | 问题ID | 描述 | 级别 | Agent | 状态 | 排期 |
|:---:|:----:|:------:|------|:----:|:-----:|:----:|:----:|
| **C.10.01** | 10 | C01 | VaTransactions.vue i18n硬编码修复 | ✅ | 🐋 大鱼 | ✅ 已修复 | 已完成 |
| **C.10.02** | 10 | C02 | va-api.js 拦截器修复（JWT+401） | ✅ | 🐋 大鱼 | ✅ 已修复 | 已完成 |
| **C.10.03** | 10 | C02涟漪 | cdp-api.js 缺少JWT请求拦截器 | 🔴 | 🐋 大鱼 | 🔲 待修 | 本周 |
| **C.10.04** | 10 | C02涟漪 | buyer-api.js 缺少JWT请求拦截器 | 🔴 | 🐋 大鱼 | 🔲 待修 | 本周 |
| **C.10.05** | 10 | C02涟漪 | document-api.js 缺少JWT请求拦截器 | 🔴 | 🐋 大鱼 | 🔲 待修 | 本周 |
