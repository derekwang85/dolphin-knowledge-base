# 🌊 Batch 10 整体涟漪审查

> **审查日期**: 2026-06-25 | **审查者**: Reasonix (deepseek-chat)
> **范围**: 安全底座 / Swagger / Flyway / 索引 / Figma 精修 / 审计日志 / 请求耗时监控

---

## 一、变更清单总览

| # | 变更域 | 入件 | 出件 | 风险等级 |
|---|--------|------|------|---------|
| 1 | **安全底座** | `SecurityConfig.java`, `JwtTokenProvider`, `JwtTokenStore`, `JwtAuthenticationFilter`, `JwtAuthenticationEntryPoint`, `UserDetailsServiceImpl`, `AuthController` | 全 API 需 JWT 认证 | 🔴 **高** |
| 2 | **Swagger/OpenAPI** | `SwaggerConfig.java`, 10 个 Controller `@Tag`/`@Operation`, DTO `@Schema` | API 文档可通过 `/swagger-ui.html` 浏览 | 🟢 低 |
| 3 | **Flyway 迁移** | `V1~V14` 迁移脚本, `application.yml` baseline 配置 | 数据库全模块受 Flyway 管理 | 🟡 中 |
| 4 | **索引优化** | `V9__add_performance_indexes.sql`, `V11__add_missing_indexes.sql` | 38 个新索引 | 🟢 低 |
| 5 | **Figma 精修** | `KycList.vue` 新建, `TransactionHistory.vue` 增强, 路由变更, E2E 重写, i18n 补充 | 前端 UI 对齐 Figma 设计 | 🟡 中 |
| 6 | **审计日志** | `AuditAspect`, `AuditLogAsyncWriter`, `@AuditLog` 注解, `V12__create_audit_log.sql` | 全业务方法异步记录操作审计 | 🟡 中 |
| 7 | **请求耗时监控** | `RequestTimerAspect` | 所有 HTTP 请求耗时日志 | 🟢 低 |

---

## 二、逐个变更深度分析

### 2.1 安全底座 — 🔴 高

#### 变更内容
- JWT 无状态认证体系 (`OncePerRequestFilter`)
- Redis 令牌存储 (`JwtTokenStore`) — 支持吊销/登出
- BCrypt 密码编码
- CORS 全开放（开发阶段）
- CSRF 禁用（API 无状态模式）
- Swagger/OpenAPI 端点白名单

#### 潜在涟漪

| # | 涟漪 | 影响范围 | 触发条件 | 严重度 |
|---|------|---------|---------|-------|
| 1.1 | **未认证用户访问非公开端点 → 401 JSON 而非 HTML 重定向** | 所有前端 HTTP 调用 | 前端未正确处理 401 响应 | ⚠️ 中 |
| 1.2 | **CORS `allowedOriginPatterns: *` + `allowCredentials: true` 组合不安全** | 生产环境安全合规 | 部署到生产环境前未收紧 CORS | 🔴 高 |
| 1.3 | **`JwtTokenStore` 强依赖 Redis 可用性** | 所有 JWT 校验 | Redis 宕机 → `isValidToken()` 返回 false → 所有请求被拒绝 | 🔴 高 |
| 1.4 | **JWT Secret 硬编码在 `application.yml`** | 密钥泄露风险 | 代码仓库访问权限被获取 | 🔴 高 |
| 1.5 | **`jwt.secret` 为固定 Base64 字符串** | 密钥轮换场景 | 需要更换密钥时需协调前后端发布窗口 | 🟡 中 |
| 1.6 | **`UserDetailsServiceImpl` 每次请求加载数据库** | 高频 API 响应延迟 | 高并发场景下每请求一次 DB 查询 | 🟡 中 |
| 1.7 | **`AuthController.logout()` 从 Header 提取 token 但 Swagger UI 可能传参方式不一致** | Swagger Try it out 体验 | 开发者用 Swagger UI 测试登出 | 🟢 低 |
| 1.8 | **Swagger 端点白名单 `/swagger-ui/**` 可能暴露 API 文档至公网** | 生产安全 | 生产环境未加额外访问控制 | 🟡 中 |

