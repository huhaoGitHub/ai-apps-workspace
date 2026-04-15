---
name: session-4-clone-gitlab
description: "Use when: 需要根据 GitLab 项目 ID 拉取一份主仓，并按指定类型与数量批量生成提示词、落盘到分类型目录、再把主仓复制成多份子仓（Bug 修复必须在独立子仓造 bug）。适用于 session-4 场景。"
---

# GitLab 拉取与批量提示词技能

## 功能概述

这个技能用于处理 session-4 的项目拉取与批量提示词生成入口。它先拉取一份主仓，再按类型和数量生成提示词文件，并复制出与提示词一一对应的子仓目录。

这个技能负责：

- 根据项目 ID 补全项目名
- 拼接 GitLab 仓库地址并执行单份 clone（主仓）
- 按类型和数量批量生成提示词
- 按项目/类型两级目录写入提示词文件
- 按提示词复制主仓为多个子仓，做到一条提示词对应一个子仓
- 对 Bug 修复类型在对应子仓内设计 bug，避免互相污染

这个技能不负责：

- GitHub 仓库创建与推送
- 本地分支创建、切换、跟踪

## 使用场景

- 用户给出一个或多个 GitLab 项目 ID
- 需要按统一规则拉取到 session-4 工作区
- 需要按类型配额批量生成提示词（例如 bug 修复*4）
- 需要将主仓复制为多个子仓并与提示词编号对齐

## 命令

| 命令 | 说明 |
|------|------|
| clone | 默认命令。根据一个或多个项目 ID 逐个拉取主仓到本地 |
| info | 仅输出项目名和仓库地址，不执行 clone |
| generate | 根据项目和类型配额生成提示词文件，并复制对应子仓 |
| append | 向已有提示词文件末尾追加一轮内容（兼容旧流程） |

## 默认配置

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| GitLab Host | gitlab.jzxhnh.com | 公司私有 GitLab 地址 |
| GitLab Group | prompt2repo | 项目所属 Group |
| 项目 ID 补全规则 | 不足 5 位前面补 0 | 如 1849 -> label-01849 |
| 项目名前缀 | label- | 默认项目名前缀 |
| 认证方式 | HTTPS + Access Token | 禁止使用 SSH 克隆 |
| 默认下载目录 | D:\charles\program\ai\apps\02.work session\session-4\gitlab source\ | 主仓与子仓根目录 |
| 结果目录 | D:\charles\program\ai\apps\02.work session\session-4\ai-model-result\ | 提示词结果根目录 |
| 文件名前缀 | A | 固定前缀 |
| index 规则 | 全局累计 01-19 | 本轮最多 19 条 |
| GitLab Access Token | 由用户在首次使用时提供，不在 skill 中存储 | HTTPS 克隆认证，用户名固定为 oauth2 |

## 执行流程

### clone

1. 用户输入一个或多个项目 ID。
2. 系统补全项目 ID，生成项目名。
3. 逐个拼接 GitLab 仓库地址。
4. 逐个确认本地主仓目录并执行 `git clone`。
5. 验证主仓目录存在且包含 `.git`。
6. 初始化项目级目录结构（结果根目录与子仓根目录）。
7. 向用户返回仓库地址和落盘目录。

### info

1. 用户输入一个或多个项目 ID。
2. 系统补全项目名。
3. 输出项目名与仓库地址。
4. 不执行 clone，不写文件。

### generate

1. 用户输入项目 ID（或项目名）与类型配额，例如：

```text
项目：1035
bug 修复*5
0-1 代码生成*5
Feature 迭代*5
代码理解*1
代码重构*1
工程化*1
代码测试*1
```

