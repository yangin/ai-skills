# AI 技术评审委员会

`ai-tech-review-committee` 用多角色专家委员会评审并补强软件系统方案。运行时规则以 `SKILL.md` 为准。

`SKILL.md` 保持精简；触发示例和跨工具入口放在本 README，避免增加运行时 token。

## USAGE

Codex 个人 skill：

```text
~/.codex/skills/ai-tech-review-committee/SKILL.md
```

Claude Code 个人 skill：

```text
~/.claude/skills/ai-tech-review-committee/SKILL.md
```

Claude Code 项目级 skill：

```text
.claude/skills/ai-tech-review-committee/SKILL.md
```

Cursor Project Rule：

```text
.cursor/rules/ai-tech-review-committee.mdc
```

触发示例：

```text
使用 $ai-tech-review-committee，帮我评审这个软件系统方案，并从产品、架构、工程、安全、可靠性、测试、AI/data、UX、交付成本角度补强。
```

```text
按 AI 技术评审委员会流程评审下面的方案。先不要写代码，请给出多角色专家意见、关键风险、补强后的方案和下一步行动。
```

```text
请作为技术评审委员会主席，组织产品、架构、工程、安全、SRE、QA、Data/AI、UX、交付成本等角色，对这个方案做 Go / Rework 判断。
```
