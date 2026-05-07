# Bugfix

`bugfix` 让每个 bug 在独立 git worktree 中闭环修复：建分支、修、自检、提交、合回基线、解冲突、清理。多个窗口可以同时跑互不污染。

运行时规则以 `SKILL.md` 为准。`SKILL.md` 故意保持精简（每次触发都会进入上下文），背景与说明放在本 README。

## 触发方式

**仅命令主动触发**。本 skill 不对普通对话做语义匹配，即使消息里出现 "bug"、"worktree"、"隔离修复" 等词也不会被动启用。需要用以下任一方式显式调用：

```text
/bugfix
$bugfix
@bugfix
使用 bugfix skill 修复 ...
```

之所以这样做：worktree 流程对于一次性小改动是过度操作，被动触发会让普通 bugfix 对话付出不必要的隔离成本。如果你只是想让 AI 普通修一个 bug，不要点名本 skill。

> Claude Code 的 skill 仅靠 frontmatter `description` 决定触发，没有 `manual: true` 之类的开关，所以这条约束写在 description 第一句，不能完全移到独立字段。

## 适用场景

- 同时打开多个会话窗口分别修不同的 bug。
- 不希望一份未完成的修复污染另一份的上下文。
- 担心两份修复改到同一段代码、commit 混在一起难拆。
- 想让 AI 在合并冲突简单时自行解决，复杂时主动交还。

## 不适用场景

- 普通的一次性小改动、不涉及隔离的快速 bugfix。
- 只读 review、纯架构讨论。
- 需要跨多个仓库协调的修复（本 skill 只管单仓库 worktree）。

## 关键设计

| 决策 | 取值 |
|---|---|
| 合并目标 | **启动 skill 时所在分支**，不假设 main |
| worktree 位置 | 仓库外 `<repo-parent>/<repo-name>-worktrees/<branch>` |
| 分支前缀 | `bugfix/<slug>-<yyyymmddHHMM>` |
| 合并前自检 | 自动探测 lint / test，**命中即跑、缺失即跳** |
| 合并方式 | `git merge --no-ff`，保留并行修复拓扑 |
| 简单冲突 | AI 自解（≤3 文件且不涉及关键路径） |
| 复杂冲突 | 保留冲突状态交还用户，**不** abort |
| 推送 | **不**自动 push |
| 清理 | 成功删 worktree + 分支；失败 / 冲突保留现场 |

## 关键路径定义（不自动解的冲突）

- Lock 文件：`package-lock.json`、`yarn.lock`、`pnpm-lock.yaml`、`Cargo.lock`、`poetry.lock`、`go.sum`、`Gemfile.lock`
- 数据库 schema / migration 目录
- CI 配置：`.github/workflows/`、`.gitlab-ci.yml`、CircleCI 配置
- 构建配置：`webpack.*`、`vite.*`、`rollup.*`、`tsconfig.json`、`next.config.*`

## 多窗口并发

- 每次合并前都 `git fetch` + `git pull --ff-only`，靠 git 引用更新原子保证。
- `git pull --ff-only` 失败（远端已被另一窗口推过）→ 不强解，报告交还用户。
- skill 内部不做跨窗口锁；用户负责把不同 bug 分配给不同窗口。

## USAGE

Codex 个人 skill：

```text
~/.codex/skills/bugfix/SKILL.md
```

Claude Code 个人 skill：

```text
~/.claude/skills/bugfix/SKILL.md
```

Claude Code 项目级 skill：

```text
.claude/skills/bugfix/SKILL.md
```

Cursor Project Rule（手动触发型，需 `@bugfix` 显式引用）：

```text
.cursor/rules/bugfix.mdc
```

触发示例：

```text
使用 $bugfix 修这个 bug：登录页错误提示文案错乱。
```

```text
/bugfix
我开了三个窗口分别修：登录文案、订单超时、上传按钮 hover 状态。
```

```text
@bugfix 在隔离 worktree 中修复 issue-482，自动跑 lint/test，简单冲突自己解。
```

## 失败现场恢复

如果 skill 在自检失败 / 复杂冲突时停下，会保留 worktree 和分支。手动继续：

```bash
# 进入保留的 worktree 继续修
cd <repo-parent>/<repo-name>-worktrees/bugfix-<slug>-<yyyymmddHHMM>

# 或丢弃这次尝试
git worktree remove --force <repo-parent>/<repo-name>-worktrees/bugfix-<slug>-<yyyymmddHHMM>
git branch -D bugfix/<slug>-<yyyymmddHHMM>
```
