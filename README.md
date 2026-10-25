# pro-prompt-engineer

一个 Claude 技能（Skill）：根据 Anthropic 官方文档，把你的需求写成专业提示词。适用于系统提示词、Claude Code 任务提示词、智能体或 API 提示词，以及一次性对话提示词。

## 依据

- [Claude Prompting Best Practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)
- [Claude Code Prompt Library](https://code.claude.com/docs/en/prompt-library)

## 安装

把 `SKILL.md` 放进技能目录：

- **Claude Code**：`~/.claude/skills/pro-prompt-engineer/SKILL.md`（个人）或 `<项目>/.claude/skills/pro-prompt-engineer/SKILL.md`（项目）
- **Claude.ai / Claude 桌面应用**：把本目录打包成 zip，在 Settings → Capabilities → Skills 中上传

```bash
git clone https://github.com/lepockyio-ops/pro-prompt-engineer.git ~/.claude/skills/pro-prompt-engineer
```

## 用法

对 Claude 说：“帮我写一个……的提示词”或“优化这段提示词”。Claude 会这样处理：

1. 判断要写的是哪类提示词
2. 缺少关键信息时最多问 3 个问题，或先做合理假设
3. 按官方原则起草：写清楚、说明原因、给示例、用 XML 标签分区
4. 按需加入官方原文片段，比如并行调用工具、先查证再回答、防止过度设计、行动边界
5. 交付前逐项检查，然后给出可直接复制的提示词，附上设计说明和可替换的变量
