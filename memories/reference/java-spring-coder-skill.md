---
name: Java/Spring编码规范技能参考
description: 组长提供的java-spring-coder技能及4份参考文档，编码阶段必须遵循，跨项目通用
type: reference
---

组长提供的 Java/Spring Boot 编码规范，保存在 `C:\Users\admin\.qwen\skills\java-spring-coder\`，跨项目通用。

**Why:** 不同项目、不同模块的编码规范存在差异化（统一返回封装类、基础父类、包结构、命名规则），不能直接套用通用模板。这些规范在编码阶段必须遵循。

**How to apply:**
- 技能主文件：`SKILL.md` — 编码前先梳理项目上下文、区分通用稳定规则与项目个性化规则
- 参考文档：
  - `references/java-coding-standards.md` — 命名/不可变性/Optional/Stream/DI/异常/项目结构/日志/配置/测试
  - `references/java-spring-data-access.md` — 对象转换/数据库表设计/批量处理/事务控制/空安全
  - `references/java-spring-runtime-topics.md` — 并发/缓存/配置安全/异步/MQ事务/安全/性能/设计模式
  - `references/java-spring-quality-and-delivery.md` — Apifox接口文档/测试规范/代码评审Checklist
- 已写入 `C:\Users\admin\.qwen\QWEN.md` 全局指令，与需求设计规范叠加生效
- 规范一（需求设计规范）管"做什么、怎么设计"，规范二（本技能）管"怎么写代码"
- 编码时也可通过 `skill: "java-spring-coder"` 主动调用技能
