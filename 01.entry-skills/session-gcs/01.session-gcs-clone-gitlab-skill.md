---
name: session-gcs-clone-gitlab
description: "Use when: 需要根据 GitLab 项目 ID 拉取源码、推送到 GitHub、创建三个对比分支（0414-original / 0414-dyad / 0414-trinity）、并生成单条提示词供多模型对比评测。适用于 session-gcs 场景。"
---

# GitLab 拉取 · GitHub 推送 · 多模型对比提示词技能

## 功能概述

这个技能用于处理 session-gcs 的项目拉取、GitHub 同步与单条提示词生成入口。核心目标是为三个模型（`0414-original`、`0414-dyad`、`0414-trinity`）提供完全一致的起点代码，并配套一条经去 AI 化优化的提示词。

对比组合：

- `0414-original` VS `0414-dyad`
- `0414-original` VS `0414-trinity`

这个技能负责：

- 根据项目 ID 补全项目名
- 拼接 GitLab 仓库地址并执行 clone（origin 仓）
- 将源码推送到 GitHub 公共仓库，仓库名为 `<项目名>`
- 在 GitHub 上创建三个对比分支并推送
- 将三个分支分别 clone 到本地独立目录
- 对 origin 仓代码进行类型分析，由用户确认任务类型
- 调用 `PromptArchitect` agent 生成**单条**提示词
- 调用 `humanizer-zh` agent 对提示词进行去 AI 化优化
- 将最终提示词写入对应结果目录

这个技能不负责：

- 对三个分支仓库进行任何代码修改
- 执行模型测试或记录模型回答
- 多条提示词批量生成（session-gcs 每次只生成一条）

## 使用场景

- 用户给出一个 GitLab 项目 ID
- 需要同时对比三个模型版本的能力
- 需要一条经去 AI 化优化的提示词作为共同输入

## 命令

| 命令 | 说明 |
|------|------|
| setup | 默认命令。拉取 GitLab 源码→确认任务类型→（Bug 修复类则在 origin 注入 bug）→推送 GitHub→创建并拉取三个对比分支→**自动进入 generate 流程** |
| generate | 传入已确认的类型，生成并优化单条提示词、写入结果文件 |
| info | 仅输出项目名和仓库地址，不执行任何操作 |

## 默认配置

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| GitLab Host | gitlab.jzxhnh.com | 公司私有 GitLab 地址 |
| GitLab Group | prompt2repo | 项目所属 Group |
| 项目 ID 补全规则 | 不足 5 位前面补 0 | 如 1849 → label-01849 |
| 项目名前缀 | label- | 默认项目名前缀 |
| GitLab 认证方式 | HTTPS + Access Token | 禁止使用 SSH 克隆 |
| GitLab Access Token | 由用户在首次使用时提供，不在 skill 中存储 | 用户名固定为 oauth2 |
| GitHub 用户名 | attitudeshuai | GitHub 推送账号 |
| GitHub 仓库名 | `<项目名>` | 推送到 GitHub 时使用 |
| GitHub 可见性 | public | 所有 GitHub 仓库设为公共访问 |
| GitHub 连接方式 | 优先 SSH，不可用时回退 HTTPS + PAT | 遵循 github-repo-sync skill |
| 对比分支 | 0414-original / 0414-dyad / 0414-trinity | 三个分支从 main 创建 |
| 提示词数量 | 每次仅生成 1 条 | session-gcs 固定单条 |
| 提示词文件数量 | 每个类型目录生成 6 份 | 每个模型对应 1 份对话内容 + 1 份评价结果，共 3 模型 × 2 = 6 份 |
| 本地根目录 | `02.work session/session-gcs/gitlab source/` | origin 仓与分支仓根目录 |
| 结果根目录 | `02.work session/session-gcs/ai-model-result/` | 提示词结果根目录 |
| 文件名前缀 | A | 固定前缀 |

## 执行流程

### setup（默认命令）

1. 用户输入项目 ID。
2. 补全项目 ID，生成项目名（如 `1035 → label-01035`）。
3. 拼接 GitLab HTTPS 地址，执行 clone 到本地 origin 仓目录。
4. 验证 origin 仓目录存在且包含 `.git`。
5. 扫描 origin 仓项目结构，展示初步分析摘要，**需要用户从以下 7 种类型中确认一种**：

   | 类型 | 说明 |
   |------|------|
   | Bug 修复 | 定位并修复现有 bug |
   | 0-1 代码生成 | 从零开始实现新功能或新模块 |
   | Feature 迭代 | 在现有功能基础上追加小功能 |
   | 代码理解 | 解释逻辑、梳理调用链、分析风险 |
   | 工程化 | 环境、构建、配置、CI/CD 相关 |
   | 代码重构 | 性能、可读性、解耦，不改变行为 |
   | 代码测试 | 覆盖正常/异常/边界三类路径的测试 |

   **类型未经用户确认，禁止进行后续任何操作。**