#### 建议
1. **生产部署前**收紧 CORS 为具体域名列表，而非 `*`
2. 引入 **Redis 高可用** 或实现 `JwtTokenStore` 的降级策略（本地缓存 + 短 TTL）
3. 密钥改为**环境变量注入**，从 `.env` 或 vault 读取
4. 考虑引入 `@Cacheable` 缓存 `UserDetails`（短 TTL 如 5 分钟）减少 DB 压力
5. 生产环境 `/swagger-ui/**` 应加 IP 白名单或 VPN 保护

---

### 2.2 Swagger/OpenAPI 文档 — 🟢 低

#### 变更内容
- `SwaggerConfig` 全局配置 + 4 个 API 分组
- 10 个 Controller 全部添加 `@Tag`、`@Operation`、`@ApiResponses`
- 端点路径分组清晰（buyer-api / ops-api / auth-api / admin-api）
- 安全方案定义为 `bearerAuth` HTTP Bearer

#### 潜在涟漪

| # | 涟漪 | 影响范围 | 触发条件 | 严重度 |
|---|------|---------|---------|-------|
| 2.1 | **分组路径重叠** — `ops-api` 的 `pathsToMatch` 覆盖了 `/api/v1/kyc/**` 和 `/api/v1/documents/**`，与 `buyer-api` 重叠 | Swagger UI 中同一端点出现在两个分组 | 查看 Swagger UI 分组列表 | 🟢 低 |
| 2.2 | **分组定义中 buyer-api 包含 `/api/v1/kyc/**` 但 KYC 是运营端操作** | API 文档分组语义混乱 | Buyer 开发者查看 buyer-api 文档 | 🟢 低 |
| 2.3 | **`@Schema` 注解完整性** 部分 DTO 可能未覆盖所有字段 | API 文档字段描述缺失 | 通过 Swagger UI 查看 DTO 模型 | 🟢 低 |

#### 建议
1. 细化分组路径：`buyer-api` 应排除 KYC 路径（KYC 是运营端行为）
2. 建议增加 CI 检查：编译期校验所有 DTO 字段是否有 `@Schema`

---

### 2.3 Flyway 迁移脚本 — 🟡 中

#### 变更内容
- Baseline 设置在 V8（V1~V8 为初始化 DDL）
- V9 索引补充 / V10 软删除 / V11 缺失索引 / V12 审计日志 / V13 参考数据 / V14 业务种子数据
- `baseline-on-migrate: true` — 对已有数据库幂等

#### 潜在涟漪

| # | 涟漪 | 影响范围 | 触发条件 | 严重度 |
|---|------|---------|---------|-------|
| 3.1 | **Baseline 版本**设为 8 但 V1~V8 脚本仍然会被 Flyway 扫描到并尝试执行（若表已存在则 `CREATE TABLE IF NOT EXISTS` 安全，但 V9+ 的 `ALTER TABLE ADD COLUMN` 若列已存在会报错） | 从头初始化数据库 | 首次构建或重置数据库 | 🔴 高 |
| 3.2 | **`V10__add_soft_delete_fields.sql`** 使用 `ALTER TABLE ADD COLUMN` 无 `IF NOT EXISTS` 保护 | 重复执行迁移 | Flyway 回滚后重跑 | 🟡 中 |
| 3.3 | **`V12__create_audit_log.sql`** 使用 `CREATE TABLE IF NOT EXISTS` + 动态 SQL 补充索引，但 `SET @dbname = (SELECT DATABASE())` 在 prepared statement 中可能因连接池上下文失效 | 审计日志表索引缺失 | 特殊 MySQL 配置下动态 SQL 执行失败 | 🟡 中 |
| 3.4 | **`V13__seed_reference_data.sql` 和 `V14__seed_business_data.sql`** 种子数据的 `id` 硬编码，若生产环境已有数据则主键冲突 | 生产环境首次部署 | 生产环境存在不同初始数据 | 🔴 高 |
| 3.5 | **`V1~V8` 未使用 `IF NOT EXISTS`** | 重复执行迁移报错 | Flyway 因 chesum 不匹配重跑 | 🔴 高 |

