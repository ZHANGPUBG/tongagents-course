# Lesson 1 安装文档 — 本地 wheel 方式 (无 Nexus 权限)

> **v0.2.0 重要变更**: 本示例 **不再依赖 BigAI Nexus 私有源**。仓库自带
> `wheels/tongagents-wheels.zip`，解压后执行 `uv sync` 即可。
>
> ⚠️ “本地 wheel”只表示不需要 Nexus；`pydantic / litellm / fastapi / boto3` 等
> Python 依赖仍会从 PyPI（或镜像）下载。详见 [`../wheels/README.md`](../wheels/README.md)。

---

## 0. 环境先决条件

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
cd tongagents-course    # 仓库根目录

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

## 2. 配置 pip 临时使用 Nexus（**不需要** — 已废弃）

> ⚠️ v0.2.0 已废弃。本节保留仅为对比参考, 实际安装请走第 3 节 uv 方式。

---

## 3. 创建 / 同步环境（uv）

```bash
# 安装 uv（如果还没有）
curl -LsSf https://astral.sh/uv/install.sh | sh

cd lesson1
uv sync
```

`uv sync` 会读取 `pyproject.toml` 的 `[tool.uv.sources]`，安装仓库本地 wheel，
并创建 `.venv`，使用仓库中的 `uv.lock` 锁定依赖。不需要手动 `pip install`。

---

## 4. 安装本地 wheel（由 uv 自动完成）

`lesson1/pyproject.toml` 已声明：

```toml
dependencies = [
    "tongagents==2.7.65",
    "tongagents-cli==1.8.65",
    "pytest>=7.4.0",
    "pytest-asyncio>=0.23.0",
    "python-dotenv>=1.0.0",
]

[tool.uv.sources]
tongagents = { path = "../wheels/tongagents-2.7.65-cp312-cp312-linux_x86_64.whl" }
tongagents-cli = { path = "../wheels/tongagents_cli-1.8.65-cp312-cp312-linux_x86_64.whl" }
```

所以只要先解压 wheel，`uv sync` 就会自动安装，不会去公共 PyPI 找 tongagents。

---

## 5. 验证安装

```bash
# 5.1 验证 Python 模块
uv run python -c "import tongagents; print('tongagents', tongagents.__version__)"

# 5.2 验证 SDK 核心 API
uv run python -c "
from tongagents.agent import Agent, AgentSettings
from tongagents.workflow.simple_workflow import (
    node_declare, Workflow, NodeBase, NodeConfig,
)
print('All SDK core APIs importable.')
print('AgentSettings fields:', list(AgentSettings.model_fields.keys()))
"

# 5.3 验证本地示例可跑
uv run python echo_agent.py
uv run python agent_settings_demo.py
uv run python node_declare_demo.py
```

预期输出:

```
tongagents 2.7.65
All SDK core APIs importable.
AgentSettings fields: ['name', 'description', 'capabilities', 'model', 'temperature', 'max_iterations', 'verbose', 'topic']
```

---

## 6. 跑测试

```bash
uv run python -m pytest tests/ -v
```

预期：32 个测试 PASS。

---

## 7. 升级 / 卸载

```bash
# 升级依赖（重新解析 uv.lock）
uv sync --upgrade

# 查看本地版本
uv pip show tongagents
uv pip show tongagents-cli

# 彻底删除本地环境（保留仓库中的 uv.lock）
rm -rf .venv
```

---

## 8. 常见问题

### 8.1 wheel 与本地 Python ABI 不匹配

错误: `tongagents-2.7.65-cp312-cp312-linux_x86_64.whl is not a supported wheel on this platform.`

解决：

- 确认系统是 **Linux x86_64**、Python 是 **3.12.x**。
- Windows 请使用 WSL2；macOS 请使用 Docker（linux/amd64）。
  详见 [`../docs/environment.md`](../docs/environment.md)。
- 当前仓库 zip 没有 `py3-none-any` wheel；如果确实需要其他平台，请联系维护者。

### 8.2 `ModuleNotFoundError: No module named 'tongagents'`

- 确认已执行 `uv sync`（不是只执行 `uv run`）。
- 确认已解压 `wheels/tongagents-wheels.zip`。
- 确认 `pyproject.toml` 里的 `[tool.uv.sources]` 路径指向 `../wheels/` 下真实存在的 whl。

### 8.3 缺依赖 / `ImportError`

lesson1 自身依赖（pytest / python-dotenv）和 tongagents 的运行时依赖都由 `uv sync`
统一安装。如果网络不可用，需要提前准备依赖 wheelhouse 或配置 PyPI 镜像。

---

## 9. 为什么用本地 wheel?

| 方案 | 优点 | 缺点 |
|---|---|---|
| **本地 wheel (推荐)** | 不需要 Nexus 凭据 / 跨环境一致 / 版本可控 | 需维护者分发 wheel；依赖仍需 PyPI |
| Nexus pip.conf | 自动解析依赖 | 需要内网 + 凭据 + 写 ~/.config/pip/pip.conf |
| `[tool.uv.sources]` editable | 调试方便 | 需要 uv + 内网 + 编辑源 |

本课程面向**外部开发者**（无 Nexus 权限），统一用本地 wheel。

---

## 10. 参考链接

- **课程仓库**：`https://github.com/temp-bigai/tongagents-course`
- **tongagents 主项目**：`https://github.com/temp-bigai/Tong-Agent`
- **wheel 构建脚本**：`Tong-Agent/scripts/build-and-upload-tongagents.sh`
