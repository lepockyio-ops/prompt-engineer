---
name: pro-prompt-engineer
description: 按 Anthropic 官方提示词最佳实践和 Claude Code 提示词库，把用户需求写成专业提示词（系统提示词、Claude Code 任务提示词、智能体或 API 提示词）。用户说“写/优化/改进提示词”时使用。
---

# 专业提示词生成器

依据两份官方文档：
- Claude Prompting Best Practices：https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices
- Claude Code Prompt Library：https://code.claude.com/docs/en/prompt-library

目标：把用户的想法写成一份能直接复制使用、符合官方最佳实践的提示词。

---

## 一、工作流程（按顺序执行）

### 第 1 步：判断提示词类型

| 类型 | 典型场景 | 写法要点 |
|---|---|---|
| A. 系统提示词 / API 提示词 | 产品内置助手、客服机器人、RAG 应用 | 角色、背景、XML 分区、示例、输出格式、边界情况 |
| B. Claude Code / 编程智能体任务提示词 | 让 Claude Code 修 bug、加功能、重构、审查 | 写清想要的结果、验证方法、参考文件、可度量的目标、相关材料 |
| C. 长时间运行的智能体系统提示词 | 多窗口编码、研究、自动化流程 | 状态管理、上下文压缩说明、行动边界、并行工具调用、子智能体使用规则 |
| D. 一次性对话提示词 | 写作、分析、翻译、总结 | 清楚直接、说明背景和理由、提供示例、指定格式 |

### 第 2 步：补齐缺失信息

先检查以下要素是否已知，缺失项最多问 3 个关键问题；用户不在或明确要求直接写时，做出合理假设，并在交付时列出假设：
- 目标：要完成的最终结果是什么？成功标准是什么？
- 受众和用途：谁读这份输出，用在哪里？
- 输入：有哪些文档、数据、代码或示例？
- 输出：格式、长度、语气、语言？
- 约束：哪些事必须做或不能做，原因是什么？
- 目标模型和运行环境：API、Claude Code、claude.ai，或其他环境？

### 第 3 步：按核心原则起草（见第二节）

### 第 4 步：从模块库中选用合适的模块（见第三、四节），不要全部堆进去

### 第 5 步：逐项检查清单（见第六节）

### 第 6 步：按交付格式输出（见第七节）

---

## 二、核心原则（官方最佳实践）

1. **清楚直接**：把 Claude 当作“聪明但刚入职、不了解你们规范的新同事”。检验方法：把提示词给一个缺少背景的同事看，如果对方看不懂，Claude 也会看不懂。想要超出字面要求的结果，就要明确提出（如“尽量包含相关功能和交互，不要只做基础版本”）。
2. **说明原因**：不要只写规则，还要说明这条规则为什么重要。Claude 能根据理由推广到相似情况。
   - 差：`NEVER use ellipses`
   - 好：`你的回复会由语音合成引擎朗读，引擎无法读出省略号，所以不要使用省略号。`
3. **步骤用编号列表**：顺序或完整性重要时，用编号或项目符号列出。
4. **用示例引导**：3–5 个示例，要求相关（贴近真实场景）、多样（覆盖边界情况，避免让模型学到不该学的模式）、有结构（用 `<example>` / `<examples>` 包裹）。
5. **用 XML 标签分区**：用 `<instructions>`、`<context>`、`<input>`、`<documents>` 等，标签名保持一致、能看出含义，可以嵌套。
6. **设定角色**：在系统提示词里写一句角色设定即可明显聚焦，例如“你是一名专精 Python 的编程助手”。
7. **长上下文（2 万 token 以上）**：把长文档放在最前面，问题和指令放在最后（官方测试中质量最多可提升 30%）；每份文档用 `<document><source>…</source><document_content>…</document_content></document>` 包裹；要求先引用相关原文，再完成任务。
8. **说要做什么，不说不要做什么**：
   - 差：`不要使用 markdown`
   - 好：`用连贯的段落写作。`
9. **提示词风格会影响输出风格**：想要纯文本输出，提示词本身就少用 markdown。
10. **用词明确表示要行动**：“你能建议一些修改吗”通常只会得到建议；要让 Claude 直接修改，就写“修改这个函数以提升性能”。
11. **语气不要过强**：新模型对系统提示词更敏感。把“CRITICAL: You MUST…”改成“在……时使用这个工具”。删除“如有疑问就使用 X”这类容易导致过度调用的写法。
12. **思考方式**：优先给出概括性指令（如“仔细思考”），不要手写过细的思考步骤，因为模型的推理通常比人写的步骤更好。关闭思考时，可以要求先推理，再把答案放进 `<answer>` 标签。不要要求在 `<thinking>` 标签里写推理（部分新模型可能拒绝）。
13. **自我检查**：编码和数学任务加上“完成前，按 [测试标准] 验证你的答案”。（Opus 5 不加也会主动验证，迁移旧提示词时可删除此类说明。）
14. **不要使用 prefill**：从 Claude 4.6 起，在最后一个 assistant 轮次预填内容会返回 400 错误。改为在指令中明确格式要求。

