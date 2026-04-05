# Session 4 执行流程

这份文档用于固定 session-4 的标准执行顺序，避免在“拉项目、生成 prompt、执行模型、记录结果、分析输出”之间漏步骤。

## 1. 准备阶段

先确认 session-4 相关目录：

- 项目代码：`02.work session/session-4/gitlab source/`
- 模型结果：`02.work session/session-4/ai-model-result/`
- 提示词文件：`03.trae prompts/session-4.md`
- 拉取技能：`01.entry-skills/session-4/session-4-clone-gitlab-skill.md`
- 分析技能：`01.entry-skills/session-4/analysis-skill.md`

## 2. 拉取项目

在 skill app 中发起对话，给出项目 ID，例如：

```text
项目 id: 2532
```

执行时结合 `01.entry-skills/session-4/session-4-clone-gitlab-skill.md`，完成以下动作：

- 将项目拉取到 `02.work session/session-4/gitlab source/<项目名>/`
- 生成 prompt，并写入 `03.trae prompts/session-4.md`
- 创建对应的模型结果记录文件 `02.work session/session-4/ai-model-result/<项目名>.md`

## 3. 确认提示词

打开 `03.trae prompts/session-4.md`，找到当前项目名对应的 prompt。

如果该项目下存在多条 prompt：

- 明确本次要测试的是哪一条
- 不要把多条 prompt 混在同一次模型执行里
- 后续记录结果时，按实际执行的 prompt 对应填写

## 4. 执行模型

用 trae-cn 打开对应项目目录，然后把选定的 prompt 发给模型执行。

执行时建议记录：

- 当前测试项目名
- 实际使用的 prompt 内容
- 模型返回的 session id
- 是否发生多轮追问或修正

## 5. 记录模型结果

把模型的 session id 和完整对话内容写入：

- `02.work session/session-4/ai-model-result/<项目名>.md`

记录时遵循下面的原则：

- 一条 prompt 对应一组结果记录
- 如果有多轮对话，按轮次依次追加在同一个文件里
- 每一轮都单独写清楚：用户提示词、模型回答、session id

## 6. 分析结果

分析时使用以下输入：

- `02.work session/session-4/ai-model-result/<项目名>.md`
- `01.entry-skills/session-4/analysis-skill.md`
- `03.trae prompts/session-4.md`
- `02.work session/session-4/gitlab source/<项目名>/`

分析目标是：

- 判断模型是否完成 prompt 要求
- 区分“没做成”和“做得一般”
- 输出最终评价结果到 `02.work session/session-4/ai-model-result/<项目名>-评价结果.md`

## 7. 输出物

完成一轮后，session-4 下应至少有这些结果：

- 项目代码目录
- prompt 记录
- 模型执行记录
- 评价结果文件



