# Talpo 路线与验收门槛

## Gate 0：执行链路 Spike

目标：证明当前机器上可以稳定调用一个本地 Worker。

验收：

- Codex CLI 可以通过 Ollama 运行一个极小任务。
- Python 能捕获退出码、stdout、stderr 和 JSONL 事件。
- Worker 能在测试 Git 仓库中创建文件并运行一个测试。
- 设置超时后能停止进程，失败时不伪装成成功。

## Gate 1：单任务 Supervisor

目标：一个 Task 可以完成“执行 → 验收 → 状态落盘”。

验收：

- 生成 `TASKS.json` 和 `STATE.json`。
- Verifier 失败后状态为 `FAIL`，通过后状态为 `PASS`。
- 进程中断后 `talpo resume` 可以继续。

## Gate 2：V0.1 闭环

目标：完成有限重试、专家升级和最终报告。

验收：

- 本地失败最多重试配置次数。
- 升级上下文只包含目标、当前任务、相关 diff、错误和约束。
- 专家建议可交还 Worker 执行。
- 生成包含测试、耗时、Token 和人工介入次数的 `FINAL_REPORT.md`。

## Gate 3：真实 Benchmark

目标：在 Roco Poam 上完成一次可复查运行。

验收：

- 从干净工作树开始运行。
- 记录模型、版本、硬件、配置、命令和完整结果。
- 至少保留一次成功运行和一次失败/恢复记录。
- 不把静态检查或部分完成写成端到端成功。

## 近期明确不做

- Web 控制台和多用户系统
- 多 Provider 市场和插件生态
- 自动部署、自动推送和生产环境操作
- 复杂规划算法或 LLM-as-a-Judge 主验收