2. 标准化项目名（如 `1035 -> label-01035`，带后缀时如 `1035 -plus -> label-01035-plus`）。
3. 若主仓不存在，先执行 clone 拉取主仓。
4. 校验本轮总数不超过 19；index 使用全局累计并补零（01-19）。
5. **调用 `PromptArchitect` agent**，传入项目名、类型配额和代码目录上下文，由其生成每条提示词内容。
6. 以项目/类型两级目录写入提示词文件，文件名格式：`A-<项目名>-<类型>-<index>.md`；提示词内容来自上一步 PromptArchitect 的输出。
7. 每个提示词文件正文必须包含同名标识串：`A-<项目名>-<类型>-<index>`。
8. 按提示词逐条复制主仓到对应子仓目录（子仓与主仓同级并列，禁止嵌套在主仓内部），确保一条提示词对应一个子仓。
9. **子仓复制完成后，禁止再对任何子仓进行任何操作**（包括文件修改、代码变更、git 操作、造 bug 等）。generate 流程到此结束，后续修改必须在新的独立会话中由用户明确指令。
10. 输出汇总：成功项、失败项、文件路径、子仓路径。

### append

1. 用户输入一个或多个项目 ID（或项目名）与目标类型，并提供追加内容。
2. 读取 `02.work session/session-4/ai-model-result/<项目名>/<项目名>-<类型>/` 下已有提示词文件。
3. 统计该类型目录内已有 index，确定下一条 index。
4. 在文件末尾追加如下模板：

```markdown
---

提示词内容：<提示词内容>

模型回答 trae session id：

模型回答内容：
```

5. 如果用户未提供提示词内容，则提示词字段留空，等待用户手动填写。
6. 向用户确认追加成功，说明 index 和文件路径。

## 输入规则

支持的项目 ID 输入形式：

- `1849`
- `1849,1850,1851`
- `1849 1850 1851`
- 一行一个项目 ID

项目名生成规则：

```bash
PADDED=$(printf "%05d" <项目ID>)
PROJECT_NAME="label-${PADDED}"
```

如果用户明确指定项目名，则以用户提供的项目名为准。

项目名后缀规则：

- 用户可在项目名后追加自定义后缀，例如 `-plus`，格式为 `label-<ID>-<后缀>`。
- 后缀影响所有目录名、文件名及提示词文件内容中的项目名标识串。
- 主仓 `.git` 目录及源码文件内容（app.js、package.json 等）保持原始仓库内容不变，不随后缀修改。
- 示例：`1968 -plus` → 项目名 `label-01968-plus`，主仓仍 clone 自 `label-01968`。

类型输入支持：

- `bug 修复*3`
- `0-1 代码生成*3`
- `代码生成*3`（`0-1 代码生成` 的简写）
- `Feature 迭代*3`
- `代码理解*1`
- `代码重构*1`
- `工程化*1`
- `代码测试*1`

## 路径规则

- 主仓目录：`02.work session/session-4/gitlab source/<项目名>-plus/<项目名>/`
- 类型子仓目录：`02.work session/session-4/gitlab source/<项目名>-plus/<项目名>-<类型>/`
- 单条提示词子仓：`02.work session/session-4/gitlab source/<项目名>-plus/<项目名>-<类型>/A-<项目名>-<类型>-<index>/`
- 模型结果根目录：`02.work session/session-4/ai-model-result/<项目名>-plus/`
- 类型结果目录：`02.work session/session-4/ai-model-result/<项目名>-plus/<项目名>-<类型>/`
- 单条提示词文件：`02.work session/session-4/ai-model-result/<项目名>-plus/<项目名>-<类型>/A-<项目名>-<类型>-<index>.md`

> ⚠️ **子仓与主仓同级并列，禁止嵌套在主仓内部。** 子仓目录放在 `<项目名>-plus/<项目名>-<类型>/` 下，与主仓 `<项目名>-plus/<项目名>/` 并列在同一 `<项目名>-plus/` 目录中，绝不能出现在主仓目录的子级路径中。

## 提示词文件初始化

在 generate 写入完成后，需要检查：

- `02.work session/session-4/ai-model-result/<项目名>/`

如果目录不存在：

- 创建项目目录与各类型目录（若不存在）
- 按配额写入提示词文件模板

模板示例：

