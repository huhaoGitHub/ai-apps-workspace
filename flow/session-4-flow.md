# Session 4 执行流程

## 1. 拉取项目

在 skill app 中发起对话，输入：

项目 id: 2532

执行时结合 session-4-clone-gitlab-skill.md，先完成项目拉取。

## 2. 确认提示词

打开 prompts/session-4.md，找到当前项目名对应的 prompt。

如果该项目下存在多个 prompt：

- 只保留本次要测试的 1 个 prompt
- 删除或移走另一个 prompt
- 这样后续做结果比对时不会混淆

## 3. 执行 prompt

用 trae-cn 打开项目。

将保留下来的 prompt 发给模型执行。

## 4. 记录模型结果

把第一个 solo coder 的 session id 和完整对话内容复制到对应结果文件中：

ai-model-result/label-02532.md

如果有多轮对话：

- 按轮次依次追加到同一个结果文件
- 每一轮都单独起一个清晰的开头
- 开头至少包含：用户提示词、模型回答、session id

## 5. 分析结果

分析时使用以下两个输入：

- ai-model-result/label-02532.md
- analysis-skill.md

分析目标：

- 判断模型是否完成 prompt 要求
- 记录未完成原因或过程中的不足
- 输出最终评价结果



