---
name: session-4-clone-gitlab
description: "Use when: 需要根据一个或多个 GitLab 项目 ID 拉取代码、拼接仓库地址、确认项目名并执行单份 clone，并在拉取完成后调用 PromptArchitect agent 生成提示词。适用于从公司 GitLab 用 HTTPS + Access Token 拉取 prompt2repo 下的项目到 session-4 目录。"
---

# GitLab 拉取代码技能

## 功能概述

根据用户提供的一个或多个项目 ID，自动补全项目名并拼接 GitLab 仓库地址，使用 HTTPS + Access Token 将代码拉取到本地目录。拉取完成后，继续调用 PromptArchitect agent，基于各自项目内容生成可直接使用的提示词，并将结果写入 prompts 目录下对应的 session 文件。

该技能只处理 GitLab 拉取代码，不处理以下事项：

- 多份对比克隆
- GitHub 仓库创建与推送
- 本地分支创建、切换、跟踪

---

## 使用场景

- 用户提供一个或多个项目 ID，例如 1849 或 1849, 1850，需要从 GitLab 拉取代码
- 需要按固定规则补全项目名并构造仓库地址
- 需要在 session-4 默认目录下逐个拉取仓库代码
- 需要在代码拉取完成后，基于每个仓库内容生成对应的高精度提示词
- 需要将生成结果沉淀到 prompts/session-4.md 中，便于后续追加和复用

---

## 命令

| 命令 | 说明 |
|------|------|
| clone | 默认命令。根据一个或多个项目 ID 逐个拉取代码到本地 |
| info | 仅输出一个或多个项目 ID 对应的项目名和仓库地址，不执行 clone |

---

## 默认配置

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| GitLab Host | gitlab.jzxhnh.com | 公司私有 GitLab 地址 |
| GitLab Group | prompt2repo | 项目所属 Group |
| 项目 ID 补全规则 | 不足 5 位前面补 0 | 如 1849 -> label-01849；994 -> label-00994 |
| 项目名前缀 | label- | 默认项目名前缀 |
| 认证方式 | HTTPS + Access Token | 禁止使用 SSH 克隆 |
| 默认下载目录 | D:\charles\program\ai\apps\session-4\ | 拉取代码的根目录 |
| 默认子文件夹名 | 项目名本身 | 如 label-01849 |
| GitLab Access Token | glpat-JJiSr7nMVsWDidJozLD2mG86MQp1OjJuCA.01.0y156vhp6 | HTTPS 克隆认证，用户名固定为 oauth2 |

---

## 执行流程

### clone

```text
1. 用户输入一个或多个项目 ID，例如 1849 或 1849,1850,1851
2. 逐个补全项目 ID，生成项目名，例如：label-01849、label-01850、label-01851
3. 逐个构造仓库地址：
   https://oauth2:<token>@gitlab.jzxhnh.com/prompt2repo/<项目名>.git
4. 如用户未特别指定，项目名使用默认补全结果；若用户要改名，按项目逐个确认
5. 自动确定每个项目的本地目录：
   D:\charles\program\ai\apps\session-4\<项目名>\
6. 逐个执行 git clone
7. 逐个验证目标目录存在，且包含 .git
8. 逐个调用 PromptArchitect agent，基于各自已拉取项目生成提示词
9. 将每个项目的项目名和提示词写入 prompts/session-4.md
10. 若 prompts/session-4.md 已存在，则以问答追加方式逐条写入，不覆盖原内容
11. 逐个检查 session-4/ai-model-result/<项目名>.md 是否存在：
    - 不存在：创建 session-4/ai-model-result/ 文件夹（若不存在），新建 <项目名>.md，将 prompts/session-4.md 中该项目下的所有 prompt 条目（第 1 条、第 2 条等）全部写入，每条对应一组「提示词 + 回答 session id + 回答内容」字段，session id 和内容留空
    - 已存在：跳过，不覆盖
12. 向用户输出各项目的目录位置、仓库地址和生成的提示词结果（token 需脱敏显示）
```

### info