#### 建议
1. V1~V8 应添加 `CREATE TABLE IF NOT EXISTS` 保证幂等（或者严格限制 baseline 只标记已存在的表）
2. V9~V11 的 `ALTER TABLE ADD COLUMN` 和 `ADD INDEX` 改为使用 `IF NOT EXISTS` 模式或 prepared statement 动态检测
3. 种子数据的 id 改为 `INSERT IGNORE` + 依赖 `SELECT id` 子查询获取 ID，避免硬编码
4. 考虑添加 `checksum` 验证步骤，确保迁移脚本不会被意外修改

---

### 2.4 索引优化 — 🟢 低

#### 变更内容
- V9: 7 个新索引（复合索引覆盖排序 + 状态）
- V11: 38 个新索引（`updated_at` 排序、`status` 过滤、JOIN 字段、历史表 `history_at`）
- 覆盖高频查询模式: `WHERE fact_id=? AND detail_type=?`, `WHERE account_id=? ORDER BY created_at DESC`

#### 潜在涟漪

| # | 涟漪 | 影响范围 | 触发条件 | 严重度 |
|---|------|---------|---------|-------|
| 4.1 | **新增 38 个索引对写入性能的影响** | 所有 INSERT/UPDATE/DELETE | 高并发写入场景 | 🟢 低 |
| 4.2 | **`oms_*_detail` 表的 `idx_fact_detail_type` 复合索引与已有单列索引 `idx_fact_id` 冗余** | 索引维护成本 | 数据库分析慢查询时 | 🟢 低 |
| 4.3 | **`V9` 和 `V11` 对同一张表 `oms_tt_payment` 分别添加了 `idx_created_at` 和 `idx_updated_at`** — 两张独立索引，复合查询无法同时利用 | `ORDER BY created_at DESC, updated_at DESC` | 混合排序查询 | 🟢 低 |

#### 建议
1. 监控写入性能基线，与索引添加前对比
2. 可考虑清理 `idx_fact_id` 单列索引（被复合索引覆盖）
3. 若常见排序模式为 `ORDER BY updated_at DESC, created_at DESC`，可考虑复合索引

---

### 2.5 Figma 精修 — 🟡 中

#### 变更内容
- **KYC 列表页** (`KycList.vue`) — 完全新建，对齐 Figma 151636
- **CDP 交易历史** (`TransactionHistory.vue`) — UI 增强 + i18n
- **路由变更** — 根路径重定向改为 `/kyc`（非 `/kyc/new`），新增编辑路由
- **E2E 测试套件** — 3 个文件完全重写覆盖 7 大模块 + 三大链路
- **i18n 补充** — KYC 状态翻译 + CDP 交易历史翻译

#### 潜在涟漪

| # | 涟漪 | 影响范围 | 触发条件 | 严重度 |
|---|------|---------|---------|-------|
| 5.1 | **根路径 `/` 重定向从 `/kyc/new` 改为 `/kyc`** — 旧书签/URL 使用 `/kyc/new` 的用户会看到列表页而非表单 | 用户习惯 | 用户刷新或通过书签访问 | 🟡 中 |
| 5.2 | **KYC 路由新增 `/kyc/:id` 和 `/kyc/:id/edit`** — 与原有 `/kyc/new` 路由可能冲突（Vue Router 路径匹配优先级） | 路由解析 | 访问 `/kyc/new` 时可能匹配到 `/kyc/:id` 模式 | 🔴 高 |
| 5.3 | **侧边栏 KYC 菜单项从 `/kyc/new` 改为 `/kyc`** — 导航入口变更 | 运营端导航 | 用户点击侧边栏 KYC 按钮 | 🟡 中 |
| 5.4 | **E2E 测试全面重写** — 若 Figma 设计后续变更，测试需同步更新维护成本高 | 测试维护 | Figma 设计迭代 | 🟢 低 |

