# ADR-012: 前端 API 拦截器架构

> **状态**: 🟢 Accepted | **日期**: 2026-06-12 | **关联**: ADR-011

---

## 背景

TradeOMS Buyer 端和运营端均为单页应用（Vue 3 + Axios），需要统一管理：

- 每个请求自动携带 JWT Token
- 统一的 API 响应解包（unwrap `ApiResponse<T>` 包装类）
- 统一的 401 处理（Token 过期/无效 → 清除本地 Token → 跳转登录页）
- 避免 401 重定向循环
- 复用性强（多个 store 文件共用同一个 Axios 实例）

## 决策

**采用单 Axios 实例 + 双拦截器（Request + Response）架构**，统一在 `frontend/src/store/api.js` 中管理。

### 架构总览

```
Vue 组件 / Store
     │
     │ import { api, createKYCApplication, ... }
     ▼
api.js (单 Axios 实例)
     │
     ├─ Request Interceptor
     │    └─ localStorage.getItem('token') → Bearer 注入
     │
     ├─ HTTP 请求
     │
     └─ Response Interceptor
          ├─ 成功: res.data → unwrap() → 返回 body.data
          └─ 失败: 401 → clear token → redirect to /login
```

### 1. Request Interceptor — Token 注入

```javascript
api.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('token')
    if (token) {
      config.headers['Authorization'] = 'Bearer ' + token
    }
    return config
  },
  (error) => Promise.reject(error)
)
```

**设计决策**：
- token 从 `localStorage` 读取，而非 sessionStorage（持久化登录态，浏览器关闭不丢失）
- 无 token 时直接发送（公共端点如 `/auth/login` 不需要 token）
- 不缓存 token（每次请求都读取 localStorage，避免闭包缓存导致过期 token 泄漏）

### 2. Response Interceptor — 统一解包 + 401 处理

```javascript
api.interceptors.response.use(
  (res) => res,  // 成功响应由 unwrap() 处理
  (err) => {
    // 401 → 清除 Token → 跳转登录页
    if (status === 401) {
      localStorage.removeItem('token')
      localStorage.removeItem('userRole')
      if (!onLoginOrRegisterPage()) {
        window.location.hash = '#/login'
      }
    }
    return Promise.reject(err)
  }
)
```

### 3. Unwrap 函数 — 标准化响应解包

```javascript
function unwrap(res) {
  const body = res.data
  if (body && typeof body.code === 'number') {
    if (body.code >= 200 && body.code < 300) return body.data
    throw new Error(body.message || `API error code: ${body.code}`)
  }
  return body
}
```

**设计决策**：
- 后端统一返回 `ApiResponse<T>` 格式: `{code, message, data}`
- `unwrap()` 在调用侧按需使用，而非拦截器自动执行
- 让每个 store 文件显式调用 `unwrap()`，保持透明性

### 4. 路由守卫 — 前端路由级保护

每个 store 模块（`kyc-api.js`, `quote-api.js`, `order-api.js` 等）共用同一个 `api` 实例，不重复创建 Axios 实例。

路由层在 `vue-router` 的 `beforeEach` 守卫中检查 token 存在性：

```javascript
router.beforeEach((to, from, next) => {
  const token = localStorage.getItem('token')
  if (to.meta.requiresAuth && !token) {
    next('/login')
  } else {
    next()
  }
})
```

### 5. 401 防重定向循环

在 Response Interceptor 中检查当前路径：

```javascript
const path = window.location.hash.replace(/^#\//, '/') || window.location.pathname
if (!path.includes('/login') && !path.includes('/register')) {
  window.location.hash = '#/login'
}
```

**理由**：当 401 发生在 login 页面本身（如 token 已过期但仍尝试调用非认证端点）时，不应该再重定向到 login 造成循环。

## 影响

- **正向**：
  - 单一入口点，所有 API 调用统一认证
  - 401 自动处理，前端代码中不需要重复写 try-catch-401
  - 路由守卫 + 拦截器双保险，确保未认证用户无法访问受保护页面
- **约束**：
  - 依赖 `localStorage`（XSS 风险），不使用 HttpOnly Cookie（SPA 跨域考量）
  - 多 Tab 同步不自动处理（一个 Tab 登出后其他 Tab 不会立即感知）
- **多端复用**：API store 文件结构支持 Buyer/Operations 共用同一套 Axios 实例
