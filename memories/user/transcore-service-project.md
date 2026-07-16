---
name: transcore-service project reference
description: 交易核心服务: 路径 F:\code\yinsheng\transcore-service, 4模块(web/biz/dal/facade), Java8/SpringBoot2.3.6, ShardingSphere5.1.2分库分表, Nacos+Feign微服务
type: reference
---

## 项目路径

`F:\code\yinsheng\transcore-service`

## 基本信息

- **项目名**: transcore-service (交易核心服务)
- **groupId**: `com.wpgjpay`, **Java 8**, **Spring Boot 2.3.6.RELEASE**, Spring Cloud Hoxton.SR9 + Alibaba 2.2.1
- **多模块 Maven**: web(启动+Controller) -> biz(业务) -> dal(数据访问) -> facade(API契约，独立发布)
- **根包**: `com.wpgjpay.transcore.service`
- **功能**: 商户管理、设备管理、订单查询、支付路由配置、分账、结算等交易核心能力

## 模块职责

| 模块 | 职责 |
|------|------|
| transcore-service-facade | Feign接口定义(19个Qry/Mnt) + 138个DTO(req/res/dto)，独立发布到Nexus |
| transcore-service-dal | MyBatis + tk.mybatis通用Mapper + ShardingSphere分库分表，~60+ DO实体，53个XML映射文件 |
| transcore-service-biz | 17个Service(查询/维护) + 外部集成(trade/MRS/BSS) + Redis缓存 + RabbitMQ + 邮件 |
| transcore-service-web | 25个Controller(FacadeImpl) + MapStruct对象转换 + GlobalExceptionHandler + Nacos配置 |

## 关键技术栈

- **数据库**: MySQL(阿里云RDS posx_prd库)，ShardingSphere-JDBC 5.1.2(按merchant_no HASH_MOD分64表 + 读写分离)，Druid连接池
- **缓存**: Redis Cluster(Jedis)
- **MQ**: RabbitMQ(阿里云AMQP)
- **注册/配置**: Nacos(阿里云MSE `mse-fa2e79b6-nacos-ans.mse.aliyuncs.com`)
- **RPC**: OpenFeign声明式调用
- **对象映射**: MapStruct 1.2.0.Final
- **工具**: Lombok, Guava, Hutool 5.8.10, Fastjson 1.2.38, POI 4.1.2
- **监控**: CAT(大众点评) 3.0.3
- **容器**: Docker + K8s，CentOS7.9+JDK8镜像
- **端口**: server=8081, management=8888

## 测试约定

- **框架**: JUnit 4 (`@RunWith(SpringJUnit4ClassRunner.class)`)
- **基类**: `BaseFacadeTest` — `@SpringBootTest` + `@ActiveProfiles("stg5")`
- **位置**: 27个测试均在 `transcore-service-web/src/test/java/`
- **层次**: Controller层测试17个(FacadeImplTest) + Service层测试7个(ServiceImplTest) + Mapper层测试
- **默认跳过**: `pom.xml` 中 `skipTests=true`

## 配置文件

- `application.properties` — 通用配置
- `bootstrap-prd_k8s.properties` — 生产K8s环境
- `bootstrap-fat_k8s.properties` — 测试K8s环境