#### 建议
1. **立即验证路由冲突**：Vue Router 中 `/kyc/new` 和 `/kyc/:id` 的注册顺序 — `/kyc/new` 必须在 `/kyc/:id` 之前，否则 `new` 会被当作 `:id`
   - 当前 router/index.js 中顺序：`/kyc` (KycList) → `/kyc/new` (KycNew) → `/kyc/:id` (KYCForm) → `/kyc/:id/edit` (KYCFormEdit) — ✅ 顺序正确
2. 旧书签跳转 `/kyc/new` 的用户应重定向到 `/kyc`（可在路由 guard 中处理）

---

### 2.6 审计日志 — 🟡 中

#### 变更内容
- `@AuditLog(action, module)` 注解 + `AuditAspect` AOP 切面
- `AuditLogAsyncWriter` 异步写入 `oms_audit_log` 表
- V12 创建审计日志表（含 5 个索引）
- 旧 `AuditLogAspect`（application 层）和 `AuditLoggable` 注解标记为 `@Deprecated(forRemoval)`

#### 潜在涟漪

| # | 涟漪 | 影响范围 | 触发条件 | 严重度 |
|---|------|---------|---------|-------|
| 6.1 | **`@Async` 异步写入** — 若 `@EnableAsync` 未在启动类配置，`write()` 方法退化为同步调用 | 请求响应延迟增加 | 审计日志写入耗时较长时 | 🟡 中 |
| 6.2 | **审计日志异步写入失败静默吞异常**（仅打 WARN 日志） | 操作追溯能力 | 审计日志入库失败但无告警 | 🟡 中 |
| 6.3 | **`AuditAspect` 拦截 `@AuditLog` 注解方法** — 但当前控制器代码中**未看到实际使用该注解** | 审计日志功能未生效 | 所有业务操作 | 🔴 高 |
| 6.4 | **旧 `AuditLogAspect` 和 `AuditLoggable` 标记 `@Deprecated(forRemoval=true)`** 但仍在类路径中 | 代码混淆 | 开发者误用旧注解 | 🟢 低 |
| 6.5 | **`oms_audit_log` 表 append-only** — 无数据归档策略，长期运行后表数据膨胀 | 查询性能、存储成本 | 系统运行数月后 | 🟡 中 |
| 6.6 | **`detailsJson` 字段可能包含敏感信息（如密码、Token）** | 数据安全合规 | 审计日志被导出分析 | 🔴 高 |

#### 建议
1. 确认启动类配置了 `@EnableAsync` — 检查 `TradeOMSApplication.java` 或 `@SpringBootApplication` 主类
2. **在所有 Controller 的写操作方法上添加 `@AuditLog` 注解**，当前代码中仅 `AuditAspect` 存在但没有被任何 Controller 使用
3. 增加审计日志归档策略（按月/季度自动归档至历史表）
4. `detailsJson` 序列化时**过滤敏感字段**（密码、Token、身份证号等）

---

### 2.7 请求耗时监控 — 🟢 低

#### 变更内容
- `RequestTimerAspect` 拦截所有 `@RequestMapping` 方法
- 三级日志: INFO(<500ms) / WARN(500~1999ms) / ERROR(>=2000ms)

#### 潜在涟漪

| # | 涟漪 | 影响范围 | 触发条件 | 严重度 |
|---|------|---------|---------|-------|
| 7.1 | **切面只拦截 `@RequestMapping`** — `@GetMapping`、`@PostMapping` 等是 `@RequestMapping` 的派生注解，Spring AOP 按注解类型匹配时默认只匹配**直接标注**的注解 | 所有 Controller 方法 | 请求到达未被 `@RequestMapping` 直接标注的方法 | 🔴 高 |
| 7.2 | **`@Around` 切面内部使用 `RequestContextHolder`** — 在异步/定时任务场景下可能获取不到请求上下文 | 定时任务抛出 NPE | 定时任务中直接调用 Service 方法 | 🟢 低 |
| 7.3 | **日志输出级别基于耗时** — 开发环境大量请求超 500ms（含数据库慢查询预热期）会产生过多 WARN 日志 | 日志噪音 | 开发/测试环境启动初期 | 🟢 低 |

