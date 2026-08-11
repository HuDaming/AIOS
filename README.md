# AIOS

AIOS（AI Operating System）是一个面向 AI 辅助软件工程的能力编排仓库。它把可复用能力、执行角色、交付流程和经验资产分开管理，使 Codex 能够按明确边界完成规划、实现、验证与复盘。

Phase 1 只建立治理骨架、Core Skills 注册表和第一条 feature-development 工作流，不实现 Phase 2 的具体 Agent、领域 Skill、评测器或自动化工具。

## 四层边界

| 层 | 定义 | 负责 | 不负责 |
| --- | --- | --- | --- |
| Skill | 可独立调用、可评估、可复用的单项能力 | 规定如何完成一种工作 | 决定完整项目流程或保存项目经验 |
| Agent | 承担职责并组合 Skills 的执行角色 | 在权限和上下文内作出执行决策 | 取代 Workflow 或把知识写进提示词 |
| Workflow | 编排阶段、门禁、交付物和角色协作的过程 | 定义工作顺序与完成标准 | 实现单项能力或充当知识库 |
| Knowledge | 经验证、可追溯的事实、模式、决策与教训 | 为后续任务提供项目和领域上下文 | 自动执行任务或替代验证 |

依赖方向保持单向：Workflow 编排 Agent，Agent 调用 Skill；三者读取或产出 Knowledge，但 Knowledge 不反向控制执行。

## 整体生命周期

```text
Idea
  -> Discover / Clarify
  -> Specify
  -> Architect
  -> Plan
  -> Implement
  -> Debug / Review
  -> Verify
  -> Capture Knowledge
  -> Done
```

Skill 自身采用独立生命周期：`trial -> evaluate -> install -> distill -> maintain`。任何 Skill 必须先试用和评估，再决定安装；只有需要固化为自有能力时才进入蒸馏，安装后的 Skill 持续维护并定期复评。

## 目录

```text
config/       AIOS 与 Skill 治理策略
skills/       按能力领域分层的 Skill 容器
workflows/    可执行流程定义
agents/       Agent 角色定义
knowledge/    模式、决策、失败、教训与研究
specs/        需求与规格
templates/    可复用模板
registry/     Skill、Agent、Workflow 注册表
evals/        Skill、Agent、Workflow 评测资产
```

## Phase 1 使用方式

1. 从 `registry/` 查找已登记能力和流程。
2. 按 `config/skill-policy.yaml` 判断 Skill 的作用域与生命周期状态。
3. 功能开发遵循 `workflows/feature-development/WORKFLOW.md` 的门禁。
4. 只把已验证、可复用且注明来源的结论写入 `knowledge/`。

## 当前状态

- Phase：1
- Core Skills：10 项已登记
- Workflow：`feature-development` 已定义
- Agent 实现、领域能力与自动化评测：留待后续阶段