---

## 三、模块库（官方原文片段，按需选用）

> 片段保留英文原文（官方测试使用英文措辞）。如果用户要求全中文提示词，可以忠实翻译。

### 3.1 主动执行（默认直接动手）
```
<default_to_action>
By default, implement changes rather than only suggesting them. If the user's intent is unclear, infer the most useful likely action and proceed, using tools to discover any missing details instead of guessing. Try to infer the user's intent about whether a tool call (e.g., file edit or read) is intended or not, and act accordingly.
</default_to_action>
```

### 3.2 保守执行（只在明确要求时动手）
```
<do_not_act_before_instructions>
Do not jump into implementation or change files unless clearly instructed to make changes. When the user's intent is ambiguous, default to providing information, doing research, and providing recommendations rather than taking action. Only proceed with edits, modifications, or implementations when the user explicitly requests them.
</do_not_act_before_instructions>
```

### 3.3 并行工具调用
```
<use_parallel_tool_calls>
If you intend to call multiple tools and there are no dependencies between the tool calls, make all of the independent tool calls in parallel. Prioritize calling tools simultaneously whenever the actions can be done in parallel rather than sequentially. For example, when reading 3 files, run 3 tool calls in parallel to read all 3 files into context at the same time. Maximize use of parallel tool calls where possible to increase speed and efficiency. However, if some tool calls depend on previous calls to inform dependent values like the parameters, do NOT call these tools in parallel and instead call them sequentially. Never use placeholders or guess missing parameters in tool calls.
</use_parallel_tool_calls>
```

### 3.4 减少幻觉（先查证再回答）
```
<investigate_before_answering>
Never speculate about code you have not opened. If the user references a specific file, you MUST read the file before answering. Make sure to investigate and read relevant files BEFORE answering questions about the codebase. Never make any claims about code before investigating unless you are certain of the correct answer - give grounded and hallucination-free answers.
</investigate_before_answering>
```

### 3.5 防止过度设计
```
Avoid over-engineering. Only make changes that are directly requested or clearly necessary. Keep solutions simple and focused:
- Scope: Don't add features, refactor code, or make "improvements" beyond what was asked.
- Documentation: Don't add docstrings, comments, or type annotations to code you didn't change. Only add comments where the logic isn't self-evident.
- Defensive coding: Don't add error handling, fallbacks, or validation for scenarios that can't happen. Only validate at system boundaries (user input, external APIs).
- Abstractions: Don't create helpers, utilities, or abstractions for one-time operations. Don't design for hypothetical future requirements.
```

### 3.6 不为通过测试而硬编码
```
Please write a high-quality, general-purpose solution using the standard tools available. Implement a solution that works correctly for all valid inputs, not just the test cases. Do not hard-code values or create solutions that only work for specific test inputs. Tests are there to verify correctness, not to define the solution. If the task is unreasonable or infeasible, or if any of the tests are incorrect, please inform me rather than working around them.
```

### 3.7 自主行动与安全边界
```
Consider the reversibility and potential impact of your actions. You are encouraged to take local, reversible actions like editing files or running tests, but for actions that are hard to reverse, affect shared systems, or could be destructive, ask the user before proceeding.
Examples of actions that warrant confirmation:
- Destructive operations: deleting files or branches, dropping database tables, rm -rf
- Hard to reverse operations: git push --force, git reset --hard, amending published commits
- Operations visible to others: pushing code, commenting on PRs/issues, sending messages, modifying shared infrastructure
When encountering obstacles, do not use destructive actions as a shortcut (e.g. don't bypass safety checks with --no-verify or discard unfamiliar files that may be in-progress work).
```

### 3.8 上下文压缩与长任务
```
Your context window will be automatically compacted as it approaches its limit, allowing you to continue working indefinitely from where you left off. Therefore, do not stop tasks early due to token budget concerns. As you approach your token budget limit, save your current progress and state to memory before the context window refreshes. Always be as persistent and autonomous as possible and complete tasks fully.
```
```
This is a very long task, so it may be beneficial to plan out your work clearly. It's encouraged to spend your entire output context working on the task - just make sure you don't run out of context with significant uncommitted work. Continue working systematically until you have completed this task.
```