6. 用户确认类型后，根据类型走不同分支：

   **若类型为『Bug 修复』：**
   - 调用 `PromptArchitect` agent，根据代码设计一个想实际线上会出现的 bug（逻辑、业务、状态、权限等）。
   - 将 bug 注入 **origin 仓本地文件**，做最小验证（29 类型分析确认可复现）。
   - 在 origin 仓内提交 bug：
     ```bash
     git add -A
     git commit -m "inject bug for gcs evaluation"
     ```
   - **bug 注入完成后，禁止再对 origin 仓做任何代码修改。**

   **若类型为其他 6 种：**
   - origin 仓保持 GitLab 原始状态，直接进入下一步。

7. 在 origin 仓目录内，按照 **github-repo-sync skill** 的流程配置 GitHub 远程地址、首次推送到 `<项目名>` 仓库（设为 public）。

8. 推送完成后，在 GitHub 上从 main 创建三个对比分支：

   ```bash
   git push origin main:0414-original
   git push origin main:0414-dyad
   git push origin main:0414-trinity
   ```

   > 若类型为 Bug 修复，three 个分支因继承自 main 而自动包含相同的 bug，无需逐个分支单独操作。

9. 将三个分支分别 clone 到本地独立目录（从 GitHub 拉取）：

   ```bash
   # 0414-original
   git clone -b 0414-original git@github.com:attitudeshuai/<项目名>.git \
     "02.work session/session-gcs/gitlab source/<项目名>/<项目名>-0414-original"

   # 0414-dyad
   git clone -b 0414-dyad git@github.com:attitudeshuai/<项目名>.git \
     "02.work session/session-gcs/gitlab source/<项目名>/<项目名>-0414-dyad"

   # 0414-trinity
   git clone -b 0414-trinity git@github.com:attitudeshuai/<项目名>.git \
     "02.work session/session-gcs/gitlab source/<项目名>/<项目名>-0414-trinity"
   ```

10. 验证四个本地目录均存在（origin + 三个分支）。
11. 输出汇总：GitLab 地址、GitHub 地址、本地目录列表、分支状态、**确认的任务类型**。
12. **setup 完成后，立即自动执行 generate 流程**，无需用户再次输入命令。

### generate

1. 用户传入 setup 阶段已确认的任务类型（如 `类型:工程化`）。
   - 若未传入类型，必须让用户手动确认，不能跳过。
2. 读取 origin 仓代码，结合确认的类型**调用 `PromptArchitect` agent**，传入项目名、类型（配额固定 `*1`）和 origin 仓代码目录上下文：
   - Bug 修复类型：代码中已有 bug，prompt 描述 bug 现象和触发条件，**不要把 bug 代码位置或修复方案交交。**
   - 其他类型：按类型策略正常出题。
3. 将 PromptArchitect 输出的提示词**交给 `humanizer-zh` agent** 进行去 AI 化优化，确保语气自然、口语化，像真实工作里的需求或抛出的问题。
4. 根据项目上下文和任务类型，从以下 5 种约束类型中**任选 3–4 个**，为提示词添加约束标签并附到提示词正文末尾：

   | 约束类型 | 示例内容 |
   |----------|----------|
   | 技术栈或依赖约束 | 项目已有 xlsx.js，不要引入其他依赖 |
   | 架构或模式约束 | 现有项目是原生 JS，不要引入框架或构建工具 |
   | 代码风格或规范约束 | 保持现有的函数命名风格，不要改名 |
   | 非代码回复约束 | 只输出需要修改的文件，不要全量输出 |
   | 业务逻辑约束 | 已中奖记录不允许被修改 |

   选择原则：选与项目实际情况和任务类型匹配的约束，不要填充无关约束。Bug 修复类型必选『非代码回复约束』（不要暴露修复方向）。
5. 将优化后的提示词（含约束标签）写入 **6 个**结果文件（见路径规则）：
   - **对话内容文件 × 3**：内容完全一致，文件名后缀分别为 `-original-对话内容`、`-dyad-对话内容`、`-trinity-对话内容`，供模型填写对话记录。
   - **评价结果文件 × 3**：内容为空白评价模板，文件名后缀分别为 `-original-评价结果`、`-dyad-评价结果`、`-trinity-评价结果`，供人工填写评分与结论。
6. ⚠️ **写入完成即为 generate 流程的终点。禁止对三个分支仓库进行任何操作（包括但不限于：文件修改、代码变更、git 操作）。**

