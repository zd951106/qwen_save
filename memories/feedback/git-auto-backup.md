---
name: 个人Git仓库自动备份规则
description: 每天早晨自动保存.qwen目录到个人GitHub仓库(qwen_save)，中午询问是否保存，换机器时clone即可恢复
type: feedback
---

每天自动将 `C:\Users\admin\.qwen` 目录备份到个人 GitHub 仓库 `https://github.com/zd951106/qwen_save.git`。

**Why:** 用户希望方法论文件（需求设计规范、编码规范、工作方式反馈等）能跨工作环境保存，换工作时带走。

**How to apply:**
- **早晨第一次对话**：自动执行 `git add . && git commit -m "auto: 早晨自动备份" && git push`，不询问用户
- **中午12点后第一次对话**：询问用户是否需要保存更新，确认后执行 git add + commit + push
- 仓库地址：`https://github.com/zd951106/qwen_save.git`
- 工作目录：`C:\Users\admin\.qwen`
- `.gitignore` 已排除：projects/、tmp/、file-history/、todos/、usage/、settings.json、*.log
- 已设置 durable cron 任务（7天自动过期，需定期重建）
- **换机器时**：`git clone https://github.com/zd951106/qwen_save.git ~/.qwen` 即可恢复所有方法论
- 如果 git push 失败（如网络问题），告知用户但不阻塞后续工作
