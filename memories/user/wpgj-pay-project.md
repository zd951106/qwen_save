---
name: wpgj-pay project reference
description: Cross-project pointer to the wpgj-pay payment-gateway project — absolute path, module structure, and test conventions for generating tests from other IDEA project windows
type: user
---

用户在多个 IDEA 项目窗口中并行工作,需要在其他项目中也能为 wpgj-pay 生成测试用例。以下是跨项目操作所需的关键信息。

## 项目路径

`F:\code\ysh\code\wpgj-pay`

## 基本信息

- **项目名**: wpgj-pay (旺铺管家扫码交易系统)
- **groupId**: `com.wpgjpay`, **Java 21**, **Spring Boot 2.7.18**, Spring Cloud 2021.0.9
- **多模块 Maven**: pay-action(含 pay-action-web 子模块,服务入口)、pay-common(通用)、pay-consoumer(消息处理)、pay-core(业务核心)、pay-gateway(支付网关)、pay-dal(数据访问)、pay-opensaas(OpenSaaS 开放平台)
- **根包**: `com.shiyi.posx.gateway`

## 测试约定(生成测试时遵循)

- **框架**: JUnit 4 + spring-boot-starter-test,**不使用 Mockito**,以集成测试为主(连真实环境)
- **BaseTest 基类**(各模块不同):
  - pay-gateway → `com.shiyi.BaseTest`(`@RunWith(SpringJUnit4ClassRunner.class)` + `@SpringBootTest(classes = {BootApplication.class})` + `@ActiveProfiles("prd_k8s")`,含 getRandom/getBindKey/getPaymentKey 工具方法)
  - pay-consoumer → `com.shiyi.posx.gateway.BaseMqTest`(同上但 `@ActiveProfiles("dev")`)
  - pay-action-web → `BaseTest`(无 Spring 注解,纯单元测试)
- **风格**: `@Slf4j` 日志;`@Resource`/`@Autowired` 注入;`org.junit.Assert` 断言;Controller 测试用 MockMvc + WebApplicationContext(`MockMvcBuilders.webAppContextSetup(context).build()`)
- **命名**: 测试方法用场景描述如 `refund1()`、`refund2()`;中文注释描述场景;`@author yu` 头注释
- **测试目录**: `src/test/java/com/shiyi/`(pay-gateway 根级) 或 `src/test/java/com/shiyi/posx/gateway/`(按包结构)

## 如何在另一个项目中为 wpgj-pay 生成测试

1. 从本记忆获取项目路径和测试约定
2. 用 `read_file` / `grep_search`(指定 path 为 `F:\code\ysh\code\wpgj-pay`)读取目标类的最新代码
3. 按上述测试约定生成测试,放到对应模块的 `src/test/java/` 目录
4. **注意**: 约定可能变化,生成前应先读取目标模块现有的 BaseTest 和测试文件验证

## 关键依赖

hutool 5.8.38、fastjson 2.0.57、abel533 mapper 3.0.1、ShardingSphere 5.1.1(分库分表)、Redis/Redisson、lombok
