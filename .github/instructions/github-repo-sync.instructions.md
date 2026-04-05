---
description: "Use when: 用户提到推代码、拉代码、同步 GitHub、配置 origin、首次推送仓库、fetch、pull、push、SSH 地址、Personal Access Token、PAT，且希望按 attitudeshuai 账号下的 GitHub 仓库规范执行。优先加载 github-repo-sync skill。"
---

# GitHub 同步指引

当用户的请求涉及 GitHub 仓库同步时，优先加载 `.github/skills/github-repo-sync/SKILL.md`，并按其中流程执行。

适用触发词包括但不限于：

- 推代码
- 推送到 GitHub
- 首次推送仓库
- pull
- fetch
- push
- 同步远程仓库
- 配置 origin
- 改远程地址
- SSH 地址
- Personal Access Token
- PAT

执行要求：

1. 默认优先使用 SSH 地址：`git@github.com:attitudeshuai/<项目名>.git`
2. 只有在 SSH 不可用时，才回退到 HTTPS + PAT
3. 不要把真实 PAT 明文写入仓库文件；优先使用环境变量
4. 如果场景是本地初始仓库首次推送到 GitHub，推送完成后必须再执行一次 fetch 和 pull
5. 对已存在的 `origin`，先检查再修改，不要未经确认直接覆盖
6. 执行前先确认当前目录是否为 Git 仓库、当前分支、远程地址和工作区状态
7. 遇到冲突、未提交修改或远程地址不一致时，先说明风险，再继续执行

输出时至少应说明：

- 当前仓库路径
- 当前分支
- origin 地址
- 本次执行的是 push、pull、fetch 还是首次推送
- 是否使用 SSH
- 是否回退到 HTTPS + PAT
- 如果是首次推送，是否已补做 fetch 和 pull
