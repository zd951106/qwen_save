---
name: 实现代码时添加日志方便调试
description: 编写业务代码时在关键节点（方法入口/外部调用/异常捕获/状态变更）添加日志，方便测试和线上排查
type: feedback
---

编写实现代码时，遵循以下日志规范，方便测试调试和线上排查：

**核心规则**：

1. **入参日志是第一行可执行代码**：方法入口立即打印，无论是否有参数，确保能看到方法被调用
   - Controller：`log.info("::methodName req:{}", JSONObject.toJSONString(req))`
   - Service：`log.info("::methodName req:{}", JSONObject.toJSONString(req))`
   - 无参方法：`log.info("::methodName")` — 也要打印入口标记

2. **响应日志与请求日志分行**：方法返回前打印，req 和 res 分开两条日志
   - Controller：`log.info("::methodName res:{}", JSONObject.toJSONString(res))`
   - Service：`log.info("::methodName resSize:{}", size)` 或 `log.info("::methodName res id:{}", id)`

3. **Controller 禁止 void 返回**：所有 Controller 方法必须返回响应体结构（RespBody/ResBody），不能返回 void（文件导出除外）

4. **Service 所有方法必须有返回值**：禁止 void，至少返回影响行数或 ID

5. 数量/统计操作后记录数量：`log.info("::methodName count:{}", total)`

6. **数据库查询结果每条必须记录**：Mapper/DAO 调用后立即打印查询结果，方便定位数据源头问题
   - 列表查询：`log.info("::methodName db result count:{}", list.size())`
   - 单条查询：`log.info("::methodName db Mapper简称 key:{} result:{}", key, JSONObject.toJSONString(result))`
   - 写入操作：`log.info("::methodName db insert id:{}", entity.getId())`
   - 预期空值：`log.warn("::methodName db 未找到 key:{}", key)`

7. 异常：`log.error("业务描述", e)` — 含完整堆栈

8. 预期失败（非异常）：`log.warn("::methodName 失败原因 key:{}", value)`

9. 数据完整打印：日志必须完整序列化数据（用 JSONObject.toJSONString），不只打印单个字段

**Why:** 用户多次强调日志的重要性——缺少入参日志无法确认方法是否被调用，缺少响应日志无法判断返回结果，只打印部分字段无法排查数据问题。

**How to apply:**
- Java 项目：`log.info("::methodName req:{}", JSONObject.toJSONString(req))` + `log.info("::methodName res:{}", JSONObject.toJSONString(res))`
- Vue 项目：`console.log` 开发调试用，生产环境用统一日志工具
- 不要记录敏感信息（密码、密钥、完整卡号）
- 日志级别：info 用于关键流程节点，warn 用于预期失败，error 用于异常
- 所有 Controller 和 Service 方法全覆盖，无遗漏
