# ADR-011: JWT 认证架构

> **状态**: 🟢 Accepted | **日期**: 2026-06-12 | **关联**: ADR-002

---

## 背景

TradeOMS 需要一套统一的认证体系，覆盖 Buyer 端（Web 浏览器）、运营端（Web 浏览器）和 OpenClaw Agent（API 调用）。需求：

- 无状态认证（Token 自包含 claims）
- Token 可撤销（登出/强制下线）
- 多端并发登录，支持切换
- Spring Security 原生集成

## 决策

**采用 JWT (HMAC-SHA256) + Redis Token 持久化 + Spring Security Filter 的三层架构**。

### 架构总览

```
Client (浏览器/OpenClaw)
     │
     │ POST /api/v1/auth/login {username, password}
     ▼
AuthController
     │
     │ 校验用户名密码 → 生成 JWT
     ▼
JwtTokenProvider (HMAC-SHA256)
     │
     │ jwt:active:{token}  →  {userId}
     │ jwt:user:{userId}:tokens  →  Set<token>
     ▼
JwtTokenStore (Redis)
     │
     │ 返回 JWT → 客户端存储 (localStorage)
     ▼
Client
     │
     │ 后续请求: Authorization: Bearer <token>
     ▼
JwtAuthenticationFilter (OncePerRequestFilter)
     │
     │ 1. 提取 Bearer token
     │ 2. JwtTokenProvider.validateToken() — 签名+过期校验
     │ 3. JwtTokenStore.isValidToken() — Redis 活跃校验
     │ 4. 设置 SecurityContextHolder
     ▼
API 端点 (@PreAuthorize 控制)
```

### 1. JwtTokenProvider

| 功能 | 实现 |
|------|------|
| 签名算法 | HMAC-SHA256 (io.jsonwebtoken + javax.crypto) |
| Secret 来源 | `jwt.secret` (application.yml / 环境变量) |
| 默认过期 | 24h (86400000ms)，通过 `jwt.expiration-ms` 可配置 |
| Claims | `sub=username`, `user_id`, `role`, `iat`, `exp` |
| 生成 | `generateToken(userId, username, role)` |
| 验证 | `validateToken(token)` — 签名验证 + 过期验证 |
| 刷新 | `refreshToken(token)` — 用旧 token 的 claims 生成新 token |
| Claims 提取 | `getUsername()`, `getUserId()`, `getRole()` |

### 2. JwtTokenStore (Redis)

**Redis Key 设计**：

| Key | 类型 | 用途 | TTL |
|-----|------|------|-----|
| `jwt:active:{jwt}` | String (value=userId) | 活跃令牌校验 | 与 JWT exp 对齐 |
| `jwt:user:{userId}:tokens` | Set (members=jwt) | 用户当前令牌集合 | 与最久 token 对齐 |

**操作**：
- `storeToken(token, userId, username)` — 登录成功后存储
- `isValidToken(token)` — 每次请求校验（双重验证：JWT 签名 + Redis 存在性）
- `revokeToken(token, userId)` — 登出
- `revokeAllUserTokens(userId)` — 强制下线（密码修改/安全事件）

### 3. JwtAuthenticationFilter

- 继承 `OncePerRequestFilter`，确保每个请求只执行一次
- 从 `Authorization: Bearer <token>` 提取 JWT
- 执行两层校验：签名验证 + Redis 活跃检验
- 校验通过后设置 `SecurityContextHolder`
- 校验失败或缺失 Token → 请求继续到 `JwtAuthenticationEntryPoint`

### 4. JwtAuthenticationEntryPoint

- 处理 `AuthenticationException` — 返回 401 Unauthorized
- 统一 JSON 响应格式: `{"code": 401, "message": "Unauthorized", "data": null}`

### 5. UserDetailsServiceImpl

- 实现 `UserDetailsService` 接口
- `loadUserByUsername(username)` — 从 `oms_user` 表加载用户 + 角色
- 角色映射: `User → Set<SimpleGrantedAuthority>`

## 安全设计原则

| 原则 | 说明 |
|------|------|
| 签名不可伪造 | HMAC-SHA256, SecretKey 仅服务端持有，环境变量注入 |
| 双因子校验 | 签名验证（防篡改） + Redis 存在性（防已撤销令牌） |
| 自动过期 | JWT exp + Redis TTL 对齐，过期自动失效 |
| 可撤销 | 登出/强制下线直接删除 Redis 记录 |
| 用户隔离 | `jwt:user:{userId}:tokens` 支持查询和管理指定用户的所有令牌 |
| CSRF 无状态 | Bearer Token 方式，不需要 CSRF Token |

## 影响

- **正向**：无状态认证 + 可撤销能力，兼顾安全性和扩展性
- **约束**：依赖 Redis（TokenStore 不可用则所有请求 401），需保证 Redis 高可用
- **性能**：每次请求两次 Redis read（校验活跃性），QPS 受 Redis 吞吐限制
- **风险**：JWT payload 未加密（claims 明文可见），不存放敏感信息