#### 建议
1. **⚠️ 重要**：修改切点表达式为 `@annotation(org.springframework.web.bind.annotation.RequestMapping) || @annotation(org.springframework.web.bind.annotation.GetMapping) || ...` 或使用 Spring 的 `@RequestMapping` 类级别组合匹配（推荐用 `execution(* com.tradeoms.interfaces.rest..*.*(..))` 模式匹配）
2. 在切面内对 `RequestContextHolder.getRequestAttributes()` 做 null 安全判断（已做 ✅）

---

## 三、跨变更耦合分析

### 3.1 安全底座 × Swagger

```
SecurityConfig 白名单 Swagger 路径
  └── /v3/api-docs/**, /swagger-ui/**, /swagger-ui.html
      └── 生产环境应加 IP 白名单或移除
```

**风险**: Swagger 文档端点在生产环境暴露 API 结构，配合 JWT 认证方式公开在文档中，攻击者可获知精确的认证方式。

**建议**: 生产环境移除 Swagger 白名单或替换为 `@Profile("dev")` 条件加载。

### 3.2 Flyway × 索引 × 审计日志

```
V12 创建 oms_audit_log 表
  └── AuditAspect 异步写入
      └── 若 V12 未执行成功 → 审计日志写入抛异常（已 catch）
```

**依赖链**: 审计日志功能强依赖 V12 迁移成功。但因异步写入异常已 catch 并打 WARN，不会阻断主流程。

**风险**: 审计日志无声丢失。

### 3.3 安全底座 × 审计日志

```
JwtAuthenticationFilter 设置 SecurityContextHolder
  └── AuditAspect 从 SecurityContext 提取用户
      └── 若认证失败（401）则提取为 "anonymous"
```

**合规性**: 未认证请求的审计日志记录为 `anonymous`，覆盖了 `/api/v1/auth/**` 的登录/注册端点。

### 3.4 Figma 精修 × 前端路由 × 后端 API

```
Figma 151636 (KYC 列表页)
  └── KycList.vue 新建
      └── 依赖后端 GET /api/v1/kyc/applications
          └── 需 @AuditLog 注解补充
```

**依赖链**: 前端新页面依赖后端 API 稳定可用。若 API 变更需同步更新前端。

### 3.5 请求耗时监控 × Flyway 预热

```
RequestTimerAspect 记录慢请求
  └── Flyway V9~V11 新增 45 个索引
      └── 首次启动时索引构建 + 查询缓存未预热
          └── 启动初期大量慢请求告警
```

**运营噪音**: 系统重启后的预热期会产生大量 ERROR 级别慢请求日志。

---

## 四、综合评价

### 4.1 需立即修复 (Priority P0-P1)

| # | 问题 | 责任域 | 建议操作 |
|---|------|--------|---------|
| P0 | **[6.3] `@AuditLog` 注解未被任何 Controller 使用** | 审计日志 | 在所有 Controller 写操作方法上添加 `@AuditLog` |
| P1 | **[7.1] `RequestTimerAspect` 切点可能漏拦截派生注解** | 请求监控 | 将切点改为 `execution(* com.tradeoms.interfaces.rest..*.*(..))` |
| P1 | **[1.3] `JwtTokenStore` 无 Redis 降级策略** | 安全底座 | 实现本地缓存兜底（如 Caffeine + 短 TTL） |
| P1 | **[1.4] JWT Secret 硬编码于 `application.yml`** | 安全底座 | 改用环境变量 `${JWT_SECRET}` |
| P1 | **[5.2] Vue Router 路由顺序风险** | Figma 精修 | 验证 `/kyc/new` 在 `/kyc/:id` 之前注册（当前 ✅ 已正确） |

### 4.2 需尽快处理 (Priority P2)

