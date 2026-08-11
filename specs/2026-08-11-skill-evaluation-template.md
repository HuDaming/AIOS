# Skill Evaluation Template Design

## Status

- Status: Approved design
- Date: 2026-08-11
- Owner: Apple情绪化
- Phase: 2

## Goal

为 AIOS 增加一套供人工和 Codex 直接填写的 Markdown Skill 评估记录模板，用于统一记录试用证据、评分维度、采用结论和复评日期。

## Scope

### Included

- 在 `evals/skills/` 提供单一 Markdown 评估模板。
- 覆盖候选 Skill 的首次评估、继续评估和已采用 Skill 的复评。
- 引用 `config/skill-policy.yaml` 中的评分维度、权重、门禁和生命周期。
- 规定实际评估记录的命名和条件必填规则。
- 提供可由静态检查验证的固定章节和枚举值。

### Excluded

- 不创建自动评分程序、CLI、GitHub Actions 或其他评测自动化。
- 不创建或修改 Agent、Workflow、Skill 实现和 Skill Registry。
- 不增加 YAML 数据模板或 Markdown Front Matter。
- 不改变 `config/skill-policy.yaml` 已定义的生命周期、评分阈值或权重。

## File Design

新增文件：

```text
evals/skills/skill-evaluation-template.md
```

使用该模板创建的评估记录按以下格式命名：

```text
YYYY-MM-DD-<skill-id>-evaluation.md
```

`<skill-id>` 必须与 Skill Registry 使用相同的小写 `kebab-case` 标识。模板文件本身保留 `skill-evaluation-template.md` 名称，不带日期。

## Template Structure

### 1. Evaluation Metadata

记录以下字段：

- Skill ID
- Skill name
- Skill version
- Source
- Scope：`global` 或 `project`
- Current lifecycle stage：`trial`、`evaluate`、`install`、`distill` 或 `maintain`
- Evaluator
- Evaluation date
- Related task or requirement

### 2. Trial Evidence

使用表格记录一个或多个代表性试用案例。每项包含：

- Case ID
- Goal
- Input or context
- Expected result
- Actual result
- Verification method
- Evidence reference
- Result：`pass` 或 `fail`

评估不得只有主观结论。每个用于支持评分或决策的试用案例都必须提供可复核的验证方式和证据引用。证据可以是仓库相对路径、测试命令及摘要、任务记录或稳定 URL，但不得包含密码、Token、私钥或敏感数据。

### 3. Scorecard

评分表固定包含 `config/skill-policy.yaml` 当前定义的七个维度：

1. `relevance`
2. `reliability`
3. `verifiability`
4. `interoperability`
5. `maintainability`
6. `safety`
7. `efficiency`

每个维度记录：

- Score：1–5 的整数
- Rationale
- Evidence reference

模板展示当前权重以便人工计算，但必须明确声明：`config/skill-policy.yaml` 是维度、权重和阈值的唯一事实来源；若模板与策略文件不一致，以策略文件为准。

加权总分计算方式：

```text
weighted total = sum(dimension score * dimension weight)
```

模板记录计算后的加权总分。当前采用阈值为 4；无论加权总分多少，`safety` 低于 4 都会阻塞安装。

### 4. Gate Checks

使用复选框确认：

- 已完成至少一个有代表性的试用案例。
- 关键评分均有证据支持。
- 已检查权限、隐私、数据和不可逆操作风险。
- 已检查与 Codex、AIOS 仓库规则及相关 Skills 的兼容性。
- Source、version、scope、owner 和 activation 已明确，或已记录缺失项。
- 安全分数满足安装门禁，或已明确标记为阻塞。

未通过的门禁必须在决策依据或后续动作中处理，不能静默忽略。

### 5. Decision

`Decision` 必须且只能使用以下一个值：

- `install`
- `distill`
- `maintain`
- `continue-evaluation`
- `do-not-adopt`

同时记录：

- Decision rationale
- Known limitations
- Required follow-up actions

`continue-evaluation` 和 `do-not-adopt` 是评估结论，不是 Skill 生命周期阶段。`do-not-adopt` 必须填写拒绝原因。

### 6. Review Schedule

日期使用 `YYYY-MM-DD` 格式，规则如下：

- `install`、`distill`、`maintain`：必须填写 `Review date`。
- `continue-evaluation`：必须填写 `Next evaluation date`。
- `do-not-adopt`：复评日期可留空。

模板同时提供 `Review trigger`，用于记录即使尚未到日期也应提前复评的条件，例如上游版本变化、权限模型变化、连续失败或维护状态变化。

## Data Flow

```text
Skill candidate or installed Skill
  -> representative trial evidence
  -> seven-dimension scorecard
  -> policy gate checks
  -> fixed decision
  -> review schedule or rejection reason
  -> completed evaluation record
```

评估记录是证据文档，不自动修改 `registry/skills.yaml`。若结论要求生命周期或注册信息变化，必须在单独任务中审查并修改注册表。

## Error and Boundary Handling

- 缺少证据的评分必须标记为未完成，不得用默认分数代替。
- 评分不是 1–5 整数时，评估记录不得视为完成。
- 使用未定义结论值时，评估记录不得视为完成。
- `safety < 4` 且结论为 `install` 时，评估无效。
- 条件必填日期或拒绝原因缺失时，评估记录不得视为完成。
- 模板不得保存敏感信息；证据应引用受控位置。

## Verification

实施后执行静态验证，确认：

1. 文件位于 `evals/skills/skill-evaluation-template.md`。
2. Markdown 包含六个必需章节。
3. Scorecard 包含七个策略维度，且不存在额外评分维度。
4. 模板列出五个允许的 Decision 值。
5. 模板明确 `skill-policy.yaml` 为权重和阈值的唯一事实来源。
6. 模板明确安全阻塞规则、条件日期规则和拒绝原因规则。
7. 实例命名规则为 `YYYY-MM-DD-<skill-id>-evaluation.md`。
8. `git diff --check` 不报告格式错误。

## Acceptance Criteria

- 使用者无需阅读模板实现说明即可完成一份可复核的 Skill 评估记录。
- 试用证据、七维评分、门禁、结论和复评计划之间可相互追溯。
- 评估结论和生命周期阶段不会混淆。
- 模板不会成为评分策略的第二事实来源。
- 本次改动不扩展到评测自动化、Agent、Workflow 或 Skill 实现。
