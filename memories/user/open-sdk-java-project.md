---
name: open-sdk-java project reference
description: Cross-project pointer to the open-sdk-java SDK project — absolute path, project structure, test conventions, and relationship to wpgj-pay
type: user
---

## 项目路径

`F:\code\ysh\code\open-sdk-java`

## 基本信息

- **项目名**: open-sdk-java (银享惠机构支付SDK)
- **groupId**: `com.yinxiangpay`, **artifactId**: `open-sdk-java`, **Java 8**, Maven 单模块
- **根包**: `com.yinxiangpay.sdk`
- **作用**: 封装银享惠开放平台的签名、验签、加密、HTTP 通信，供机构方调用

## 与 wpgj-pay 的关系

open-sdk-java 是 **SDK 客户端**，wpgj-pay 是 **服务端**。SDK 通过 `client.post("/opensaas/route", publicParam)` 调用 wpgj-pay 的 OpenSaaS 路由接口，路由根据 `method` 参数分发到不同适配器。

## SDK 核心类

- `YinxiangPaySdkClient`: 核心客户端，提供 `post`(明文)、`postEncrypted`(混合加密)、`upload`(文件上传) 三种模式
- `SdkConfig`: 配置类（平台公钥、商户私钥、appId、baseUrl、version）
- `SdkResponse`: 响应类（code/msg/data，`isOk()` 判断成功 code="0000"）
- `AesRsaUtil`: 签名验签 + AES/RSA 加解密工具

## 请求报文结构

SDK `post` 方法将业务参数序列化为 JSON 放入 `data` 字段，加上 `serialNo`/`timestamp`/`appId`/`version`/`signature`，整体发送到 `baseUrl + apiPath`。

OpenSaaS 路由请求的 `data` 内部结构：
- 统一网关: `{method, certId, version, bizContent:{...}}`
- 业务网关: `{method, partnerId, version, bizContent:{...}}`

## 测试约定

- **框架**: JUnit 4 (junit 4.13.2), **无 Spring**, 纯集成测试（连真实测试环境）
- **依赖**: fastjson 1.2.83, httpclient 4.5.14, commons-lang3
- **Base URL**: `https://posx.yinxiangpay-test.com`
- **风格**: `@Before` 初始化 SdkConfig + client，`@After` 关闭 client；中文注释描述场景；`System.setOut(new PrintStream(System.out, true, "UTF-8"))` 解决 Windows 乱码
- **测试文件**: `src/test/java/com/yinxiangpay/sdk/` 下，如 `TradeSaasTest.java`(正常流程)、`TradeSaasRobustTest.java`(健壮性测试)、`TradeTest.java`(交易接口)、`MerTest.java`、`TradeWuxsTest.java`
- **断言**: `TradeSaasRobustTest.java` 引入了 `org.junit.Assert` 断言，其他测试文件仅 printResponse 无断言

## 如何在 open-sdk-java 中为 wpgj-pay 写测试

1. 从本记忆获取项目路径和 SDK 结构
2. 用 `read_file` / `grep_search`（指定 path 为 `F:\code\ysh\code\wpgj-pay`）读取服务端适配器的校验逻辑
3. 在 open-sdk-java 的 `src/test/java/com/yinxiangpay/sdk/` 下创建测试文件
4. 测试通过 SDK client 调用 `/opensaas/route`，验证服务端的参数校验和业务逻辑