### info

1. 用户输入一个或多个项目 ID。
2. 补全项目名。
3. 输出 GitLab 地址、GitHub 地址（`<项目名>`）、本地目录结构预览。
4. 不执行任何 clone、push、写文件操作。

## 路径规则

```
# origin 仓（GitLab 源码）
02.work session/session-gcs/gitlab source/<项目名>/<项目名>-origin/

# 三个分支仓（从 GitHub clone）
02.work session/session-gcs/gitlab source/<项目名>/<项目名>-0414-original/
02.work session/session-gcs/gitlab source/<项目名>/<项目名>-0414-dyad/
02.work session/session-gcs/gitlab source/<项目名>/<项目名>-0414-trinity/

# 模型结果根目录
02.work session/session-gcs/ai-model-result/<项目名>/

# 类型结果目录
02.work session/session-gcs/ai-model-result/<项目名>/<项目名>-<类型>/

# 对话内容文件（3 份，内容一致，供模型填写对话记录）
02.work session/session-gcs/ai-model-result/<项目名>/<项目名>-<类型>/A-<项目名>-<类型>-original-对话内容.md
02.work session/session-gcs/ai-model-result/<项目名>/<项目名>-<类型>/A-<项目名>-<类型>-dyad-对话内容.md
02.work session/session-gcs/ai-model-result/<项目名>/<项目名>-<类型>/A-<项目名>-<类型>-trinity-对话内容.md

# 评价结果文件（3 份，供人工填写评分与结论）
02.work session/session-gcs/ai-model-result/<项目名>/<项目名>-<类型>/A-<项目名>-<类型>-original-评价结果.md
02.work session/session-gcs/ai-model-result/<项目名>/<项目名>-<类型>/A-<项目名>-<类型>-dyad-评价结果.md
02.work session/session-gcs/ai-model-result/<项目名>/<项目名>-<类型>/A-<项目名>-<类型>-trinity-评价结果.md
```

> ⚠️ **origin 仓与三个分支仓同级并列，均在 `<项目名>/` 下，禁止相互嵌套。**

## 提示词文件模板

generate 完成后，每个类型目录下生成 **6 份**文件：3 份对话内容 + 3 份评价结果。

### 对话内容文件模板（`-对话内容.md`，3 份内容完全一致）

```markdown
A-<项目名>-<类型>-01

用户第一次提示词：<humanizer-zh 优化后的提示词内容>

约束标签：
- 技术栈或依赖约束：<具体约束内容>
- 架构或模式约束：<具体约束内容>
- 代码风格或规范约束：<具体约束内容>
- 非代码回复约束：<具体约束内容>

注：约束标签从上述 5 种类型中任选 3–4 个，固定格式为『约束类型：内容』。

模型第一次回答 trae session id：

模型第一次回答内容：



用户第二次提示词：

模型第二次回答内容：



用户第三次提示词：

模型第三次回答内容：
```

### 评价结果文件模板（`-评价结果.md`，每个模型独立一份）

```markdown
A-<项目名>-<类型>-<模型名>-评价结果

模型：0414-<模型名>

## 第一轮评价

提示词理解准确度：

技术深度：

回答完整性：

表达清晰度：

综合评分（1-5）：

评价备注：



## 第二轮评价

提示词理解准确度：

技术深度：

回答完整性：

表达清晰度：

综合评分（1-5）：

评价备注：



## 第三轮评价

提示词理解准确度：

技术深度：

回答完整性：

表达清晰度：

综合评分（1-5）：

评价备注：



## 总体评价

总分：

优势：

不足：

与其他模型对比结论：
```

说明：
- 对话内容文件：提示词内容填入「用户第一次提示词」字段，第二次、第三次提示词及回答字段留空占位，供用户手动补充。3 份内容完全一致，仅文件名后缀不同。
- 评价结果文件：3 份评分模板各自独立（模型名不同），内容均为空白，供人工逐条填写。
- 如果目标文件已存在，跳过，不覆盖。

## 输入规则

支持的项目 ID 输入形式：

- `1035`
- 一行一个项目 ID

项目名生成规则：

```bash
PADDED=$(printf "%05d" <项目ID>)
PROJECT_NAME="label-${PADDED}"
```

如果用户明确指定项目名，以用户提供的为准。

## GitHub 推送规范

本技能的 GitHub 相关操作全部遵循 **github-repo-sync skill** 中的规范：

- 优先使用 SSH（`git@github.com:attitudeshuai/<项目名>.git`）
- SSH 不可用时回退到 HTTPS + PAT（环境变量 `GITHUB_PAT`，不明文保存 token）
- 首次推送后必须执行一次 `git fetch origin` 和 `git pull origin main`
- GitHub 仓库创建时设为 **public**

