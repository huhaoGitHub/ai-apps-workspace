# AI App Workspace

这个仓库用于管理一套围绕项目拉取、提示词生成、模型执行记录和结果分析的工作流。

它不是单个业务项目的源码仓库，而是一个面向多项目、多轮实验的工作台。核心用途包括：

- 按 session 拉取并整理项目代码
- 为项目生成可复用的提示词
- 记录模型执行过程、session id 和多轮对话
- 基于代码、提示词和结果文件做完成度分析
- 沉淀流程文档、技能文件和执行约定

## 目录分层

```text
apps/
├── .github/                   # Workspace agents / instructions
├── [pre-work]workflow steps/  # 执行流程文档
├── 01.entry-skills/           # 入口技能
├── 02.work session/           # 各 session 的工作区
└── 03.trae prompts/           # 各 session 的提示词记录
```

## 目录说明

### .github/

存放工作区级别的 agent 和 instruction 配置，用来约束 PromptArchitect、分析流程等行为。

### [pre-work]workflow steps/

存放流程文档，用于固定执行步骤。这里主要解决“先做什么、后做什么、结果写到哪里”的问题。

### 01.entry-skills/

存放入口技能文件，定义具体任务如何执行。当前主要包括两类能力：

- 项目拉取与提示词生成
- 结果分析与评价输出

### 02.work session/

存放每个 session 的实际工作目录。一个 session 下通常包含两类内容：

- `gitlab source/`：拉下来的项目代码
- `ai-model-result/`：模型执行记录和评价结果

例如 `02.work session/session-4/` 就是 session-4 的完整工作区。

### 03.trae prompts/

保存每个 session 对应的提示词记录。通常一个项目会在这里保留当前测试用的 prompt，供后续执行和分析阶段引用。

## 推荐流程

一次完整的 session 流程通常是：

1. 根据项目 ID，把代码拉到 `02.work session/<session>/gitlab source/`。
2. 基于项目结构生成提示词，并写入 `03.trae prompts/<session>.md`。
3. 将提示词发给模型执行，记录 session id 和完整回答。
4. 把执行过程沉淀到 `02.work session/<session>/ai-model-result/`。
5. 对照代码、提示词和结果文件，输出评价结果。

## Session 4 约定

当前工作区主要围绕 session-4 运行，默认约定如下：

- 项目代码目录：`02.work session/session-4/gitlab source/<项目名>/`
- 模型结果目录：`02.work session/session-4/ai-model-result/`
- 提示词文件：`03.trae prompts/session-4.md`
- 流程文档：`[pre-work]workflow steps/session-4-flow.md`
- 入口技能：`01.entry-skills/session-4/`

## 仓库定位

这是一个 AI 项目实验工作流仓库，重点是：

- 组织材料
- 固化流程
- 沉淀结果
- 复用技能

它不以运行某一个业务应用为目标，而是为多项目、多轮模型测试提供统一的管理入口。
