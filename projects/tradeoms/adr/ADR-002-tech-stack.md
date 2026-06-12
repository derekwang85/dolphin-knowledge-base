# ADR-002: 技术栈选型

> **状态**: 已接受 | **日期**: 2026-06-09

---

## 背景

原方案（PDF 中的 OntOS 六层架构）技术栈过于复杂：
- Neo4j + GaussDB + Jena + HermiT + Pellet + Drools + TensorFlow + LLM + LangChain + Kong + GraphQL Federation

需要简化至一个普通 Java 团队可维护的水平。

## 决策

**采用精简技术栈**：

| 组件 | 选择 | 代替原方案中的 |
|------|------|--------------|
| 应用框架 | Java + Spring Boot 3 | LangChain, OpenCode |
| 数据库 | MySQL 8.0 | GaussDB, Jena RDF |
| 缓存 | Redis 7 | - |
| 向量检索 | Qdrant | Neo4j 本体图 |
| 文件存储 | MinIO | - |
| AI 交互 | OpenClaw | 自定义 Agent 框架 |
| LLM 推理 | 外部 LLM API | HermiT, Pellet, TensorFlow |
| 规则引擎 | Java 策略模式 | Drools |
| API | REST (Spring Cloud Gateway) | Kong, GraphQL Federation |

## 选型理由

- **Java**: 与 CTRM 体系技术栈一致，大宗交易需要强类型保障
- **MySQL**: 够用，运维成本最低
- **Qdrant**: 轻量级向量引擎，部署简单，满足语义搜索需求
- **OpenClaw**: 已有成熟基础设施，WeCom 集成 + LLM 编排
- **Java 策略模式替代规则引擎**: 代码即文档，IDE 可追踪，调试友好

## 舍弃但保留接入点的技术

| 技术 | 状态 | 何时可能启用 |
|------|------|-------------|
| Neo4j 图数据库 | ⏸️ 暂缓 | 客户关系网络分析需求明确后 |
| 消息队列 | ⏸️ 暂缓 | CTRM 同步出现性能瓶颈后 |
| Drools 规则引擎 | ❌ 永久 | Java 策略模式足矣 |