### 3.9 多窗口状态管理（写进提示词的规则）
- 第一个窗口单独写提示词：先搭好框架（测试、`init.sh`），后续窗口再按待办列表迭代。
- 结构化状态（测试结果、任务状态）存为 JSON（如 `tests.json`）；进度笔记用纯文本（`progress.txt`）；用 git 记录状态和检查点。
- 告诉模型启动时怎么做：“先运行 pwd；查看 progress.txt、tests.json 和 git log；开发新功能前先跑一遍基础集成测试。”
- 写明：“It is unacceptable to remove or edit tests because this could lead to missing or buggy functionality.”

### 3.10 子智能体使用规则
```
Use subagents when tasks can run in parallel, require isolated context, or involve independent workstreams that don't need to share state. For simple tasks, sequential operations, single-file edits, or tasks where you need to maintain context across steps, work directly rather than delegating.
```

### 3.11 避免过度思考
```
When you're deciding how to approach a problem, choose an approach and commit to it. Avoid revisiting decisions unless you encounter new information that directly contradicts your reasoning. If you're weighing two approaches, pick one and see it through. You can always course-correct later if the chosen approach fails.
```

### 3.12 工具调用后反思
```
After receiving tool results, carefully reflect on their quality and determine optimal next steps before proceeding. Use your thinking to plan and iterate based on this new information, and then take the best next action.
```

### 3.13 结构化研究
```
Search for this information in a structured way. As you gather data, develop several competing hypotheses. Track your confidence levels in your progress notes to improve calibration. Regularly self-critique your approach and plan. Update a hypothesis tree or research notes file to persist information and provide transparency. Break down this complex research task systematically.
```
先写清研究的成功标准，再要求用多个来源交叉验证。

### 3.14 输出风格与格式
- 少用 markdown、多写段落：使用 `<avoid_excessive_markdown_and_bullet_points>` 模块，要求用段落写作；只在列举彼此独立的项目或用户要求列表时才用列表。（对本来就少用格式的模型可删除。）
- 纯文本数学：`Format your response in plain text only. Do not use LaTeX... Write all math expressions using standard text characters.`
- 工具调用后总结：`After completing a task that involves tool use, provide a quick summary of the work you've done.`
- 清理临时文件：`If you create any temporary new files, scripts, or helper files for iteration, clean up these files by removing them at the end of the task.`

### 3.15 前端设计
避免千篇一律的“AI 风格”页面，在 `<frontend_aesthetics>` 中写明：选用有辨识度的字体（避免 Arial、Inter 等常见字体）；用 CSS 变量建立统一的配色，以主色加醒目强调色为主；动效以 CSS 为主，做好一次页面加载时的错落入场动画；背景用渐变、图案等营造层次；避免白底紫色渐变和套路化布局；在明暗主题和不同风格之间变化。同时加上质量要求，例如“尽量包含相关功能和交互，做出完整版本”。

---

## 四、Claude Code 任务提示词（官方 Prompt Library 的模式）

### 4.1 六条写法原则
1. **写想要的结果，不写具体步骤**：让 Claude 自己找相关文件。例如：`add rate limiting to the public API and make sure existing tests still pass`
2. **给出验证方法**：在同一句里要求运行、测试、对比或验证。例如：`write the migration, run it against the dev database, and confirm the schema matches`
3. **指定参考对象**：例如：`add a settings page that follows the same layout as the profile page`
4. **给出可度量的目标**：例如：`get the bundle size under 200KB and show me what you removed`
5. **直接提供相关材料**：贴上错误信息、日志、截图，或用 `@文件` 引用。例如：`why is the build failing? @build.log`
6. **说明回答形式**：写明格式、长度和受众。例如：`explain ... as an HTML page with a diagram, then open it in my browser`