```text
1. 用户输入一个或多个项目 ID
2. 自动补全各项目 ID，生成项目名
3. 逐个输出项目名和仓库地址
4. 不执行任何下载操作
```

---

## AI 执行规范

### Step 1：解析多个项目 ID 并生成项目名

支持的输入形式：

- 1849
- 1849,1850,1851
- 1849 1850 1851
- 一行一个项目 ID

```bash
PADDED=$(printf "%05d" <项目ID>)
PROJECT_NAME="label-${PADDED}"
```

批量模式下应先将输入拆分为项目 ID 列表，再逐个处理。每个项目都独立生成项目名。

如果用户明确要求修改项目名，则以用户提供的项目名为准。

### Step 2：逐个拼接仓库地址

```bash
REPO_URL="https://oauth2:${TOKEN}@gitlab.jzxhnh.com/prompt2repo/${PROJECT_NAME}.git"
```

对用户展示地址时，token 必须脱敏：

```text
https://oauth2:***@gitlab.jzxhnh.com/prompt2repo/<项目名>.git
```

### Step 3：逐个执行克隆

```powershell
$projectIds = @("1849", "1850")

foreach ($projectId in $projectIds) {
   $padded = "{0:D5}" -f [int]$projectId
   $projectName = "label-$padded"
   $targetDir = "D:\charles\program\ai\apps\session-4\$projectName"
   $repoUrl = "https://oauth2:<token>@gitlab.jzxhnh.com/prompt2repo/$projectName.git"

   if (Test-Path $targetDir) {
      Write-Output "目标目录已存在：$targetDir"
   } else {
      git clone $repoUrl $targetDir
   }
}
```

如果某个项目的目标目录已存在，先针对该项目提示用户选择：

- 跳过
- 覆盖
- 改用新目录名

未得到确认前，不要删除已有目录。

### Step 4：逐个验证结果

```powershell
foreach ($projectDir in $projectDirs) {
   Test-Path "$projectDir\.git"
   Get-ChildItem $projectDir -Force | Select-Object -First 10
}
```

验证成功后，向用户汇报：

- 项目名
- 本地目录
- 脱敏后的仓库地址
- 是否成功拉取

### Step 5：逐个调用 PromptArchitect 生成提示词

在确认仓库已成功拉取后，必须对每个成功拉取的项目分别调用 PromptArchitect agent。调用目标是让 agent 基于当前项目内容生成一份可直接用于后续开发或任务执行的高质量提示词。

PromptArchitect 在执行该步骤时，必须遵循 workspace 指令文件 .github/instructions/promptarchitect-session-4.instructions.md。

执行要求：

- agent 名称必须精确使用 PromptArchitect
- 明确告知 agent 当前项目路径、项目名和任务目标
- 要求 agent 输出一份去 AI 味、可直接复用的专家级提示词
- 若仓库中存在 README、需求文档、配置文件或主要源码目录，应作为提示词生成的上下文依据
- 生成完成后，必须检查 prompts/session-4.md 中是否已存在当前项目对应的、或语义相近的提示词
- 若已存在语义相近的提示词，必须先询问用户是否保留旧版本、替换旧版本，或两者都保留
- 批量模式下，重复检查和保留确认必须逐个项目执行，不能一次性跳过

示例调用说明：

```text
请使用 PromptArchitect agent，基于已拉取的项目目录和代码结构，逐个项目生成可直接用于该项目开发任务的高精度提示词。提示词应结合项目目标、代码结构、关键约束、输出要求和执行边界，避免空泛表述。生成前先检查 prompts/session-4.md 中是否已有当前项目或语义相近的提示词；如果有，先询问用户是否保留旧版本。生成后将每个项目的结果按问答格式写入 prompts/session-4.md。
```

### Step 6：逐个写入 prompts/session-4.md

输出文件规则：

- 保存目录固定为 d:\charles\program\ai\apps\prompts\
- session-4 对应文件名固定为 session-4.md
- 文件不存在时，新建文件并写入第一条项目记录
- 文件已存在时，按问答记录逐条追加到文件末尾，不覆盖已有内容
- 批量模式下，一个项目对应一条独立记录

