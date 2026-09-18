# Talpo 开发规范

## 项目定位

Talpo 是一个 Local-first、支持确定性验收和云端专家升级的软件工程 Supervisor。
它负责目标、任务、调度、状态、重试、升级和验收；具体的文件修改、命令执行和 Agent Loop 优先交给成熟的执行底座。

V0.1 的目标是证明一条最小闭环真实可用：

```text
Goal → Task → Local Worker → Verifier → Retry → Escalation → Resume → Report
```

## 开发原则

1. 先验证闭环，再扩展功能；没有可运行证据的设计不算完成。
2. 不在 V0.1 重写 Coding Agent Runtime，不引入 LangChain、LangGraph、CrewAI、向量数据库或 Web UI。
3. 核心状态必须落盘，不能把对话上下文当作数据库。
4. 模型自评不能作为完成条件；完成必须由测试、构建、健康检查或其他确定性 Gate 证明。
5. 本地模型失败后有限重试，达到阈值才升级；禁止无限重复相同尝试。
6. 默认在独立 Git worktree 或明确的项目目录内工作，禁止未经授权修改项目外文件、推送远程或执行生产操作。
7. 优先支持 Windows + PowerShell，同时保持 Python 代码和数据格式可跨平台。
8. 每项重要决策、实验结果、失败原因和未验证假设都必须写入工作记录或决策记录。

## V0.1 边界

必须完成：

- `talpo run` 和 `talpo resume` 的最小 CLI
- 项目初始化和 `.agent/` 状态文件
- 一个本地 Worker 适配器（优先 Codex CLI + Ollama）
- Shell/test/HTTP 等确定性 Verifier
- 有上限的 Retry、失败上下文保存和一次 Cloud Escalation 接口
- JSONL 运行记录和 `FINAL_REPORT.md`
- Roco Poam 的一次真实端到端实验

暂不做：

- 多 Agent 群聊、复杂 DAG、浏览器自动化、分布式 Worker
- 自动生产部署、自动 `git push`、复杂权限中心
- 为了“平台化”而提前引入大型框架

## 工作方式

每次会话开始时：

1. 阅读本文件、`docs/ARCHITECTURE.md`、`docs/ROADMAP.md` 和最近一条工作记录。
2. 检查工作区状态，确认当前任务和未解决问题。
3. 先写清本次会话的目标与验收条件，再修改代码。

每次会话结束时：

1. 运行与改动匹配的最小验证。
2. 记录实际完成内容、命令和结果、失败或未验证事项。
3. 更新状态文件或工作记录；不要把计划写成已完成。

## 记录约定

- 架构取舍写入 `docs/decisions/`，使用 `ADR-NNNN-*.md`。
- 每次工作写入 `docs/worklog/YYYYMMDD-<topic>.md`。
- 实验结果必须包含环境、输入、命令、结果和限制。
- 状态只允许使用：`PLANNED`、`RUNNING`、`PASS`、`FAIL`、`BLOCKED`、`DONE`。