## 认证说明

| 远程 | 方式 | 说明 |
|------|------|------|
| GitLab | HTTPS + Access Token | 用户名固定 `oauth2`，token 不存储在 skill 中，对外展示替换为 `***` |
| GitHub | SSH（优先）/ HTTPS + PAT | 遵循 github-repo-sync skill |

## 异常处理

1. 若 GitLab 拉取失败，检查项目名拼写（如 `lable-` 误拼），可通过 GitLab API 搜索项目。
2. 若 origin 目录已存在，必须先征求用户确认，不得直接覆盖。
3. 若 GitHub 仓库已存在同名仓库，提示用户确认是否使用现有仓库或重命名。
4. 若 SSH 不可用，自动回退到 HTTPS + PAT，并告知用户当前使用的认证方式。
5. **类型未经用户确认，禁止进行 GitHub 推送和分支操作。**
6. **Bug 修复类型的代码注入只在 origin 仓进行一次，推送 GitHub 后禁止再改。**
7. **generate 完成后，任何对分支仓库的代码操作均被视为违规，必须停止。**

## 示例

### 输入

```text
项目 ID：1035
命令：setup
```

### setup 输出

```text
项目名：label-01035
GitLab 地址：https://oauth2:***@gitlab.jzxhnh.com/prompt2repo/label-01035.git
GitHub 仓库：https://github.com/attitudeshuai/label-01035（public）
任务类型：工程化（公用代码未注入 bug）

本地目录：
  origin  : 02.work session/session-gcs/gitlab source/label-01035/label-01035-origin/
  分支仓库:
    0414-original : 02.work session/session-gcs/gitlab source/label-01035/label-01035-0414-original/
    0414-dyad     : 02.work session/session-gcs/gitlab source/label-01035/label-01035-0414-dyad/
    0414-trinity  : 02.work session/session-gcs/gitlab source/label-01035/label-01035-0414-trinity/

GitHub 分支：main / 0414-original / 0414-dyad / 0414-trinity ✓
```

> Bug 修复类型示例中，输出中也应显示：`任务类型：Bug 修复（bug 已注入 origin + 已 commit）`。

### generate 示例输出（用户确认类型为"工程化"后）

```text
提示词文件（3 份，内容一致）：
  # 对话内容（3 份，内容一致）
  02.work session/session-gcs/ai-model-result/label-01035/label-01035-工程化/A-label-01035-工程化-original-对话内容.md
  02.work session/session-gcs/ai-model-result/label-01035/label-01035-工程化/A-label-01035-工程化-dyad-对话内容.md
  02.work session/session-gcs/ai-model-result/label-01035/label-01035-工程化/A-label-01035-工程化-trinity-对话内容.md
  # 评价结果（3 份，空白模板）
  02.work session/session-gcs/ai-model-result/label-01035/label-01035-工程化/A-label-01035-工程化-original-评价结果.md
  02.work session/session-gcs/ai-model-result/label-01035/label-01035-工程化/A-label-01035-工程化-dyad-评价结果.md
  02.work session/session-gcs/ai-model-result/label-01035/label-01035-工程化/A-label-01035-工程化-trinity-评价结果.md

内容：<humanizer-zh 优化后的提示词>

⚠️ generate 流程结束，三个分支仓库未做任何修改。
```

## 注意事项

1. **session-gcs 每次固定只生成 1 条提示词**，不支持批量配额（如 `*5`）。
2. 任务类型必须在 setup 阶段、**GitHub 推送之前**确认；Bug 修复类型进一步必须在 origin 推送前注入 bug。
3. Bug 修复类型：bug 只在 origin 本地注入一次，commit 后推送到 GitHub main，三个对比分支因继承自 main 而天然包含相同的 bug，无需逐分支单独注入。
4. 三个分支仓是模型执行时的工作区，**本技能在 generate 完成后不再触碰**。
5. 每个类型目录下生成 6 份文件：3 份对话内容（`-original-对话内容` / `-dyad-对话内容` / `-trinity-对话内容`，内容一致）+ 3 份评价结果（`-original-评价结果` / `-dyad-评价结果` / `-trinity-评价结果`，空白模板）。
6. humanizer-zh 优化是必须步骤，不可跳过；PromptArchitect 的原始输出不直接写入文件。
7. GitHub 仓库可见性必须为 public，在 setup 阶段确认。
8. 对比逻辑记录：

   - `0414-original` VS `0414-dyad`（原始 VS 双端对话）
   - `0414-original` VS `0414-trinity`（原始 VS 三端对话）
