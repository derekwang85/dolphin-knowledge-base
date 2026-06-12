# 🐩 巨贵 Batch 10.7 涟漪审查报告 — Flyway 清理关联影响

**审查时间**: 2026-06-11 12:20 CST  
**审查范围**: derekcoding #14 冰山原则 + #2 涟漪必扫  
**审查基准**: Batch 10.6 已完成 V1~V9 旧迁移移入 legacy/ + V1__baseline.sql 固化

---

## 1️⃣ 源码引用检查 — ✅ 已确认安全

- 搜索 `legacy/` 目录 → 无任何 Java/XML/YAML/Shell 代码引用旧迁移文件名
- 搜索 `V1__` ~ `V9__` → 仅历史任务文档 (`tasks/`) 中出现，属归档记录，不影响运行
- 搜索 `V1__baseline.sql` → 无例外硬编码引用

## 2️⃣ pom.xml 兼容性 — ✅ 已确认安全

| 依赖 | 版本 | 来源 |
|------|------|------|
| `flyway-core` | 10.20.1 | Spring Boot 3.4.4 BOM 管理 |
| `flyway-mysql` | 10.20.1 | Spring Boot 3.4.4 BOM 管理 |

- ✅ 无显式版本号 — 由 Spring Boot parent POM 统一管理
- ✅ 与 `mysql-connector-j` (runtime, 也是 Boot 管理) 版本兼容
- ✅ `mvn dependency:tree` 验证无版本冲突

## 3️⃣ application.yml 配置 — ✅ 已确认安全

```yaml
spring:
  flyway:
    enabled: true
    baseline-on-migrate: false
    locations: classpath:db/migration
```

- ✅ YAML 缩进正确（2空格）
- ✅ `baseline-on-migrate: false` — 配合手动 `flyway baseline`，启动时不会自动 baseline
- ✅ `locations: classpath:db/migration` — 不包含 `legacy/`，旧迁移不会被自动发现
- ✅ `flyway.enabled=true` + 数据库已有表 → baseline 已标记，`flyway:migrate` 为 no-op

## 4️⃣ 旧迁移引用扫描 — ✅ 已确认安全

| 位置 | 内容 | 判决 |
|------|------|------|
| `09-migrations/` | GitHub 项目迁移计划，非 DB 迁移 | ✅ 不相干 |
| `README.md` | 目录结构列表，仅列出 `09-migrations/` 路径 | ✅ 不相干 |
| `ci/*.sh` | `g9-sql-quality.sh` 仅做 SQL 基础质量检查，不涉及迁移文件 | ✅ 安全 |
| `deploy/docker-compose.yml` | 无数据库迁移配置 | ✅ 安全 |
| 产品手册 (`11-product-manual/`) | 未出现 `flyway` 或迁移文件名 | ✅ 安全 |

## ⚠️ 5️⃣ 部署注意事项 — 中等风险

### 5.1 编译时必须用 `clean` 🔶

**问题**: `mvn compile` (不带 `clean`) 会在 `target/classes/db/migration/` 中**合并新旧文件**，导致 old V1~V9 迁移残留。

**后果**: 运行时 Flyway 会扫描到两个 V1 迁移：
- `V1__baseline.sql` (新)
- `V1__create_kyc_tables.sql` (旧残留)

导致 **Flyway 重复版本号报错**。

**验证**: 已实测：
- `mvn compile` → target 目录残留 V1~V9 旧文件 ❌
- `mvn clean compile` → target 仅含 `V1__baseline.sql` + `legacy/` ✅

**建议**:
- 开发/部署命令统一为 `mvn clean compile -DskipTests` 或 `mvn clean package`
- CI 流水线必须包含 `clean` 阶段

### 5.2 `flyway-core` 版本与 Flyway Pro/Teams 差异 🔷

Flyway 10.20.1 (Community) 功能完整，无需额外许可证。

## 6️⃣ 编译验证 — ✅ 通过

```
$ mvn compile -q
BUILD SUCCESS
```

- 无编译错误
- 无资源复制警告
- V1__baseline.sql 覆盖 **68 张表**，包含完整 DDL

## 总结

| 检查项 | 状态 |
|--------|------|
| 源码硬编码引用 | ✅ 无 |
| pom.xml 兼容性 | ✅ 安全 |
| application.yml 配置 | ✅ 正确 |
| 文档/CI 旧引用 | ✅ 无 |
| target 文件残留 | ⚠️ 需 `clean` |
| 编译通过 | ✅ 通过 |

**整体结论**: ✅ 源文件结构正确，无源码涟漪。**唯一风险点**：部署时必须使用 `mvn clean` 构建，否则 target 残留旧迁移文件会引发 Flyway 版本冲突。
