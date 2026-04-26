# Git Commit

`git-commit` 是一个自动生成并创建 Conventional Commits 的 skill。运行时规则以 `SKILL.md` 为准。

`SKILL.md` 故意保持精简，因为它会在每次触发 skill 时进入上下文；详细安装说明放在本 README，避免增加运行时 token。

## USAGE

Codex 个人 skill：

```text
~/.codex/skills/git-commit/SKILL.md
```

Claude Code 个人 skill：

```text
~/.claude/skills/git-commit/SKILL.md
```

Claude Code 项目级 skill：

```text
.claude/skills/git-commit/SKILL.md
```

Cursor Project Rule：

```text
.cursor/rules/git-commit.mdc
```

触发示例：

```text
使用 $git-commit，帮我提交当前变更。
```

```text
/git-commit
```

```text
请分析当前 diff，如果内容过多就拆分成多个 commit，message 用英文 Conventional Commits。
```

```text
Generate concise commit messages and commit these changes with feat/fix/docs prefixes.
```