### 4.2 按开发阶段整理的模板（`{}` 为需要填写的内容）
- **熟悉项目**：`give me an overview of this codebase: architecture, key directories, and how the pieces connect`
- **理解代码**：`explain what {path} does and how data flows through it. write it up as {format}` ／ `where do we {behavior}?` ／ `what would break if I deleted {target}?` ／ `look through the commit history of {path} and summarize how it evolved and why` ／ `which files would I need to touch to {change}?`
- **规划**：`plan how to refactor the {target} to {goal}. list the files you would change, but don't edit anything yet` ／ `I want to build {feature}. interview me about implementation, UX, edge cases, and tradeoffs until we have covered everything, then write the spec to SPEC.md` ／ `list the error states, empty states, and edge cases for {feature} that the design needs to cover`
- **原型**：`implement this design, then take a screenshot of the result, compare it to the original, and fix any differences`
- **实现**：`look at how {example} is implemented to understand the pattern, then build {new} the same way` ／ `read issue #{issue}, implement the fix, and run the tests` ／ `find every place we say "{copy}"..., show me each one in context, then update them all to "{new}". leave tests and the changelog alone`
- **测试**：`write tests for {path}, run them, and fix any failures` ／ `write tests for {feature} first, then implement it until they pass`
- **重构**：`migrate everything from {from} to {to}: identify every place that needs to change, then make the changes` ／ `optimize {target} to bring {metric} from {current} down to under {goal}`
- **审查**：`review my uncommitted changes and flag anything that looks risky before I commit` ／ `use a subagent to review {path} for security issues and report what it finds`
- **纠正方向**：`that is not right: {feedback}. try a different approach` ／ `that is too much. keep only the changes to {scope} and undo your other edits` ／ `you keep {mistake}. add a rule to CLAUDE.md so this stops happening`
- **调试**：`the {test} test is failing, find out why and fix it` ／ `here is a build error. fix the root cause and verify the build succeeds`
- **故障排查**：`{symptom}. check the logs, recent deploys, and config changes, then tell me the most likely cause`
- **发布**：`compare {from} to {to} and draft release notes grouped by feature, fix, and breaking change`
- **自动化**：`create a /{name} skill for this project that {steps}` ／ `write a hook that {action} after every {event}` ／ `summarize what we did this session and suggest what to add to CLAUDE.md`

---

## 五、常见错误与改法

| 常见错误 | 改法 |
|---|---|
| 需求模糊，例如“做一个仪表盘” | 写清范围和质量要求，并明确要求“做完整版本” |
| 只写规则，不说原因 | 每条重要约束补一句原因 |
| 大量“不要……” | 改成“要……” |
| 全部大写、CRITICAL、MUST 堆满全文 | 改成平实语气，只保留真正关键的少数限制 |
| 问题放在长文档前面 | 文档放前面，问题放最后 |
| 只给一个示例，或示例风格相同 | 给 3–5 个多样示例，并用标签包裹 |
| 用“你能建议……吗”表达“请直接修改” | 直接写要执行的动作 |
| 编程任务没有验证方法 | 要求运行测试、截图对比或确认结果 |
| 手写过细的思考步骤 | 改为概括性要求，让模型自己推理 |
| 依赖 prefill 控制格式 | 在指令和格式要求中直接说明 |
| 塞满所有模块 | 只保留和这个任务相关的模块 |

---

## 六、交付前检查清单

- [ ] 没有背景的同事读完能明白要做什么吗？
- [ ] 目标、成功标准、受众都写清楚了吗？
- [ ] 关键约束都说明了原因吗？
- [ ] 指令、背景、输入和示例用 XML 标签分开了吗？
- [ ] 长文档放在前面、问题放在最后了吗？
- [ ] 示例是否足够相关、多样，并且有结构？
- [ ] 输出格式是否用正面方式描述，提示词本身的风格是否和期望输出一致？
- [ ] 编程或智能体任务有验证方法和可度量的目标吗？
- [ ] 行动边界清楚吗（主动执行还是保守执行、哪些操作需要确认）？
- [ ] 语气是否没有过强，避免导致过度触发？
- [ ] 是否删除了和任务无关的模块？
- [ ] 占位符是否都用 `{变量}` 标出？

---

## 七、交付格式

按以下结构回复用户：

1. **最终提示词**：放在一个代码块里，方便复制。系统提示词和用户消息分开给出时，分别标注。
2. **设计说明**（3–6 条，简短）：说明用了哪些官方原则或模块，以及为什么这样用。
3. **假设与可调项**：列出补充的假设，以及用户可以替换的 `{变量}`。
4. （可选）**进一步优化建议**：例如补充真实示例、配合 eval 测试，或调整 effort 参数。

语言规则：说明部分使用用户的语言。提示词正文默认使用用户的语言，官方英文模块可保留原文；用户指定语言时按用户要求。

### 系统提示词骨架（类型 A / C 可直接套用）
```
你是{角色}，为{用户/场景}提供{核心价值}。

<context>
{背景：产品、受众、使用环境，以及这份提示词为什么这样设计}
</context>

<instructions>
1. {步骤或规则，并说明原因}
2. ...
</instructions>

<output_format>
{用正面方式描述格式、长度、语气}
</output_format>

<examples>
<example>
<input>...</input>
<output>...</output>
</example>
</examples>

{按需选用的模块，如 <investigate_before_answering>、行动边界}

{长文档放在 <documents> 中，位于问题之前}
{用户问题或任务放在最后}
```
