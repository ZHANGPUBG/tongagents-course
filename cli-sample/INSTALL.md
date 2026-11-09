# cli-sample 安装 — 本地 wheel 方式 (无 Nexus 权限)

> **v0.4.0 重要变更**: 本示例 **不再依赖 BigAI Nexus 私有源**。仓库自带
> `wheels/tongagents-wheels.zip`，解压后执行 `uv sync` 即可。
>
> ⚠️ “本地 wheel”只表示不需要 Nexus；`pydantic / litellm / fastapi / boto3` 等
> Python 依赖仍会从 PyPI（或镜像）下载。详见 [`../wheels/README.md`](../wheels/README.md)。

---

## 0. 前置

| 项目 | 要求 |
|---|---|
| Python | **3.12**（仓库自带 wheel 为 `cp312`） |
| uv | 最新版（<https://docs.astral.sh/uv/>） |
| 操作系统 | **Linux x86_64**（仓库自带 wheel 为 `linux_x86_64`） |
| 网络 | 不需要 Nexus；Python 依赖需要 PyPI 或镜像 |

> ✅ **不需要** pip.conf / Nexus 凭据 / 内网 editable 源
>
> ⚠️ Windows 请使用 WSL2，macOS 请使用 Docker（linux/amd64），详见
> [`../docs/environment.md`](../docs/environment.md)。ARM64 / Python 3.11 或 3.13
> 需要维护者额外提供 `py3-none-any` wheel；当前 zip 中没有。

---

## 1. 准备 wheels 目录

仓库根目录已自带 `wheels/tongagents-wheels.zip`，先解压：

```bash
cd tongagents-course

unzip -o wheels/tongagents-wheels.zip -d wheels/

ls -lh wheels/
# 期望看到:
# tongagents-2.7.65-cp312-cp312-linux_x86_64.whl      (17M)
# tongagents_cli-1.8.65-cp312-cp312-linux_x86_64.whl  (5.0M)

# 如果维护者单独发了 whl，也可以直接复制到 wheels/
# cp /path/to/tongagents-*.whl wheels/
# cp /path/to/tongagents_cli-*.whl wheels/
```

> 💡 当前 zip 只提供 `cp312-cp312-linux_x86_64`。其他平台 / Python 版本请联系维护者获取
> 对应的 `py3-none-any` wheel。

---

## 2. 创建 / 同步环境（uv）

```bash
# 安装 uv（如果还没有）
curl -LsSf https://astral.sh/uv/install.sh | sh

cd cli-sample
uv sync
```

`uv sync` 会读取 `pyproject.toml` 的 `[tool.uv.sources]`，安装仓库本地 wheel，
并创建 `.venv`，使用仓库中的 `uv.lock` 锁定依赖。不需要手动 `pip install`。

---

## 3. 安装本地 wheel（由 uv 自动完成）

`cli-sample/pyproject.toml` 已声明：

```toml
dependencies = [
    "tongagents==2.7.65",
    "tongagents-cli==1.8.65",
    "python-dotenv>=1.0.0",
]

[tool.uv.sources]
tongagents = { path = "../wheels/tongagents-2.7.65-cp312-cp312-linux_x86_64.whl" }
tongagents-cli = { path = "../wheels/tongagents_cli-1.8.65-cp312-cp312-linux_x86_64.whl" }
```

所以只要先解压 wheel，`uv sync` 就会自动安装，不会去公共 PyPI 找 tongagents。

---

## 4. 验证安装

```bash
# 4.1 验证 Python 模块
uv run python -c "import tongagents; print('tongagents', tongagents.__version__)"

# 4.2 验证 SDK 核心 API
uv run python -c "
from tongagents.agent import Agent, AgentSettings
print('SDK OK')
print('AgentSettings fields:', list(AgentSettings.model_fields.keys()))
"

# 4.3 验证本地 tools 子包
uv run python -c "
import sys; sys.path.insert(0, '.')
from tools import bash_tool, read_file_tool, write_file_tool
print('local tools OK')
"
```

预期输出:

```
tongagents 2.7.65
SDK OK
AgentSettings fields: [...]
local tools OK
```

---

## 5. 配置 LLM 环境变量

```bash
# 5.1 复制 .env 模板
cp .env.example .env

# 5.2 编辑 .env, 填入实际 LLM 配置 (OpenAI 兼容端点)
# OPENAI_API_KEY=sk-your-key
# OPENAI_BASE_URL=http://your-llm/v1
# OPENAI_MODEL=your-model
vim .env
```

---

## 6. 跑示例

```bash
uv run python cli_sample.py
# 启动 REPL, 输入问题, SDK 自动调本地 tools
# 输入 'exit' / 'quit' / Ctrl+D 退出
```

---

## 7. 常见问题

### 7.1 wheel 与本地 Python ABI 不匹配

错误: `tongagents-2.7.65-cp312-cp312-linux_x86_64.whl is not a supported wheel on this platform.`

解决：

- 确认系统是 **Linux x86_64**、Python 是 **3.12.x**。
- Windows 请使用 WSL2；macOS 请使用 Docker（linux/amd64）。
  详见 [`../docs/environment.md`](../docs/environment.md)。
- 当前仓库 zip 没有 `py3-none-any` wheel；如果确实需要其他平台，请联系维护者。

### 7.2 `ModuleNotFoundError: No module named 'tongagents'`

- 确认已执行 `uv sync`（不是只执行 `uv run`）。
- 确认已解压 `wheels/tongagents-wheels.zip`。
- 确认 `pyproject.toml` 里的 `[tool.uv.sources]` 路径指向 `../wheels/` 下真实存在的 whl。

### 7.3 CLI 跑起来但 LLM 不响应

- 检查 `.env` 是否配了 `OPENAI_API_KEY` / `OPENAI_BASE_URL` / `OPENAI_MODEL`
- 跑: `uv run python -c "from dotenv import dotenv_values; print(dotenv_values('.env'))"`

---

## 8. 升级 / 卸载

```bash
# 升级依赖（重新解析 uv.lock）
uv sync --upgrade

# 彻底删除本地环境（保留仓库中的 uv.lock）
rm -rf .venv
```

---

## 9. 为什么用本地 wheel?

| 方案 | 优点 | 缺点 |
|---|---|---|
| **本地 wheel (推荐)** | 不需要 Nexus 凭据 / 跨环境一致 / 版本可控 | 需维护者分发 wheel；依赖仍需 PyPI |
| Nexus pip.conf | 自动解析依赖 | 需要内网 + 凭据 + 写 ~/.config/pip/pip.conf |
| `[tool.uv.sources]` editable | 调试方便 | 需要 uv + 内网 + 编辑源 |

本示例面向**外部开发者**（无 Nexus 权限），统一用本地 wheel。
