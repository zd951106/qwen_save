---
name: GitLab只读访问规则
description: GitLab地址和令牌仅用于只读访问，任何修改操作必须先征求用户同意
type: feedback
---

# GitLab 只读访问规则

**规则**：GitLab 访问令牌仅用于**只读操作**（搜索项目、读取文件、查看配置等）。任何修改操作（commit、push、修改文件、创建分支、MR 等）都必须先征求用户明确同意。

**GitLab 信息**：
- 地址：`http://gitlab.yinxiangpay.com`
- API Token：`<GITLAB_API_TOKEN>`（私密信息，不在仓库中明文存储）
- 使用方式：通过 API 头 `PRIVATE-TOKEN: <GITLAB_API_TOKEN>`

**Why:** 用户明确指示"只能够进行访问，任何修改操作都必须征求我的意见"。

**How to apply:**
- **自主查询**：任何任务需要时，可主动通过 GitLab API 搜索项目、读取仓库文件、查看分支/提交、获取配置，无需每次征求许可
- 不可以（未经允许）：clone 到本地、修改文件、提交、推送、创建 MR、修改仓库设置
- 需要修改时：先说明要改什么文件、改什么内容，等用户确认后再操作
