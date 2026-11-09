# CLI Sample with EventSource + Agent (v5)

最小可运行的 **Tong-Agent EventSource + Agent 集成** 教学示例。

跟 `cli-sample/` 互补：

- `cli-sample/`：**Agent + Tools**（LLM 调用本地工具）
- `cli-sample-with-event-soource/`：**EventSource + Agent**（定时触发 → agent 处理 → 工具调用）

---

## 架构（v5）

```text
┌──────────────────────────────────────────────────────────────────────┐
│ stdin reader (后台线程)                                              │
│   input() → runtime.add_user_input(..., source_tag="stdin")          │
└──────────────────────────────────────────────────────────────────────┘
                               ↓
┌──────────────────────────────────────────────────────────────────────┐
│ TimerEventSource (后台线程)                                          │
│   runtime.add_event(event)                                           │
│     → AgentHandler.handle(event, runtime)                            │
│     → runtime.add_user_input(..., source_tag="es_event:...")         │
└──────────────────────────────────────────────────────────────────────┘
                               ↓
                  runtime.input_queue (统一 queue)
                               ↓
┌──────────────────────────────────────────────────────────────────────┐
│ main loop (主线程)                                                   │
│   item = runtime.get_input(timeout=0.5)                              │
│   actions = agent.step(LLMInputEvent(input=[UserPromptMessage(...)]))│
│   print(最终 TextAction.content)                                     │
└──────────────────────────────────────────────────────────────────────┘
```

v5 的关键点：

- **统一输入队列**：stdin 和 ES event 都进入 `runtime.input_queue`，主循环只从这一个来源取。
- **没有独立 dispatcher 线程**：`AgentHandler.handle(event, runtime)` 自己把 event 包装成
  user input 并推入 queue。
- **复用 cli-sample**：`AgentHandler` 注册 `cli-sample/tools/` 的 6 个工具，并构造同样的
  `StatelessReactAgent`。

---

## 文件结构

```text
cli-sample-with-event-soource/
├── README.md
├── pyproject.toml                     # uv 环境与本地 wheel 依赖
├── requirements.txt                   # 兼容旧流程的额外依赖说明
├── test_run.py                        # 只验证 queue + event 注入，不调用 LLM
└── src/
    ├── main.py                        # CLI 入口
    ├── event_source/
    │   ├── __init__.py
    │   ├── runtime.py                 # EventSourceRuntime（统一 input_queue）
    │   └── timer_source.py            # TimerEventSource
    └── handlers/
        ├── __init__.py                # 只导出 AgentHandler
        └── agent_handler.py           # 注册工具 + 构造 agent + 推 event
```

> 本示例复用 `../cli-sample/tools/`，因此需要在完整课程仓库中运行，不能只复制本目录。

---

## 安装

本示例有自己的 `pyproject.toml`，使用 uv 管理环境：

```bash
# 在仓库根目录
unzip -o wheels/tongagents-wheels.zip -d wheels/

cd cli-sample-with-event-soource
uv sync
```

`uv sync` 会根据 `[tool.uv.sources]` 安装本地 tongagents，并创建 `.venv`，使用仓库中的 `uv.lock` 锁定依赖。
本示例复用 `../cli-sample/tools/`，因此需要在完整课程仓库内运行。

> ⚠️ `tongagents` 不在公共 PyPI，不能 `pip install tongagents`。必须使用仓库
> `wheels/` 中的本地 wheel。平台/依赖说明见
> [`../wheels/README.md`](../wheels/README.md) 和
> [`../docs/environment.md`](../docs/environment.md)。

---

## 运行

### 配置 LLM

本示例不会自动读取 `.env`，需要在 shell 中导出三项环境变量：

```bash
export OPENAI_API_KEY="sk-..."
export OPENAI_BASE_URL="https://api.openai.com/v1"
export OPENAI_MODEL="gpt-4o-mini"
```

> 三项都必须设置；只设置 `OPENAI_API_KEY` 会报
> `ValueError: service url is required but not given`。

### 启动

```bash
cd cli-sample-with-event-soource
uv run python src/main.py
```

预期：

- 后台启动 `TimerEventSource`（默认每 30 秒触发，最多 5 次）
- 定时事件被包装成 user input，走与手工输入相同的 `agent.step()` 路径
- 输入 `exit` / `quit` / `q` 退出

### 快速测试（短间隔）

```bash
cd cli-sample-with-event-soource
EVENT_INTERVAL_SECONDS=1 EVENT_MAX_COUNT=2 uv run python src/main.py
```

### 集成测试

`test_run.py` 只验证 queue / event 注入逻辑，**不会调用 LLM**：

```bash
cd cli-sample-with-event-soource
uv run python test_run.py
```

它仍然会构造 `AgentHandler`，所以需要先执行 `uv sync`，并设置一个可用的
`OPENAI_BASE_URL`（可以是假地址，因为测试不会真正发请求）。

---

## 环境变量

| 变量 | 默认值 | 说明 |
|---|---|---|
| `OPENAI_API_KEY` | (无) | LLM key |
| `OPENAI_BASE_URL` | (无) | OpenAI 兼容端点 URL |
| `OPENAI_MODEL` | (无) | 模型名 |
| `EVENT_INTERVAL_SECONDS` | `30` | 定时间隔（秒） |
| `EVENT_MAX_COUNT` | `5` | 最大触发次数 |

---

## v4 → v5 关键变化

| 维度 | v4 | v5 |
|---|---|---|
| Event 注入 | `main.py` 里的独立 `es_dispatcher` 线程 | `AgentHandler.handle(event, runtime)` 直接推 queue |
| stdin | main loop 直接 `input()` | `stdin_reader` 后台线程 → `runtime.add_user_input()` |
| 输入来源 | user 和 event 分开处理 | 统一 `runtime.input_queue` |
| Runtime 配置 | `auto_dispatch=False` | 无 `auto_dispatch`，runtime 持有统一 queue |
| 代码入口 | `run_cli_loop()` + dispatcher | `run_cli_loop()` 单源消费 `runtime.get_input()` |

---

## 与 cli-sample 的对应关系

| 本示例 | cli-sample/cli_sample.py |
|---|---|
| `AgentHandler.agent` | `CliSampleAgent`（内部 `StatelessReactAgent`） |
| `run_cli_loop()` | `main()` REPL |
| `event_to_user_input()` | —（没有 ES） |
| `TimerEventSource` | — |
| 6 个工具 | `tools/` 同名工具 |

---

## 设计动机

- **为什么复用 cli-sample**：REPL、工具注册、Agent 构造都已在 cli-sample 中验证过，
  本示例只新增 EventSource 机制，避免重复造轮子。
- **为什么统一 input_queue**：让用户输入和定时事件走完全相同的处理路径，主循环不需要
  区分来源，也不需要额外的 dispatcher 线程。
- **为什么 handler 负责推 queue**：`AgentHandler` 最清楚 event 应该如何转换成 user input；
  把转换逻辑放在 handler 里，main loop 可以保持和 cli-sample 一样简单。
