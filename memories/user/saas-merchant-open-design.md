---
name: SaaS商户开通需求设计
description: OMS-admin + transcore-service 跨项目：商户管理下新增SaaS商户开通功能，含数据表DDL、Nacos产品配置、7个前后端接口、Vue页面结构的完整设计方案
type: project
---

# SaaS商户开通 — 跨项目设计方案

## 涉及项目

| 项目 | 路径 | 角色 |
|------|------|------|
| OMS-admin | `F:\code\yinsheng\oms-admin` | 前端（Vue 2 + Element UI） |
| transcore-service | `F:\code\yinsheng\transcore-service` | 后端（Spring Boot 2.3.6 + MyBatis + ShardingSphere） |

## 功能定位

在OMS「商户管理」菜单下新增「SaaS商户」页面（路由名 `saas-merchant-open`），为银享惠商户开通SaaS产品。与现有「采购商城管理→Saas商户」完全独立。

## 数据表 `t_sass_merchant`

- 唯一约束：`uk_merchant_no (merchant_no, ys_src_user_code)` — 同一商户+同一发起方只可开通一次
- `mer_code` → 关联 `merchant` 表查询商户名称、代理信息
- `open_status`: 1=启用（允许SaaS交易）, 2=停用（拦截SaaS交易）
- `logic_delete_flag`: 1=正常
- 预留字段：`reserved1`(json)、`reserved2`、`reserved3`

## 后端接口（transcore-service/saas/）

### 统一规范
- **响应体**：所有接口统一使用 `RespBody`（`com.wpgjpay.waxx.bean.res.RespBody`）封装，结构 `{"code":"0","msg":"success","data":...}`，前端统一从 `res.data` 取值
- **请求体**：所有带参接口使用 `@RequestBody` 接收请求体类，禁止 `@RequestParam` 零散参数
- **日志**：每个方法入口第一行打印入参完整JSON，返回前打印响应完整JSON，req/res 分行

### 接口清单

| 方法 | 路径 | 请求体 | 说明 |
|------|------|--------|------|
| POST | /saas/merchant/page | `SaasMerchantPageReq` | 分页列表（PageWrapper放data中） |
| POST | /saas/merchant/add | `SaasMerchantAddReq` | 新增开通（校验唯一性→INSERT→异步同步银盛） |
| POST | /saas/merchant/status | `SaasMerchantStatusReq` | 批量更新状态 |
| POST | /saas/merchant/export | `SaasMerchantExportReq` | 导出Excel（void，直接写文件流） |
| POST | /saas/merchant/import | `MultipartFile` | 导入Excel批量新增（multipart） |
| POST | /saas/product/list | 无 | Nacos产品列表 |
| POST | /saas/merchant/queryByMerCode | `SaasMerchantQueryReq` | 按通道商户号查商户信息 |

## 后端文件清单（15个文件）

| 层 | 文件 | 说明 |
|----|------|------|
| DAL | `SassMerchantDO.java` | 数据实体 |
| DAL | `SassMerchantMapper.java` | Mapper接口（extends TkBaseMapper） |
| DAL | `SassMerchantMapper.xml` | MyBatis XML（4条SQL） |
| Facade | `SaasMerchantFacade.java` | Feign接口 |
| Facade | `SaasMerchantPageReq.java` | 分页请求 |
| Facade | `SaasMerchantAddReq.java` | 新增请求 |
| Facade | `SaasMerchantStatusReq.java` | 状态请求 |
| Facade | `SaasMerchantExportReq.java` | 导出请求 |
| Facade | `SaasMerchantQueryReq.java` | 查询请求（merCode） |
| Facade | `SaasMerchantPageRes.java` | 分页响应 |
| Facade | `SaasMerchantInfoRes.java` | 商户信息响应 |
| Biz | `SaasMerchantEntity.java` | 业务实体 |
| Biz | `SaasMerchantService.java` | 业务接口 |
| Biz | `SaasMerchantServiceImpl.java` | 业务实现 |
| Web | `SaasMerchantFacadeImpl.java` | Controller |

## 前端代码结构（OMS-admin）

```
src/api/saas-merchant-open/index.js       # 7个API函数
src/views/modules/saas-merchant-open/
├── index.vue                              # 主列表页（PageForm+PageTable+多选）
├── config/mixin.form.js                   # 搜索表单配置（5条件）
├── config/mixin.table.js                  # 表格列配置（9列）
└── components/
    ├── AddDialog.vue                      # 新增弹窗（失焦自动带出）
    ├── ImportDialog.vue                   # 导入弹窗（封装公共ImportDialog）
    └── StatusBatch.vue                    # 批量状态管理
```

路由：`merchant-manage` children 新增 `saas-merchant-open`

## 核心交互流程

**新增**：输入通道商户号 → 失焦查merchant表校验 → 自动带出平台商户号/商户名称/直属代理 → 选择产品+输入发起方ID → 提交 → INSERT DB → 异步同步银盛

**批量状态**：多选行 → 选择启用/停用 → 统一更新

**导入**：上传Excel（通道商户号列）→ 批量校验 → 批量INSERT（自动带出商户信息）

## 待办事项

| # | TODO | 状态 | 阻塞 |
|---|------|------|------|
| 1 | 银盛侧同步接口定义 | 待提供 | 是 |
| 2 | Nacos产品配置 | 待配置 | 否 |
| 3 | DDL执行 | 待执行 | 是 |
| 4 | JWT操作用户获取 | hardcode "system" | 否 |
| 5 | 联调测试 | 待执行 | 否 |

## 设计文档路径

`F:\code\yinsheng\oms-admin\docs\saas商户开通需求\设计方案.md`
