# Talpo V0.1 架构边界

## 目标

Talpo 是 Supervisor，不是新的 Coding Agent Runtime。它管理一次软件工程任务从目标到验收的生命周期，并通过适配器调用外部 Worker。

## 最小数据流

```text
User Goal
  ↓
Planner → TASKS.json
  ↓
Supervisor
  ├─ Worker Adapter → 文件 / Shell / Git
  ├─ Verifier        → 测试 / 构建 / HTTP / 自定义命令
  ├─ Retry Policy    → 有上限的本地重试
  ├─ Escalation      → 最小上下文的专家诊断
  └─ State Store     → STATE.json / JSONL / 报告
```

## V0.1 组件

| 组件 | 职责 | V0.1 形式 |
|---|---|---|
| CLI | 接收目标、启动和恢复任务 | Python CLI |
| Supervisor | 调度任务、推进状态、停止和恢复 | Python 类 |
| Planner | 把目标转成小任务 | 先用结构化模型输出 |
| Worker Adapter | 调用本地或云端 Agent | 首先支持 Codex CLI + Ollama |
| Verifier | 执行确定性验收 | Shell、pytest、HTTP |
| State Store | 保存状态、事件和指标 | JSON、JSONL、Markdown |
| Escalation Provider | 生成诊断和修复建议 | 一个云端 Provider 接口 |

## 关键接口

```python
class Worker:
    def run(self, task: Task, context: RunContext) -> RunResult: ...

class Verifier:
    def verify(self, task: Task, context: RunContext) -> VerifyResult: ...

class EscalationProvider:
    def diagnose(self, context: EscalationContext) -> ExpertAdvice: ...
```

接口先保持窄小。只有出现第二个真实 Provider 或第二种 Verifier 时才抽象出插件机制。

## 状态与产物

被管理项目中使用 `.agent/`：

```text
.agent/
├── GOAL.md
├── PLAN.md
├── ACCEPTANCE.md
├── TASKS.json
├── STATE.json
├── metrics/*.jsonl
└── logs/*.jsonl
```

Talpo 自身的项目决策和跨会话记录放在 `docs/`，不要和被管理项目的运行状态混在一起。

## 完成定义

只有在以下条件全部满足时，任务才能进入 `DONE`：

1. 必需任务完成。
2. 配置的验收 Gate 全部通过。
3. 没有 Blocker 级未解决问题。
4. 生成 `FINAL_REPORT.md`，包含实际命令和结果。

模型回复“已完成”只是一条事件，不能替代 Verifier 结果。

