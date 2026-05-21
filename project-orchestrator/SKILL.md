---
name: project-orchestrator
description: 模仿 Claude Code 的高级项目管理流程。用于处理复杂项目，包括自动配置 MCP/Skills、任务拆解、循序执行及分步验证。
---

# Project Orchestrator (项目协调官)

本技能旨在将 Gemini CLI 提升为具备“结构化思维”和“自动工具集成”能力的高级工程代理。

## 核心工作流 (Standard Operating Procedure)

当用户发起一个项目或复杂任务时，必须严格遵守以下四个阶段：

### Phase 1: Environment & Tooling (环境与工具配置)
1. **需求解构**：识别项目所需的开发环境（Node.js, Python, Go 等）和外部能力。
2. **MCP/Skill 自动装载**：
   - 如果涉及网页自动化，确保 `puppeteer` 已启用。
   - 如果涉及版本控制，确保 `git` 工具链可用。
   - 如果需要特定领域的技能，使用 `gemini skills list` 检查并建议安装。
3. **工具验证**：运行简单的探测命令确认工具链路畅通。

### Phase 2: Structural Planning (结构化规划)
1. **生成任务清单**：在项目根目录创建或更新 `PLAN.md`。
2. **任务颗粒度**：每个任务必须包含：
   - **ID**: 唯一标识。
   - **Goal**: 明确的目标。
   - **Action**: 拟采用的工具或步骤。
   - **Validation**: 验证成功的具体标准（如测试通过、文件存在等）。
   - **Status**: [ ] 待办, [/] 进行中, [x] 已完成。
3. **用户确认**：展示 `PLAN.md` 概要并等待用户确认（或直接进入静默执行，取决于用户偏好）。

### Phase 3: Sequential Execution (循序执行)
1. **单一焦点**：每次仅执行 `PLAN.md` 中的一个任务。
2. **Plan-Act-Validate 循环**：
   - **Plan**: 针对当前子任务制定具体实施策略。
   - **Act**: 执行代码修改、文件创建等操作。
   - **Validate**: 立即运行相关测试或检查，未通过验证绝不进入下一环节。
3. **状态同步**：每完成一个子任务，更新 `PLAN.md` 的状态，并使用 `update_topic` 报告进度。

### Phase 4: Quality Audit & Handover (审计与交付)
1. **全局审计**：所有子任务完成后，进行全量构建或 Lint 检查。
2. **文档清理**：根据需要移除 `PLAN.md` 或将其转换为 `CHANGELOG.md`。
3. **正式交付**：总结完成情况，告知哥哥项目已就绪。

## 行为准则
- **不跳步**：即使是简单的任务，也要记录在清单中。
- **防御性检查**：在执行每一步前，检查前置依赖是否满足。
- **错误回溯**：如果 Validation 失败，必须回溯 Plan 阶段重新诊断，严禁盲目重试。
