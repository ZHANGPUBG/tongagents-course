# cli-sample

最简 [TongAgents](https://github.com/temp-bigai/Tong-Agent) CLI 样例 (走 SDK, 本地工具集).

仿造 Tong-Agent 项目的 CLI, 但极简化: **单文件 REPL + SDK `StatelessReactAgent` +
6 个本地 OS 工具** (read / write / edit / bash / glob / grep), **同步 `step()` 调用**,
**不调 `.stream()`**, **不依赖 `tongagents-cli`**.
教学目的: 演示怎么用 tongagents SDK 写一个最小可运行 CLI.

> ⚠️ **v0.4.0 重要变更**: **不再依赖 BigAI Nexus**。改用仓库自带的本地 wheel
> (`wheels/tongagents-*.whl` + `wheels/tongagents_cli-*.whl`)，由 `uv sync` 统一安装，
> 无需任何内网凭据（Python 依赖仍需 PyPI/镜像）。

> ⚠️ **依赖变化** (相对 commit 1d5bf25):
>  - 移除 `tongagents-cli` 包依赖
>  - tongagents SDK 已经去掉对 tongagents-cli 的依赖, cli-sample 也跟着简化
>  - SDK 默认 11 个工具的实现直接复制到本地 `cli-sample/tools/` 子包
>  - **保留** 6 个核心 OS 工具 (read_file / write_file / edit_file / bash / glob / grep)
>  - **省略** ExaSearchTool / SkillTool / DelegateTaskTool / QueryBackgroundProcessTool / StopTaskTool (依赖 CLI / 后台进程管理)

---

## 依赖

- Python 3.12（仓库自带 wheel 为 `cp312`）
- [tongagents SDK 2.7.65](../wheels/)（从 `../wheels/tongagents-*.whl` 本地装）
- [tongagents_cli 1.8.65](../wheels/)（从 `../wheels/tongagents_cli-*.whl` 本地装，SDK 间接依赖）
- `python-dotenv` — 读 `.env`

> ❌ **不依赖 `tongagents-cli` 包**（但要装 `tongagents_cli` wheel 给 SDK）
> ✅ **不依赖 BigAI Nexus**（本地 wheel 即可；Python 依赖仍需 PyPI/镜像）

---

## 安装 (uv 方式)

```bash
# 1. clone
git clone https://github.com/temp-bigai/tongagents-course.git
cd tongagents-course

# 2. 解压仓库自带的 wheel
unzip -o wheels/tongagents-wheels.zip -d wheels/
ls wheels/
# 期望看到 tongagents-*.whl + tongagents_cli-*.whl

# 3. 安装 uv（如果还没有）
curl -LsSf https://astral.sh/uv/install.sh | sh

# 4. 创建/同步环境（自动使用 Python 3.12）
cd cli-sample
uv sync
```

`uv sync` 会根据 `pyproject.toml` 中的 `[tool.uv.sources]` 安装本地 wheel，
并创建 `.venv`，使用仓库中的 `uv.lock` 锁定依赖。完整步骤 + 故障排查见 [INSTALL.md](./INSTALL.md)，
平台/依赖说明见 [`../wheels/README.md`](../wheels/README.md)。

> 🍎 **Windows / macOS / ARM64**: 仓库自带 wheel 是 Linux x86_64。
> Windows 请用 WSL2，macOS 请用 Docker（linux/amd64），详见
> [`../docs/environment.md`](../docs/environment.md)。

---

## 用真实 LLM 跑通

```bash
# 1. 复制模板并填写三项
cp .env.example .env
# OPENAI_API_KEY=sk-xxx
# OPENAI_BASE_URL=http://your-llm-endpoint/v1
# OPENAI_MODEL=your-model-name

# 2. 跑
uv run python cli_sample.py
```

> ⚠️ `OPENAI_API_KEY / OPENAI_BASE_URL / OPENAI_MODEL` 三项都必须设置。
> 只填 API Key 时 SDK 会报 `service url is required but not given`。

> ⚠️ **关键陷阱**: cli-sample 用 `load_dotenv(..., override=True)`,
> 因为 shell 里常设了 `OPENAI_API_KEY` / `OPENAI_BASE_URL` (例如指向
> 公开 minimax API), dotenv 默认不覆盖, 会导致 .env 不生效.

REPL 示例 (Tong-Agent 内网 doubao-seed-2-0-pro):

```
============================================================
  cli-sample: TongAgents CLI 极简样例 (走 SDK, 本地工具集)
============================================================
  Agent: StatelessReactAgent (SDK default)
  工具: 6 个本地工具 (read / write / edit / bash / glob / grep)
  模型: doubao-seed-2-0-pro-260215
  端点: http://10.1.53.240
============================================================

>>> 列出当前目录有哪些文件
  当前目录下有以下文件:
  - .env.example
  - README.md
  - cli_sample.py
  - pyproject.toml
  - tools/

>>> 读 README.md 的前 5 行
  README.md 的前 5 行内容如下:
  # cli-sample
  最简 TongAgents CLI 样例 (走 SDK, 本地工具集).
  ...

>>> 写一个 hello.txt 内容是 hello world
  已创建 hello.txt, 内容 "hello world".

>>> 你好
  你好! 有什么可以帮你的吗?

>>> exit
bye!
```

---

## SDK 用法核心

```python
# cli_sample.py 核心 (去掉 .env / imports / 错误处理)
import sys
from pathlib import Path
sys.path.insert(0, str(Path(__file__).resolve().parent))

from tools import register_default_tools  # 本地 tools/ 子包
from tongagents.tools.tool_manager import ToolManager
from tongagents.agents.llm_agent import StatelessReactAgent, ReactAgentSetting
from tongagents.agents.llm import ModelConfig, ModelProvider
from tongagents.agents.llm_agent.defs import LLMInputEvent
from tongagents.agents.llm.messages import UserPromptMessage

# 1. 注册本地 6 工具
register_default_tools()

# 2. 构造 agent (从 env 读 OPENAI_*)
llm_config = ModelConfig(model_provider=ModelProvider.OPENAI_COMPATIBLE)
settings = ReactAgentSetting(
    name="cli-sample-agent",
    llm_config=llm_config,
    tool_identifier_list=list(ToolManager.tool_classes.keys()),
    system_prompt="你是 cli-sample agent, ...",
)
agent = StatelessReactAgent(agent_settings=settings)

# 3. 同步 step — 不调 .stream()
wrap = LLMInputEvent(input=[UserPromptMessage(q)])
actions = agent.step(wrap)  # ← 一次同步返回 list[Action]
for a in actions:
    if getattr(a, "is_final_response", False):
        print(a.content)
```

> ⚠️ **第二个坑**: `LLMInputEvent.input` 必须是 **`list[UserPromptMessage]`**,
> 不能传 `str` 或单个 `UserPromptMessage`. SDK 内部 `plan()` 会做
> `[*observation.input, model_response]` 展开, 单个对象不能 unpack, 会抛
> `TypeError: 'UserPromptMessage' object is not iterable`.

---

## 本地工具集 (cli-sample/tools/)

6 个核心 OS 工具 — 从 tongagents_cli SDK `default_agent/tools.py` 复制简化.

| 工具 | 作用 | 来源 |
|---|---|---|
| `read_file` | 读文件, 支持 offset/limit | SDK ReadFileTool 简化 |
| `write_file` | 写文件, 自动建父目录, 支持 append | SDK WriteFileTool 简化 |
| `edit_file` | 精确字符串替换 (Task #2565 模式) | SDK EditFileTool 简化 |
| `bash` | shell 命令执行 (本地直跑, 无沙箱) | SDK BashTool `_execute_directly` |
| `glob` | 文件名通配符匹配 (`**/*.py`) | SDK GlobTool 简化 |
| `grep` | 内容正则搜索 | SDK GrepTool 简化 |

> **省略的工具** (vs tongagents_cli 11 个):
> - `exa_search` — 依赖外部 Exa API
> - `skill` — 依赖 SDK 内部 hooks
> - `delegate_task` / `query_background_process` / `stop_task` — 依赖 subagent + 后台进程管理

每个工具一个文件, 类名跟 SDK 一致 (方便以后升级 / 替换):

```
cli-sample/tools/
├── __init__.py         # register_default_tools() + 自动注册
├── read_file_tool.py   # ReadFileTool
├── write_file_tool.py  # WriteFileTool
├── edit_file_tool.py   # EditFileTool
├── bash_tool.py        # BashTool
├── glob_tool.py        # GlobTool
└── grep_tool.py        # GrepTool
```

---

## 文件结构

```
cli-sample/
├── cli_sample.py        # 主程序 (REPL + StatelessReactAgent + 本地 tools/)
├── tools/               # 本地工具实现 (从 SDK 复制简化)
│   ├── __init__.py
│   ├── read_file_tool.py
│   ├── write_file_tool.py
│   ├── edit_file_tool.py
│   ├── bash_tool.py
│   ├── glob_tool.py
│   └── grep_tool.py
├── pyproject.toml       # 依赖: python-dotenv；tongagents 从 ../wheels 本地装
├── README.md            # 本文件
└── .env.example         # API key + BASE_URL + MODEL 配置模板
```

---

## 测试

### 1. 静态语法测试

```bash
uv run python -c "import ast; ast.parse(open('cli_sample.py').read()); print('cli_sample.py syntax OK')"
```

### 2. 工具导入 + 注册测试 (不调 LLM)

```bash
uv run python -c "
from tools import register_default_tools
register_default_tools()
from tongagents.tools.tool_manager import ToolManager
print('tools:', sorted(ToolManager.tool_classes.keys()))
"
# 期望: tools: ['bash', 'edit_file', 'glob', 'grep', 'read_file', 'write_file']
```

### 3. REPL 端到端测试 (需 .env)

```bash
cp .env.example .env
# 填写 OPENAI_API_KEY / OPENAI_BASE_URL / OPENAI_MODEL
printf '你好\n列出当前目录\n读 README.md\nexit\n' | uv run python cli_sample.py
```

**预期**: SDK 自动跑 LLM + 自动选工具 + 自动拼回回复, 3 个 query 全过.

---

## 与 TongAgentCLI 的区别

| 维度 | TongAgentCLI (官方) | cli-sample (本样例) |
|---|---|---|
| 代码 | ~几千行 (TUI + skills + session) | ~250 行 + 6 个工具文件 |
| LLM 调用 | SDK StatelessReactAgent / InteractiveMode | SDK StatelessReactAgent — **同款** |
| 工具数 | 11 个 (含 Exa / Skill / 后台进程) | 6 个核心 OS 工具 (本地实现) |
| 依赖 tongagents-cli | ✅ | ❌ (本地 tools/) |
| Session / Memory | ✅ (持久化历史) | ❌ (极简, 每次 step 新建 agent) |
| TUI | ✅ (Textual) | ❌ (纯 stdin/stdout REPL) |
| 学习曲线 | 高 (TUI / session / skills 概念) | 低 (看 `step()` 30 行就懂 SDK 怎么用) |

适合想看 **最小可运行骨架 + 理解 SDK StatelessReactAgent + 本地工具集成** 的同学.
想看生产级 CLI 看 `tongagents-cli`.

---

## 关联仓库

- [Tong-Agent](https://github.com/temp-bigai/Tong-Agent) - 完整 SDK + CLI
- [tongagents-course](https://github.com/temp-bigai/tongagents-course) - 课程仓库 (本仓库)
- [tongagents SDK 2.7.65](https://github.com/temp-bigai/Tong-Agent) - 核心 SDK (Agent 抽象类 + StatelessReactAgent + WorkflowWithMemory)；该包不在公共 PyPI，需使用本仓库 `wheels/` 中的本地 wheel