```markdown
A-label-01035-代码生成-01

用户第一次提示词：<prompt 内容>

模型第一次回答 trae session id：

模型第一次回答内容：



用户第二次提示词：

模型第二次回答 trae session id：

模型第二次回答内容：



用户第三次提示词：

模型第三次回答 trae session id：

模型第三次回答内容：



用户第四次提示词：

模型第四次回答 trae session id：

模型第四次回答内容：


用户第五次提示词：

模型第五次回答 trae session id：

模型第五次回答内容：
```

说明：
- 提示词内容默认填充到「用户第一次提示词」字段。
- 第二次、第三次、第四次、第五次提示词及回答字段留空占位，供用户手动补充。

如果目标文件已存在，则跳过，不覆盖。

## 认证说明

- GitLab SSH 禁止使用
- 必须使用 HTTPS + Access Token
- 用户名固定为 `oauth2`
- Token 需具备 `read_repository` 或 `api` 权限
- **Token 不在 skill 文件中存储**，每次使用前由用户提供（可通过环境变量 `GITLAB_TOKEN` 传入，或在首次 clone 时直接输入）
- 对外展示地址时，token 必须替换为 `***`
- 若 `GITLAB_TOKEN` 环境变量已设置，直接使用；否则在执行 clone 前提示用户输入

## 异常处理

1. 若标准仓库路径拉取失败，检查是否存在项目名拼写异常，例如 `lable-00994`。
2. 若怀疑路径异常，可通过 GitLab API 搜索项目。
3. 若目标目录已存在，必须先征求用户确认，不得直接覆盖。
4. 若提示词文件已存在相近版本，必须先询问用户是否保留旧内容。
5. 批量输入时，单个项目失败不应阻塞其他项目，但最终汇总必须标明失败项。
6. Bug 修复类若未绑定独立子仓，必须中止执行并提示修正。

## 示例

### 输入

```text
项目：1035
命令：generate
bug 修复*5
0-1 代码生成*5
Feature 迭代*5
代码理解*1
代码重构*1
工程化*1
代码测试*1
```

### 输出

```text
项目名：label-01035
主仓目录：D:\charles\program\ai\apps\02.work session\session-4\gitlab source\label-01035-plus\label-01035
类型目录：D:\charles\program\ai\apps\02.work session\session-4\ai-model-result\label-01035\label-01035-bug修复\
提示词文件：A-label-01035-bug修复-01.md ... A-label-01035-代码测试-19.md
子仓目录：D:\charles\program\ai\apps\02.work session\session-4\gitlab source\label-01035-plus\label-01035-bug修复\A-label-01035-bug修复-01\ ...
注意：子仓与主仓并列在 label-01035-plus/ 下，非主仓子目录；复制完成后不再对子仓做任何操作。
```

## 注意事项

1. 默认先拉取单份主仓到 `02.work session/session-4/gitlab source/<项目名>-plus/<项目名>/`。
2. 结果文件必须落在 `02.work session/session-4/ai-model-result/<项目名>-plus/<项目名>-<类型>/` 下。
3. 子仓必须落在 `02.work session/session-4/gitlab source/<项目名>-plus/<项目名>-<类型>/` 下，与主仓 `<项目名>-plus/<项目名>/` 同级并列，并与提示词编号一一对应。
4. **⚠️ 子仓禁止嵌套在主仓目录内部。** 子仓目录与主仓目录必须在同一父级 `<项目名>-plus/` 下并列存放，绝不能作为主仓的子目录。嵌套会导致主仓 `.git` 状态被污染、子仓代码被 git 忽略或误追踪。
5. **⚠️ 子仓复制完成后，禁止再对任何子仓进行任何操作（包括但不限于：文件修改、代码变更、git 操作、造 bug 等）。** 子仓复制是 generate 流程的最后一步，复制完成后整个 generate 命令即告结束。后续所有代码修改工作（包括 Bug 修复类型的设计 bug）必须在新的独立会话中、由用户明确指令后才可进行，本技能不再触碰子仓内容。
6. Bug 修复相关改动只能发生在对应 bug 子仓内，且只能在子仓复制完成后的后续会话中执行，不在 generate 流程中自动进行。