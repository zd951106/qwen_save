---
name: posx-tcs 对账结算平台项目参考
description: 跨项目指针 — posx-tcs 对账结算平台的路径、模块结构、技术栈、关键约定，供其他 IDEA 项目窗口引用
type: reference
---

## 项目路径

`F:\code\yinsheng\posx-tcs`

## 基本信息

- **项目名**: posx-tcs（对账结算平台）
- **groupId**: `com.posx`, **artifactId**: `posx-tcs`, **version**: `0.0.6-SNAPSHOT`
- **Java 8**, **Spring Boot 2.3.4.RELEASE**, Spring Cloud Alibaba 2.2.1.RELEASE
- **多模块 Maven**: posx-tcs-common(通用)、posx-tcs-facade(对外接口/DTO)、posx-tcs-repository(实体/DAO)、posx-tcs-service(业务核心)、posx-tcs-web(Controller/Job 入口)
- **根包**: `com.posx.tcs`

## 技术栈要点

- **定时任务**: XXL-JOB 2.3.0（`@XxlJob` 注解，参数走 `XxlJobHelper.getJobParam()` JSON）
- **分库分表**: ShardingSphere 5.2.1（按 billDate/merchantNo 分片）
- **配置中心**: Nacos（`@RefreshScope` + `@Value` 热更新）
- **文件生成**: FreeMarker（文本对账单）+ Apache POI SXSSFWorkbook（Excel 对账单）
- **文件推送**: WebDAV（sardine / WebDavTemplate）
- **监控**: Cat（`Cat.logError` / `Cat.logEvent`）
- **日志**: `@Slf4j` + MDC traceId
- **消息**: spring-cloud-starter-stream-rabbit
- **其他**: fastjson 2.0.57, hutool 5.7.22, swagger 2.9.2, openfeign

## 关键外部 Facade 依赖

- `posx-mcs-facade`（商户系统：门店/订单/教培扩展）
- `transcore-service-facade`（分账服务）
- `agent-facade`（机构/代理商层级）
- `act-facade`
- `waxx-runtime` / `waxx-tool`（wpgjpay 基础框架）

## 核心业务域

- **对账**: 渠道对账单解析（ParseXxxRecnclnFileServiceImpl 多渠道）、对账差错处理、对账结果
- **生成账单**: 机构对账单（定时任务+FreeMarker）、商户对账单（接口+异步Excel，策略模式）
- **结算**: 分账、手续费计算

## 生成账单链路（详见项目内 docs/bill-generation-flow.md）

- 机构对账单: `GenerateOrgStmtJob` → `OrgStmtServiceImpl.generateStmt()` → `WebDAVService.push()` → `OrgStmtInfoServiceImpl.saveStmtInfo()`
- 商户对账单: `ReconciliationController` → `MerStmtServiceFactory` → `BaseMerStmtServiceImpl`/`DefaultMerStmtServiceImpl` 等

## 与其他项目的关系

- 与 **wpgj-pay**（`F:\code\ysh\code\wpgj-pay`）同属 wpgjpay 体系，共享 `com.wpgjpay` 依赖（waxx/pay-core-common 等）
- posx-tcs 是对账结算平台，wpgj-pay 是扫码交易系统，两者通过 Facade 解耦调用

## How to apply

当用户在其他项目窗口提到"对账"、"账单"、"tcs"、"posx" 时，关联此项目。需要分析 posx-tcs 代码时读此路径。
