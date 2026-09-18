# ADR-0001：Talpo 作为 Supervisor，不重写 Agent Runtime

- 状态：ACCEPTED
- 日期：2026-09-18

## 背景

Talpo 需要长时间运行任务、保存状态、有限重试、云端升级和确定性验收。文件编辑、Shell 执行、工具调用和 Agent Loop 已经由成熟的 Coding Agent CLI 提供。

## 决策

V0.1 只实现 Supervisor 层：目标、任务、Worker 适配器、Verifier、状态、重试、升级和报告。第一种 Worker 适配器优先使用 Codex CLI + Ollama；核心接口不得依赖某个模型的内部提示词或会话格式。

## 结果

- V0.1 工作量和故障面较小，可以先证明闭环。
- Codex CLI 或 Ollama 的行为变化必须被隔离在适配器内。
- 后续可以增加其他 Worker，而不重写 Supervisor。
- Talpo 的核心价值必须体现在调度、恢复、验收和成本/覆盖率指标，而不是重复实现编辑器或工具调用。

## 放弃条件

如果 Gate 0 证明目标 CLI 无法稳定提供退出码、事件或恢复能力，则暂停直接依赖该 CLI，保留相同的 Worker 接口，改为实现一个更薄的本地模型适配器。

