---
name: Preauthorized execution, but confirm approach before code changes
description: User grants blanket preauthorization for operations/tool execution across all projects (no per-action "may I"), but still wants to confirm the approach/design before non-trivial code changes are implemented
type: feedback
---

Two-layer rule:
1. **Execution / tool calls**: fully preauthorized across all projects — execute directly, do not ask "may I" or wait for permission.
2. **Approach / design**: for non-trivial code changes, first present the analysis and proposed approach, and only start editing code after the user confirms. (Trivial fixes like typos/formatting can be done directly.)

**Why:** User stated "不论在哪个项目，我会给你所有的权限，不用去询问我是否允许。全部允许" (for execution permission), then clarified "方案需要我确认" (for approach/design decisions). I initially saved "完全自主" (no approach discussion either) based on the first answer, but the user corrected this — approach confirmation is still required.

**How to apply:**
- Tool calls / commands / routine reversible actions: just do them, no permission ask.
- Non-trivial code changes: propose approach → wait for user confirmation → then implement. Do not jump straight to editing even if the fix seems obvious.
- Genuinely irreversible / high-blast-radius actions (deleting data, force-push, prod changes): state the action clearly before/while doing it so the user can intervene, but do not block waiting for approval.
- Note: tool-call confirmation popups are controlled by the Qwen Code client's approval mode, not by the model — to fully suppress those the user must set approval mode to "yolo" themselves.
