---
name: git-commit
description: 当用户要求快速生成 git commit message、自动提交当前变更、拆分提交、使用 Conventional Commits，或明确调用 git-commit 时使用。
---

# Git Commit

用最少上下文完成自动提交：先看文件级摘要，只在必要时读取精准 diff。默认会创建 commit。

## Message

- 英文、单行、祈使语气、30 个单词以内、不以句号结尾。
- 格式：`type: summary`；scope 只有明显有价值时才用。
- `type` 只选：`feat`、`fix`、`docs`、`refactor`、`test`、`chore`、`style`、`perf`、`ci`、`build`、`revert`。

## Fast Path

1. 先跑 `git status --short`；空则回复 `nothing to commit`。
2. 若有 staged changes，只处理 staged，除非用户明确要求包含全部变更。
3. 用 `git diff --cached --name-status` 和 `git diff --cached --stat` 理解 staged；无 staged 时改用 `git diff --name-status` 和 `git diff --stat`。
4. 先按路径、变更类型和主题分组；相关小改动合并成一个 commit，无关主题拆分。
5. 只有 type 或 summary 不明确时才读精准 diff：staged 用 `git diff --cached -- path`，unstaged 用 `git diff -- path`。
6. 每次只 stage 一个提交组，随后用 `git diff --cached --name-status` 和 `git diff --cached --stat` 快速核对。
7. 若核对仍不够，再读 `git diff --cached -- path`；确认后运行 `git commit -m "type: summary"`。
8. 重复直到没有可提交变更，最后只简述 commit messages 和剩余变更。

## Split Rules

- 拆分条件：不同目的、不同 type、或同一批提交会让 message 变泛。
- 不为很小且相关的文件强行拆分。
- 单文件 mixed hunks 只有在能安全非交互拆分时才拆；否则按文件粒度提交并说明限制。
- 不读完整文件，除非 diff 缺少必要上下文。

## Safety

- 不使用 `git reset --hard`、`git checkout --`、amend、rebase、force push。
- 不回滚、不覆盖用户变更；不为了拆分而改写文件内容。
- Git 命令失败就停止，报告失败命令和 blocker。
