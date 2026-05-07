---
name: bugfix
description: 仅在用户显式 `/bugfix`、`$bugfix`、`@bugfix` 或明确点名 "bugfix" skill 时启用；不要根据 bug / worktree 等关键词被动触发。每个 bug 走独立 git worktree：建分支 → 修复 → 自检 → 提交 → 合回基线 → 必要时解冲突 → 清理。
---

# Bugfix

## Guardrails

- 工作树脏（`git status --porcelain` 非空）→ 提示用户先处理，停止。
- 全程不 push / 不 force / 不 reset --hard / 不 amend / 不 rebase / 不 `--no-verify`。
- 任何 git 命令非预期失败 → 立即停止并向用户报告。
- 多窗口并发：靠 git 引用更新原子保证；`git pull --ff-only` 失败不强解，报告给用户。

## Phase 0 · 捕获基线

1. `git rev-parse --is-inside-work-tree` 校验在仓库内。
2. `git status --porcelain` 校验干净。
3. `git rev-parse --abbrev-ref HEAD` → `BASE_BRANCH`（**以启动时所在分支为准，不假设 main**）。
4. `git rev-parse HEAD` → `BASE_SHA`。
5. 由 bug 描述生成 `slug`：小写、短中划线、≤40 字符、去标点。

## Phase 1 · 建 worktree

6. 分支名：`bugfix/<slug>-<yyyymmddHHMM>`。
7. worktree 路径：`<repo-parent>/<repo-name>-worktrees/<branch-name>`（仓库**外**，避免被 glob / 索引污染）。
8. `git fetch`（失败不阻断）。
9. `git worktree add -b <branch> <path> <BASE_BRANCH>`。
10. 进入该路径执行后续步骤。

## Phase 2 · 修复

11. 严格围绕本次 bug 描述，不顺手做无关重构 / 清理。

## Phase 3 · 合并前自检

按下表探测，**命中即跑、缺失即跳**。任一失败 → 停在 worktree，输出失败摘要，**不**清理：

| 类型 | 探测条件 | 命令 |
|---|---|---|
| Node | `package.json` 含 `scripts.lint` / `scripts.test` | `npm run lint` / `npm test`（pnpm/yarn 同义替换） |
| Python | `pyproject.toml` / `tox.ini` / `Makefile` 含 lint/test target | 对应命令 |
| Go | 存在 `go.mod` | `go vet ./...`、`go test ./...` |
| Rust | 存在 `Cargo.toml` | `cargo clippy -- -D warnings`、`cargo test` |
| Makefile fallback | `Makefile` 含 `lint` / `test` target | `make lint`、`make test` |

12. 多个匹配并存 → 按项目主语言优先，最多一组 lint + 一组 test。

## Phase 4 · 提交

13. `git add -A` + `git commit -m "fix: <bug 摘要>"`：英文、单行、祈使、≤30 词、Conventional Commits。

## Phase 5 · 合回 BASE_BRANCH

14. 退回主仓库工作树。
15. `git fetch`（失败不阻断）。
16. 若 `BASE_BRANCH` 有 upstream → `git pull --ff-only`；非 ff 则停止报告。
17. `git checkout <BASE_BRANCH>` → `git merge --no-ff <bugfix-branch> -m "merge: bugfix/<slug>"`。

## Phase 6 · 冲突处理

仅在 merge 报冲突时进入，**不要** `merge --abort`。

判定为**复杂冲突**（任一命中）→ 停手交还用户，列出冲突文件 + 简要原因，保留状态：

- 冲突文件 > 3 个
- 命中 lock 文件：`package-lock.json` / `yarn.lock` / `pnpm-lock.yaml` / `Cargo.lock` / `poetry.lock` / `go.sum` / `Gemfile.lock`
- 命中数据库 schema / migration
- 命中 CI 配置：`.github/workflows/` / `.gitlab-ci.yml` / circleci
- 命中构建配置：`webpack.*` / `vite.*` / `rollup.*` / `tsconfig.json` / `next.config.*`

简单冲突 → 逐文件读冲突区块，结合 bug 描述 + 双方 commit message 决定保留逻辑；解完 `git add <file>`，全部解决后 `git commit`。

## Phase 7 · 清理

- 合并成功 → `git worktree remove <path>` + `git branch -d <branch>`。
- 自检失败 / 中止 / 复杂冲突 → **保留** worktree 与分支，向用户明确：worktree 路径、分支名、失败阶段与原因、后续手动命令。