| # | 问题 | 责任域 | 建议操作 |
|---|------|--------|---------|
| P2 | **[3.1-3.5] Flyway 迁移脚本幂等性不足** | Flyway | 所有 DDL 改为 `IF NOT EXISTS` |
| P2 | **[1.2] CORS 全开放** | 安全底座 | 生产配置限制具体域名 |
| P2 | **[1.8] Swagger 端点公网暴露** | Swagger | 用 `@Profile("dev")` 条件化 SwaggerConfig |
| P2 | **[6.6] 审计日志可能记录敏感信息** | 审计日志 | 增加敏感字段过滤器 |

### 4.3 建议跟踪 (P3)

| # | 问题 | 责任域 |
|---|------|--------|
| P3 | [2.1-2.2] Swagger 分组路径重叠导致语义混淆 | Swagger |
| P3 | [4.1] 45 个新索引对写入性能的影响需监控 | 索引 |
| P3 | [6.5] 审计日志归档策略 | 审计日志 |
| P3 | [7.3] 启动预热期日志噪音 | 请求监控 |

### 4.4 架构健康度总结

```
模块                成熟度    风险暴露面
─────────────────────────────────────────────
安全底座            🟡 可用    认证/密钥/Redis 强依赖/CSRF+CORS
Swagger/OpenAPI     🟢 完整    分组语义需微调
Flyway              🟡 可用    幂等性需加固
索引优化            🟢 完整    🔍 需监控写入性能
Figma 精修          🟢 完成    路由冲突已验证无问题
审计日志            🔴 未生效   `@AuditLog` 注解未投入使用
请求耗时监控        🟡 有风险   切点表达式可能漏拦截
```

---

## 五、总结

Batch 10 是一次**基础设施层的大规模整合**：安全底座奠定了 JWT 认证框架，Swagger 补齐了全模块 API 文档能力，Flyway 统一了数据库版本管理，索引优化提升了查询性能，Figma 精修落地了前端 UI 设计，审计日志和请求监控建立了可观测性基础。

**最大的潜伏风险在于「审计日志功能未生效」**（`@AuditLog` 注解未被实际使用）和「安全底座密钥硬编码 + Redis 强依赖」。这两个问题一个导致功能不可达，一个导致生产可靠性隐患。

**推荐的立即行动**：
1. 在所有 Controller 写操作端点添加 `@AuditLog` 注解
2. 将 JWT Secret 移入环境变量
3. 为 JwtTokenStore 增加本地缓存降级

---

*本文档基于以下源码审查完成：*
- [SecurityConfig.java](../backend/src/main/java/com/tradeoms/boot/config/SecurityConfig.java)
- [SwaggerConfig.java](../backend/src/main/java/com/tradeoms/boot/config/SwaggerConfig.java)
- [AuditAspect.java](../backend/src/main/java/com/tradeoms/infrastructure/audit/AuditAspect.java)
- [RequestTimerAspect.java](../backend/src/main/java/com/tradeoms/infrastructure/config/RequestTimerAspect.java)
- [V9__add_performance_indexes.sql](../backend/src/main/resources/db/migration/V9__add_performance_indexes.sql)
- [V10__add_soft_delete_fields.sql](../backend/src/main/resources/db/migration/V10__add_soft_delete_fields.sql)
- [V11__add_missing_indexes.sql](../backend/src/main/resources/db/migration/V11__add_missing_indexes.sql)
- [V12__create_audit_log.sql](../backend/src/main/resources/db/migration/V12__create_audit_log.sql)
- [V13__seed_reference_data.sql](../backend/src/main/resources/db/migration/V13__seed_reference_data.sql)
- [V14__seed_business_data.sql](../backend/src/main/resources/db/migration/V14__seed_business_data.sql)
- [KycList.vue](../frontend/src/views/kyc/KycList.vue)
- [router/index.js](../frontend/src/router/index.js)
- [figma-links.md](../07-ui-figma/figma-links.md)
- [figma-screenshots-inventory.md](../07-ui-figma/figma-screenshots-inventory.md)
