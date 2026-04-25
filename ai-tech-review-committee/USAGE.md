# AI 技术评审委员会使用参考

这个文件给 Skill 使用者参考，说明什么时候适合调用 `ai-tech-review-committee`，以及可以怎么写 prompt。

运行时评审流程以 `SKILL.md` 为准；本文件不要求模型在每次评审时输出。

## Trigger Timing

当你处在“准备开发、准备定方案、准备评审、准备上线或准备重构”的阶段，并希望从多个专业角色获得方案补强时，适合使用本 Skill。

适合触发的场景：

- 你显式想要 `$ai-tech-review-committee`、`AI 技术评审委员会`、`技术评审委员会`、`多角色评审`、`专家评审`。
- 你提出一个软件系统想法、需求草案、PRD、RFC、ADR、MVP 计划，希望判断是否可行、是否值得做、范围是否合理。
- 你给出架构草案、技术选型、接口设计、数据流、AI/LLM 方案、上线方案，希望评审风险和补强细节。
- 你想在开工前先做方案评审，例如“先别写代码”“先评审方案”“先帮我完善需求/方案/架构/计划”。
- 你希望从产品、架构、工程、安全、可靠性、测试、数据/AI、UX、交付成本等不同角度获得专业建议。
- 你准备做高风险改造、重构、迁移、上线、自动化决策、AI agent、数据平台或权限/隐私相关系统。

不适合默认触发的场景：

- 你只是想修一个明确 bug、写一段代码、解释一个 API、跑测试或改一个小 UI。
- 你已经明确要求直接实现，且不需要方案评审、风险分析或多角色建议。
- 你问的是纯知识解释，不涉及具体软件系统方案。
- 你只需要普通代码 review，而不是多角色方案评审。

如果一个实现请求明显缺少需求边界或存在高风险，可以先提示“建议先做一次委员会评审”，但不应强行展开完整评审，除非使用者同意或请求中已经包含评审意图。

## Cross-Tool Usage

这个 Skill 可以被 Claude Code、Codex、Cursor 或其他 AI coding assistant 使用。不同工具的入口方式不同：

- Codex / Claude Code：优先依赖 `SKILL.md` frontmatter `description` 自动匹配，也可以显式写 `$ai-tech-review-committee` 或 `/ai-tech-review-committee`。
- Cursor：可参考 `entrypoints/cursor/ai-tech-review-committee.mdc`，复制到项目的 `.cursor/rules/` 下作为 `Agent Requested` rule。
- 通用 AI 助手：直接在 prompt 里写“按 AI 技术评审委员会流程评审下面的方案”。

## Trigger Prompts

你可以直接使用下面这些 prompt。

```text
使用 $ai-tech-review-committee，帮我评审这个软件系统方案，并从产品、架构、工程、安全、可靠性、测试、AI/data、UX、交付成本角度补强。
```

```text
按 AI 技术评审委员会流程评审下面的方案。先不要写代码，请给出多角色专家意见、关键风险、补强后的方案和下一步行动。
```

```text
请作为技术评审委员会主席，组织产品、架构、工程、安全、SRE、QA、Data/AI、UX、交付成本等角色，对这个方案做 Go / Rework 判断。
```

```text
我准备开发一个系统，先别写代码。请用 AI 技术评审委员会帮我做方案评审，指出风险、缺失需求、架构建议和下一步计划。
```

```text
这是我的 MVP 想法：... 请用多角色专家评审，帮我判断是否值得做、第一版范围怎么裁剪、哪些技术风险要先验证。
```

```text
这是我的架构草案/RFC/ADR：... 请组织技术评审委员会，从架构、工程、安全、SRE、QA 和交付角度给出 Go / Rework 建议。
```

```text
我有一个 AI/LLM 系统方案：... 请用 AI 技术评审委员会评审数据、模型评估、安全、人工审核、成本和上线风险。
```

## Minimal Prompt Template

```text
按 AI 技术评审委员会流程评审下面的方案。

背景：
[项目背景]

目标用户：
[谁使用]

当前方案：
[你的需求、架构、MVP 或技术方案]

已知约束：
[时间、预算、技术栈、团队、合规、性能等]

我希望你输出：
- Go / Go with changes / Rework / Discovery needed
- 多角色专家意见
- 关键风险和缓解建议
- 补强后的方案
- Now / Next / Before Launch 行动计划
```
