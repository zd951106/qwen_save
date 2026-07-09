---
name: Cross-project SDK testing pattern
description: User works across open-sdk-java (client) and wpgj-pay (server) projects — writes SDK integration tests in open-sdk-java that verify wpgj-pay's OpenSaaS API validation and business logic
type: user
---

用户经常在 open-sdk-java 项目中编写集成测试，验证 wpgj-pay 服务端的 OpenSaaS 接口。

**典型工作流**：
1. 在 wpgj-pay 中分析服务端适配器的参数校验逻辑（`validateQueryReq`）、业务流程（`invoke` 方法）、错误码
2. 在 open-sdk-java 中编写 SDK 测试，通过 `client.post("/opensaas/route", ...)` 调用真实接口
3. 测试覆盖：正常流程 + 参数校验异常 + 业务异常 + 边界值 + 并发
4. 如果发现服务端 bug，在 wpgj-pay 中修复，同步更新 open-sdk-java 的测试预期

**涉及的 OpenSaaS 接口**（method 路由键）：
- `unify.online.trade.order.query` — 统一网关订单查询
- `ysepay.online.trade.order.query` — 业务网关订单查询
- `unify.trade.refund.query` — 统一网关退款查询
- `ysepay.online.trade.refund.query` — 业务网关退款查询

**How to apply**: 当用户提到"测试"、"查询"、"退款查询"等关键词时，优先关联这两个项目的协作关系。分析服务端代码时读 wpgj-pay，写测试时写 open-sdk-java。
