# 智能体 Token 优化与记忆管理准则 (TencentDB & DeepSeek 融合版)

本准则融合了 **DeepSeek-Reasonix** 的缓存优化策略与 **TencentDB Agent Memory** 的结构化卸载技术，旨在长对话场景下降低 Token 浪费 (最高可达 61%) 并提升任务成功率。

---

## 1. 核心原则：前缀一致性 (Prefix Stability)
**适用场景**：DeepSeek 系列模型 (支持 Prefix Cache)
- **规则 1.1**：保持 System Prompt 绝对静态。严禁将动态变量（如 `{{current_time}}`、`{{session_id}}`）放置在 Prompt 开头。
- **规则 1.2**：标准指令集。使用固定格式的 Markdown 块作为工具调用和规划模版，确保模型生成的结构在多次 Turn 中保持高度一致。

## 2. 核心技术：上下文卸载 (Context Offloading)
**适用场景**：处理大文件读取、网络搜索结果、冗长日志。
- **规则 2.1：卸载入库**。将超过 500 Token 的工具原始输出（Raw Output）直接写入本地存储 (SQLite/Markdown)，在上下文对话中仅保留：
  - **Summary**：3 行以内的核心结论。
  - **Index**：文件路径或 `node_id` (例如 `[Ref: storage/logs/run_123.log]`)。
- **规则 2.2：按需追溯**。Agent 仅在需要验证特定细节时，才通过 `grep` 或 `read_file` 访问外部存储，严禁全量回填。

## 3. 结构化记忆：符号任务画布 (Symbolic Canvas)
**适用场景**：复杂、多步骤的长任务。
- **规则 3.1：Mermaid 状态追踪**。在对话的关键节点，要求 Agent 更新一个 Mermaid 格式的状态图。
  - **优点**：图形化语言信息密度极高，比自然语言复述节省 80% 以上的进度说明 Token。
- **规则 3.2：增量更新**。仅在状态发生实质性迁移时更新画布，避免每轮对话都输出冗长的进度报告。

## 4. 上下文压缩与蒸馏 (Context Distillation)
**适用场景**：对话轮次超过 20 轮或 Token 接近窗口限制。
- **规则 4.1：滚动摘要**。实施“软限制” (Soft Limit) 提醒。当上下文达到 70% 时，触发 `/compact` 操作。
- **规则 4.2：事实提取**。将“叙述性历史”转化为“原子化事实清单”。保留已验证的假设、已修复的 Bug 和当前阻塞点，剔除中间的尝试过程。

## 5. 架构级优化 (Architectural Optimization)
- **规则 5.1：分级推理 (Tiered Reasoning)**。
  - **L1 (敏捷层)**：简单修改、状态查询 -> 使用 `deepseek-flash`。
  - **L2 (规划层)**：架构设计、复杂 Debug -> 使用 `deepseek-reasoner`。
- **规则 5.2：隔离缓存空间**。Planner 和 Executor 使用独立的 Session，防止 Planner 的发散性思考破坏 Executor 的代码缓存前缀。

---

## 实施建议 (Implementation)
1. **本地存储路径**：`./.gemini/memory/`
2. **状态记录文件**：`./.gemini/memory/CANVAS.md` (记录 Mermaid 图)
3. **事实清单文件**：`./.gemini/memory/FACTS.md`
