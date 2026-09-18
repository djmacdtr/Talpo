# Talpo 完整实施大纲

## 0. 产品定义

Talpo 是 Local-first 的软件工程 Supervisor。用户给出一个可验收的工程目标，Talpo 将目标拆成任务，调用本地 Worker 执行，运行确定性 Verifier，有限重试，必要时将最小上下文升级给云端专家，最后生成可复查的结果报告。

第一阶段不追求“可以开发任意软件”，只证明以下承诺：

```text
目标 → 小任务 → 本地执行 → 确定性验收 → 有限恢复 → 专家升级 → 最终报告
```

## 1. 成功标准

### 1.1 V0.1 产品成功

- 用户可以在一个 Git 仓库中运行 `talpo init`、`talpo run`、`talpo resume`。
- 每个任务都有状态、尝试次数、输入上下文、验证结果和日志。
- Worker 失败不会被报告为成功；进程超时、崩溃和取消都能落盘。
- Verifier 通过后才允许任务进入 `PASS` 或项目进入 `DONE`。
- 失败达到阈值后可以生成最小升级包，并把专家建议交还 Worker。
- 生成包含变更、测试、耗时、Token、升级次数和人工介入次数的 `FINAL_REPORT.md`。
- 在 Roco Poam 或等价小型项目上保留一次从干净工作树开始的真实运行记录。

### 1.2 不作为 V0.1 成功标准

- 任意项目都能无人值守完成。
- 本地模型在所有任务上达到云端模型质量。
- 自动部署生产环境或自动推送远程。
- 多 Agent 协作、复杂 DAG、Web 控制台和插件市场。

## 2. 分阶段里程碑

### M0：环境与协议 Spike

目标：确认 Worker 的实际调用方式，而不是仅凭文档假设。

工作项：

1. 记录 Codex CLI、Ollama、Python、Git 和模型版本。
2. 调查 Codex OSS Provider 对 Ollama 的模型名、`/v1/models` 响应和 metadata 要求。
3. 通过一个极小 Git fixture 验证：创建文件、运行测试、捕获 JSONL、返回退出码。
4. 增加硬超时和进程树终止探针。
5. 若 Codex + Ollama 不稳定，实现 `OllamaWorker` 直接适配器，保留相同的 `Worker` 接口。

退出条件：成功运行一次，或明确记录兼容性阻塞与替代方案。

### M1：可恢复的单任务执行器

目标：一个任务能被执行、验收、失败落盘和恢复。

建议模块：

```text
src/talpo/
├── cli.py
├── models.py
├── config.py
├── supervisor.py
├── state_store.py
├── worker.py
├── verifier.py
├── process.py
├── events.py
├── retry.py
├── report.py
└── paths.py
```

工作项：

- Pydantic 数据模型和状态枚举。
- `RunContext`、`TaskContext` 和 `AttemptContext`。
- JSON 原子写入，JSONL 事件追加写入。
- Worker 的 stdout、stderr、退出码、开始/结束时间和取消原因。
- Shell、pytest、HTTP health 三种 Verifier。
- 单任务 `run` 和中断后 `resume`。

退出条件：故意让测试第一次失败、第二次修复，状态和报告与实际结果一致。

### M2：任务规划与有限恢复

目标：从 Goal 生成简单任务列表，并按依赖执行。

工作项：

- `GOAL.md`、`PLAN.md`、`ACCEPTANCE.md` 初始化。
- `TASKS.json` 的最小字段：id、title、status、depends_on、attempts、last_error。
- 只支持线性和简单依赖，不实现通用 DAG 调度器。
- 默认 `max_local_attempts=3`。
- 对重复错误进行摘要，避免把相同 prompt 无限发送给本地模型。
- 每次任务完成后 checkpoint。

退出条件：至少三个有依赖关系的任务可恢复执行，重启 Supervisor 不丢状态。

### M3：云端专家升级

目标：只在明确条件下升级，并能继续执行。

升级触发：

- 本地连续失败达到阈值。
- 任务被标记为高难。
- Milestone Review 明确要求。

升级上下文：

```json
{
  "goal": "...",
  "task": "...",
  "constraints": [],
  "relevant_files": [],
  "git_diff": "...",
  "verification_errors": [],
  "local_attempt_summary": []
}
```

工作项：

- `EscalationProvider` 接口。
- 一个云端 Provider 实现。
- 凭据只从受控配置读取，不写入日志、报告或 Git。
- 专家输出保存为建议，不直接当作完成证明。
- 把建议交还 Worker 执行，再由 Verifier 判定。

