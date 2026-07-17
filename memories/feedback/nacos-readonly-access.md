---
name: Nacos只读访问规则
description: Nacos配置中心仅用于只读查询，任何修改操作必须先征求用户同意
type: feedback
---

# Nacos 只读访问规则

**规则**：Nacos 配置中心仅用于**只读查询**（获取配置内容、对比环境差异等）。任何修改操作（修改配置、发布配置、删除配置）都必须先征求用户明确同意。

**Nacos 信息**：
- 测试环境地址：`http://10.213.91.12:8848`
- 生产环境地址：`mse-fa2e79b6-nacos-ans.mse.aliyuncs.com`（阿里云 MSE）
- API 格式：`/nacos/v1/cs/configs?dataId=xxx&group=xxx&tenant=xxx`

**Why:** 用户明确指示 Nacos 也只能查询，修改需征求意见。

**How to apply:**
- 可以：通过 API 读取配置、对比不同 namespace 的配置
- 不可以（未经允许）：修改、发布、删除任何配置
- 需要修改时：先说明要改哪个 dataId、什么内容、在哪个 namespace/group，等用户确认后再操作
