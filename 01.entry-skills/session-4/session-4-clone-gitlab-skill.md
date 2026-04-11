---
name: session-4-clone-gitlab
description: "Use when: 需要根据一个或多个 GitLab 项目 ID 拉取代码、拼接仓库地址、确认项目名并执行单份 clone，并在拉取完成后初始化模型结果记录文件。适用于从公司 GitLab 用 HTTPS + Access Token 拉取 prompt2repo 下的项目到 02.work session/session-4/gitlab source 目录。"
---

# GitLab 拉取代码技能

## 功能概述

这个技能用于处理 session-4 的项目拉取入口。它负责把 GitLab 项目拉到本地，并初始化模型结果记录文件。

这个技能负责：

- 根据项目 ID 补全项目名
- 拼接 GitLab 仓库地址并执行 clone
- 初始化 `02.work session/session-4/ai-model-result/<项目名>.md`
- 向已有结果文件追加新一轮对话模板（append 命令）

这个技能不负责：

- 多份对比克隆
- GitHub 仓库创建与推送
- 本地分支创建、切换、跟踪

## 使用场景

- 用户给出一个或多个 GitLab 项目 ID
- 需要按统一规则拉取到 session-4 工作区
- 需要在已有结果文件末尾追加新一轮对话模板，供下一次 Trae 会话填写

## 命令

| 命令 | 说明 |
|------|------|
| clone | 默认命令。根据一个或多个项目 ID 逐个拉取代码到本地 |
| info | 仅输出项目名和仓库地址，不执行 clone |
| append | 向已有 ai-model-result/<项目名>.md 末尾追加下一轮对话模板 |

## 默认配置

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| GitLab Host | gitlab.jzxhnh.com | 公司私有 GitLab 地址 |
| GitLab Group | prompt2repo | 项目所属 Group |
| 项目 ID 补全规则 | 不足 5 位前面补 0 | 如 1849 -> label-01849 |
| 项目名前缀 | label- | 默认项目名前缀 |
| 认证方式 | HTTPS + Access Token | 禁止使用 SSH 克隆 |
| 默认下载目录 | D:\charles\program\ai\apps\02.work session\session-4\gitlab source\ | 仓库落盘目录 |
| 结果目录 | D:\charles\program\ai\apps\02.work session\session-4\ai-model-result\ | 模型结果目录 |
| GitLab Access Token | glpat-JJiSr7nMVsWDidJozLD2mG86MQp1OjJuCA.01.0y156vhp6 | HTTPS 克隆认证，用户名固定为 oauth2 |

## 执行流程

### clone

1. 用户输入一个或多个项目 ID。
2. 系统补全项目 ID，生成项目名。
3. 逐个拼接 GitLab 仓库地址。
4. 逐个确认本地目录并执行 `git clone`。
5. 验证仓库目录存在且包含 `.git`。
6. 为每个项目初始化 `ai-model-result/<项目名>.md`。
7. 向用户返回仓库地址和落盘目录。

### info

1. 用户输入一个或多个项目 ID。
2. 系统补全项目名。
3. 输出项目名与仓库地址。
4. 不执行 clone，不写文件。

### append

1. 用户输入一个或多个项目 ID（或项目名），并提供下一轮对话的提示词内容。
2. 读取 `02.work session/session-4/ai-model-result/<项目名>.md`，确认文件存在。
3. 统计文件中已有的对话轮次（即"用户第X次提示词"出现次数），确定新轮次编号 N+1。
4. 在文件末尾追加如下模板：

```markdown
---

用户第 N+1 次提示词：<提示词内容>

模型第 N+1 次回答 trae session id：

模型第 N+1 次回答内容：
```

5. 如果用户未提供提示词内容，则提示词字段留空，等待用户手动填写。
6. 向用户确认追加成功，说明新轮次编号和文件路径。

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

## 路径规则

- 仓库代码：`02.work session/session-4/gitlab source/<项目名>/`
- 模型结果：`02.work session/session-4/ai-model-result/<项目名>.md`

## 模型结果文件初始化

在 prompt 写入完成后，需要检查：

- `02.work session/session-4/ai-model-result/<项目名>.md`

如果文件不存在：

- 创建 `ai-model-result/` 文件夹（若不存在）
- 写入初始结果记录模板（提示词字段留空，由用户手动填写）

模板示例：

```markdown
用户第一次提示词：<第 1 条 prompt 内容>

模型第一次回答 trae session id：

模型第一次回答内容：

---

用户第二次提示词：<第 2 条 prompt 内容>

模型第二次回答 trae session id：

模型第二次回答内容：
```

如果文件已存在，则跳过，不覆盖。

## 认证说明

- GitLab SSH 禁止使用
- 必须使用 HTTPS + Access Token
- 用户名固定为 `oauth2`
- Token 需具备 `read_repository` 或 `api` 权限
- 对外展示地址时，token 必须替换为 `***`

## 异常处理

1. 若标准仓库路径拉取失败，检查是否存在项目名拼写异常，例如 `lable-00994`。
2. 若怀疑路径异常，可通过 GitLab API 搜索项目。
3. 若目标目录已存在，必须先征求用户确认，不得直接覆盖。
4. 若 prompt 已存在相近版本，必须先询问用户是否保留旧内容。
5. 批量输入时，单个项目失败不应阻塞其他项目，但最终汇总必须标明失败项。

## 示例

### 输入

```text
项目 ID：1849, 1850
命令：clone
```

### 输出

```text
项目名：label-01849
仓库地址：https://oauth2:***@gitlab.jzxhnh.com/prompt2repo/label-01849.git
本地目录：D:\charles\program\ai\apps\02.work session\session-4\gitlab source\label-01849\

项目名：label-01850
仓库地址：https://oauth2:***@gitlab.jzxhnh.com/prompt2repo/label-01850.git
本地目录：D:\charles\program\ai\apps\02.work session\session-4\gitlab source\label-01850\
```

## 注意事项

1. 默认按项目名同名目录逐个拉取到 `02.work session/session-4/gitlab source/` 下。
2. clone 完成后必须为每个项目检查并创建结果文件。