退出条件：构造一个本地必失败任务，确认升级包最小、建议可追踪、修复后仍需 Gate 通过。

### M4：指标、报告与真实 Benchmark

目标：让每次运行可比较、可复查、可公开展示。

必须记录：

- Worker/provider/model/version。
- 任务数、尝试数、通过数、失败数、阻塞数。
- Local Coverage Rate。
- Cloud Escalation 次数和 Cloud Token Ratio。
- 总耗时、任务耗时和人工介入次数。
- 每个 Gate 的命令、退出码和摘要。

Benchmark 初期固定：

- 一个纯 Python 小项目。
- 一个带 SQLite 和 HTTP API 的项目。
- Roco Poam MVP。

每个 Benchmark 必须记录硬件、操作系统、模型、上下文、提示版本和验收命令。

### M5：Public Alpha

目标：让陌生开发者在 10 分钟内跑通一个示例。

工作项：

- `pyproject.toml`、可安装 CLI、版本号和 CHANGELOG。
- Windows PowerShell、macOS/Linux 启动说明。
- 示例项目和失败恢复示例。
- 安全模式、dry-run、worktree、取消和日志路径说明。
- GitHub Actions：单元测试、类型检查、文档检查。
- Issue 模板、贡献指南、行为准则和安全报告流程。

## 3. 运行时架构

```text
CLI
 │
 ▼
Supervisor ───── StateStore ───── JSON / JSONL / Markdown
 │
 ├── Planner
 ├── WorkerAdapter ── Codex/Ollama（第一适配器）
 │                  └─ Direct Ollama（兼容性备用）
 ├── Verifier ───── shell / pytest / HTTP
 ├── RetryPolicy
 ├── EscalationProvider
 └── ReportBuilder
```

Supervisor 不直接解析某个模型的自然语言来判断成功。模型输出只能产生建议、事件或修改，最终状态由 Verifier 和策略共同决定。

## 4. 核心数据模型

### ProjectState

```json
{
  "schema_version": 1,
  "status": "RUNNING",
  "current_task_id": "T001",
  "completed_tasks": [],
  "blocked_tasks": [],
  "local_attempts": 0,
  "cloud_escalations": 0,
  "human_interventions": 0,
  "last_event_id": "..."
}
```

### Task

```json
{
  "id": "T001",
  "title": "...",
  "status": "PLANNED",
  "depends_on": [],
  "attempts": 0,
  "verification": null,
  "last_error": null
}
```

### RunEvent

每一条事件至少包含：event_id、run_id、timestamp、type、task_id、provider、payload。事件必须可追加，不覆盖历史事实。

## 5. 安全与恢复

- 默认只允许在目标项目目录或 worktree 内写入。
- 默认禁止 `git push`、生产部署、修改系统配置和删除外部数据。
- 每次 Worker 调用必须有超时、取消和最大输出限制。
- 子进程终止必须处理进程树，避免 Ollama/Codex 子进程泄漏。
- 凭据不得进入 prompt 快照、stdout、stderr、JSONL 或 FINAL_REPORT。
- 失败进入 `FAIL` 或 `BLOCKED`，不能为了完成率放宽验收。
- 在恢复前检查 Git 状态和上一次运行的最后事件，避免重复执行已通过任务。

## 6. 测试策略

### 单元测试

- 状态迁移合法性。
- 原子状态写入和损坏文件处理。
- Retry 计数、阈值和重复错误摘要。
- Verifier 退出码、超时和输出截断。
- 最小升级上下文生成。

### 集成测试

- Fake Worker + Fake Verifier 的完整 Supervisor 循环。
- Codex/Ollama 适配器的事件解析。
- Windows PowerShell 命令和路径。
- 进程超时、取消、重启和 resume。

### 真实验收

- Gate 0 fixture。
- Roco Poam MVP。
- 干净工作树启动和最终报告复查。

## 7. 版本路线

```text
v0.0.1  文档、治理、Gate 0 协议
v0.1.0  单任务、Verifier、Retry、Resume、Report
v0.2.0  Planner、Escalation、Metrics、Roco Benchmark
v0.3.0  多 Worker Provider、安全模式、可安装 CLI
v1.0.0  Public Alpha 验证后的稳定接口
```

## 8. 每次开发的固定流程

1. 阅读 `AGENTS.md`、本文件、`ROADMAP.md` 和最近工作记录。
2. 写清本次任务的输入、输出和验收命令。
3. 先写失败测试或最小 fixture，再实现功能。
4. 执行最小验证，保留实际输出和限制。
5. 更新工作记录和必要的 ADR。
6. 提交一个边界清晰的 Git commit。