推荐格式：

```markdown
# Session 4

## Q: <项目名>

### 项目路径
<本地目录>

### Prompt
<PromptArchitect 生成的提示词>
```

若文件已存在，则追加：

```markdown
---

## Q: <项目名>

### 项目路径
<本地目录>

### Prompt
<PromptArchitect 生成的提示词>
```

### Step 7：逐个创建模型结果记录文件

在 prompts/session-4.md 写入完成后，对每个项目执行以下操作：

1. 检查 `session-4/ai-model-result/<项目名>.md` 是否存在
2. 不存在时：
   - 创建 `session-4/ai-model-result/` 文件夹（若不存在）
   - 从 prompts/session-4.md 中读取该项目下的所有 prompt 条目（第 1 条、第 2 条……）
   - 新建 `<项目名>.md`，按条目数量依次写入，每条对应一组字段，session id 和回答内容留空：

```markdown
用户第一次提示词：<第 1 条 prompt 内容>

模型第一次回答 trae session id：

模型第一次回答内容：

---

用户第二次提示词：<第 2 条 prompt 内容>

模型第二次回答 trae session id：

模型第二次回答内容：
```

   - 若只有 1 条 prompt，只写第一组，不添加分隔线和第二组。
   - 若有 N 条，按序写入 N 组，组间用 `---` 分隔。

3. 已存在时：跳过，不覆盖。

最终向用户返回时，应包含：

- 各项目的拉取结果
- 各项目的本地目录
- 各项目的 PromptArchitect 生成结果
- prompts/session-4.md 的写入结果
- session-4/ai-model-result/<项目名>.md 的创建结果

---

## 认证说明

- GitLab SSH 虽可访问端口，但握手会失败，禁止使用 SSH clone
- 必须使用 HTTPS + Access Token
- 用户名固定为 oauth2
- Token 需具备 read_repository 或 api 权限
- 对用户展示任何命令或地址时，token 必须替换为 ***

---

## 异常处理

1. 若标准仓库路径拉取失败，检查是否存在项目名拼写异常，例如 lable-00994。
2. 若怀疑路径异常，可通过 GitLab API 搜索项目：

```bash
curl -s "https://gitlab.jzxhnh.com/api/v4/projects?private_token=<TOKEN>&search=<PADDED_ID>&per_page=10"
```

3. 若返回结果中存在实际项目路径，则应改用真实项目路径重新拼接仓库地址。
4. 若目标目录已存在，必须先征求用户确认，不得直接覆盖。
5. 若 prompts/session-4.md 中已经存在同一项目或语义接近的提示词，必须先询问用户是否保留旧内容，再决定追加还是替换。
6. 批量输入时，某一个项目失败不应阻塞其他项目继续处理，但最终汇总里要明确标出失败项。

---

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
本地目录：D:\charles\program\ai\apps\session-4\label-01849\
执行操作：git clone

项目名：label-01850
仓库地址：https://oauth2:***@gitlab.jzxhnh.com/prompt2repo/label-01850.git
本地目录：D:\charles\program\ai\apps\session-4\label-01850\
执行操作：git clone
```

---

## 注意事项

1. 本技能只负责 GitLab 拉取代码。
2. 不创建 GitHub 仓库，不推送 GitHub。
3. 不创建额外分支，不切换本地分支。
4. 如无特殊说明，默认按项目名同名目录逐个拉取。
5. 拉取成功后，必须继续调用 PromptArchitect agent 生成提示词，不能在 clone 完成后直接结束。
6. PromptArchitect 生成的提示词必须落盘到 prompts/session-4.md。
7. 如果 prompts/session-4.md 已存在，默认行为是追加，不是覆盖。
8. 追加前要先检查是否存在语义接近的旧提示词；发现重复时，先问用户是否保留。
9. 支持一次传入多个项目 ID，批量处理时按项目逐个执行、逐个确认、逐个写入。
10. prompts/session-4.md 写入完成后，必须为每个项目检查并创建 session-4/ai-model-result/<项目名>.md；文件已存在时跳过，不覆盖。