---
name: github-repo-sync
description: "Use when: 需要把本地 Git 仓库推送到 GitHub、从 GitHub 拉取代码、执行 fetch、配置 origin、处理首次推送到 attitudeshuai 账号下的仓库，且默认优先使用 SSH，SSH 不可用时再回退到 HTTPS + Personal Access Token。"
---

# GitHub 仓库推送与拉取技能

## 功能概述

这个技能用于整理本地仓库与 GitHub 仓库之间的常见同步动作，统一处理以下场景：

- 本地已有仓库，首次推送到 GitHub
- 已有关联远程仓库，继续 push
- 从 GitHub pull 最新代码
- 先 fetch 再判断差异
- SSH 不可用时，回退到 HTTPS + Personal Access Token

默认策略：

- 优先使用 SSH 地址
- 只有 SSH 不可用时，才使用 HTTPS + PAT
- 本地初始仓库首次推送到 GitHub 后，必须再执行一次 fetch 和 pull

## 默认配置

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| GitHub 用户名 | attitudeshuai | GitHub 用户名 |
| SSH 仓库地址 | git@github.com:attitudeshuai/<项目名>.git | 默认远程地址 |
| HTTPS 仓库地址 | https://github.com/attitudeshuai/<项目名>.git | SSH 不可用时的回退地址 |
| PAT 环境变量 | GITHUB_PAT | 不在仓库里明文保存 token |
| 默认分支 | main | 首次推送时默认使用 |

## 使用场景

### 1. 本地初始仓库首次推送到 GitHub

适用于：

- 本地目录已经是 Git 仓库
- 还没有 `origin`
- 需要把代码首次推送到 `attitudeshuai/<项目名>`

标准流程：

1. 检查当前目录是否为 Git 仓库。
2. 检查当前是否已存在 `origin`。
3. 不存在时，优先添加 SSH 远程地址。
4. 确认当前分支，必要时切到 `main`。
5. 执行首次推送：`git push -u origin main`。
6. 推送完成后，必须执行一次 `git fetch origin`。
7. 再执行一次 `git pull origin main`，确认本地与远程状态一致。

推荐命令：

```powershell
git remote add origin git@github.com:attitudeshuai/<项目名>.git
git branch -M main
git push -u origin main
git fetch origin
git pull origin main
```

### 2. 已有关联远程仓库，继续 push

适用于：

- 当前仓库已经配置了 `origin`
- 只需要把本地提交推上去

标准流程：

1. 检查当前分支。
2. 检查工作区是否干净。
3. 必要时先 `git fetch origin`。
4. 若本地落后远程，先 `git pull --rebase origin <当前分支>`。
5. 再执行 `git push origin <当前分支>`。

推荐命令：

```powershell
git fetch origin
git pull --rebase origin <当前分支>
git push origin <当前分支>
```

### 3. 从 GitHub 拉取代码

适用于：

- 本地已经存在仓库
- 需要同步远程最新代码

标准流程：

1. 先执行 `git fetch origin`。
2. 查看本地和远程分支差异。
3. 再执行 `git pull --rebase origin <当前分支>`。

推荐命令：

```powershell
git fetch origin
git status
git pull --rebase origin <当前分支>
```

### 4. 只获取远程更新，不合并

适用于：

- 只想看远程变化
- 暂时不想改动本地工作区

推荐命令：

```powershell
git fetch origin
git log --oneline HEAD..origin/<当前分支>
```

## 远程地址策略

### 优先使用 SSH

默认远程地址：

```text
git@github.com:attitudeshuai/<项目名>.git
```

适用前提：

- 本机已配置 SSH key
- 公钥已加入 GitHub 账号
- `ssh -T git@github.com` 可以正常通过

### SSH 不可用时回退到 HTTPS + PAT

回退地址模板：

```text
https://github.com/attitudeshuai/<项目名>.git
```

不要把 PAT 直接写进 skill 文件或仓库文件中。执行时使用环境变量：

```powershell
$env:GITHUB_PAT = "<你的 Personal Access Token>"
```

临时推送示例：

```powershell
git remote set-url origin https://attitudeshuai:$env:GITHUB_PAT@github.com/attitudeshuai/<项目名>.git
git push -u origin main
```

如果后续 SSH 恢复可用，应切回 SSH：

```powershell
git remote set-url origin git@github.com:attitudeshuai/<项目名>.git
```

## 执行规范

### Step 1：先识别当前仓库状态

执行前先检查：

```powershell
git rev-parse --is-inside-work-tree
git remote -v
git branch --show-current
git status
```

必须先确认以下信息：

- 当前目录是不是 Git 仓库
- 是否已存在 `origin`
- 当前分支名是什么
- 工作区是否有未提交修改

### Step 2：优先走 SSH

如果要新增远程地址，默认使用：

```powershell
git remote add origin git@github.com:attitudeshuai/<项目名>.git
```

如果已存在 `origin` 但地址不对，先检查后再改：

```powershell
git remote set-url origin git@github.com:attitudeshuai/<项目名>.git
```

### Step 3：首次推送必须补一次 fetch 和 pull

当场景是“本地初始仓库第一次推送到 GitHub”时，固定执行顺序：

```powershell
git branch -M main
git push -u origin main
git fetch origin
git pull origin main
```

这一步不能省略 `fetch` 和 `pull`。

### Step 4：常规同步先 fetch，再 pull 或 push

常规同步时，建议顺序固定为：

```powershell
git fetch origin
```

然后根据目标执行：

- 要更新本地：`git pull --rebase origin <当前分支>`
- 要推送远程：`git push origin <当前分支>`

## 异常处理

### 1. 当前目录不是 Git 仓库

先初始化：

```powershell
git init
git add .
git commit -m "chore: initial commit"
```

然后再进入首次推送流程。

### 2. 已存在 origin

不要直接覆盖，先查看：

```powershell
git remote get-url origin
```

若与目标仓库不一致，再确认是否替换。

### 3. SSH 失败

先验证：

```powershell
ssh -T git@github.com
```

若失败，再回退到 HTTPS + PAT，不要一开始就默认用 HTTPS。

### 4. pull 时有冲突

不要强推，不要直接覆盖。先停下来处理冲突，再继续 push。

### 5. 工作区有未提交修改

先让用户决定：

- 先提交再同步
- 暂存后同步
- 放弃本次 pull 或 push

## 输出要求

执行后返回结果时，至少要说明：

- 当前仓库路径
- 当前分支
- origin 地址
- 本次执行的是 push、pull、fetch 还是首次推送
- 是否走了 SSH
- 是否回退到 HTTPS + PAT
- 首次推送后是否已经补做 fetch 和 pull

## 注意事项

1. 默认优先 SSH，除非 SSH 验证失败。
2. 不要把真实 PAT 明文写进仓库文件。
3. 首次推送完成后，必须再执行一次 fetch 和 pull。
4. 常规同步建议固定先 fetch，再决定 pull 或 push。
5. 遇到已存在的 origin，不要未经确认直接覆盖。
