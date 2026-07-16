---
name: Controller统一响应体与请求体规范
description: 所有Controller方法必须通过RespBody/ResBody返回统一响应结构（code/message/data），所有参数通过@RequestBody接收请求体类，跨项目通用
type: feedback
---

Controller 和 Service 层必须遵循以下编码规范：

**接口HTTP方法**：

1. **默认使用 POST**：在没有特殊要求的情况下，所有接口统一使用 `@PostMapping`，不区分 GET/PUT/DELETE。简化前端调用、减少跨域预检、避免 URL 长度限制
2. 出方案时若其他 HTTP 方法（如 GET 无参查询、PUT 幂等更新）语义更合适，可保留并标注让用户确认

**统一响应体**：

1. **Controller 所有方法必须通过统一响应结构体返回**，禁止直接返回裸 DTO、Map、List、String、void 等类型
2. **复用项目现有封装类**：waxx 项目用 `RespBody`（Feign 接口 `com.wpgjpay.waxx.bean.res.RespBody`）或 `ResBody`（纯 REST Controller `com.wpgjpay.waxx.web.ResBody`），均含 `code`/`msg`/`data` 三要素。若项目无现成封装类则新建，至少含 `code`/`message`/`data`
3. **成功响应**：`RespBody.ok(data)`，data 放实际业务数据（分页数据 PageWrapper 也放 data 里）
4. **失败响应**：`RespBody.error(msg)`，由全局异常处理器统一捕获
5. **文件下载例外**：写 HttpServletResponse 流的端点（exportExcel）保持 void
6. **Service 层**：返回领域对象（Long/int/List/DTO），由 Controller 统一包装为 RespBody

**统一请求体**：

1. **所有带参接口使用 `@RequestBody` 封装为请求体类**，禁止 `@RequestParam` 逐个接收零散参数
2. 每个接口的请求体类放 `facade/req/` 包，`@Data` 注解
3. **无参方法**（如 productList）：同样使用 POST，无需请求体
4. **文件上传**（`@RequestParam MultipartFile`）属 multipart 标准模式，作为例外

**Why:** 用户要求所有接口默认 POST，简化前端调用；所有响应通过统一结构体返回，前端统一从 res.data 取值；请求参数统一用 @RequestBody 封装，避免零散参数难以维护。

**How to apply:**
- 检查项目是否有 waxx 公共库的 RespBody/ResBody，有则复用，无则新建
- Feign 接口用 `RespBody`（waxx-bean），纯 REST Controller 用 `ResBody`（waxx-web）
- Controller 实现 Feign 接口时，返回类型跟随 Feign 接口的定义
- 前端配合更新：所有接口响应从 `res.data.xxx` 取值
- 新接口默认 @PostMapping，除非有明确理由使用其他方法（出方案时标注让用户确认）
