# 提示词与交互脚本 · Prompts

存放场景中用到的**提示词（prompts）、对话流程、交互脚本**。例如：

- 驱动语言-conditioned 策略的自然语言指令模板。
- 多轮交互（人 → 机器人 → 人）的对话状态机。
- LLM 充当「任务规划器」时的 system / user prompt。

---

## 文件命名

`用途-关键词.md`，如 `task-planner-system-prompt.md`、`instruction-templates.md`。

---

## 记录建议

```markdown
# 提示词标题

## 用途
在哪个场景 / 哪个环节使用？

## Prompt
（代码块，标注语言如 text / json）

## 变量
{object}、{location} 等占位说明。

## 示例输入 / 输出
## 调优记录
```
