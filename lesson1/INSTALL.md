# Lesson 1 安装文档 — 本地 wheel 方式 (无 Nexus 权限)

> **v0.2.0 重要变更**: 本示例 **不再依赖 BigAI Nexus 私有源**。仓库自带
> `wheels/tongagents-wheels.zip`，解压后用 `pip install ./wheels/*.whl` 安装即可。
>
> ⚠️ “本地 wheel”只表示不需要 Nexus；`pydantic / litellm / fastapi / boto3` 等
> Python 依赖仍会从 PyPI（或镜像）下载。详见 [`../wheels/README.md`](../wheels/README.md)。

---

## 0. 环境先决条件

| 项目 | 要求 |
|---|---|
| Python | **3.12**（仓库自带 wheel 为 `cp312`） |
| pip | **>= 21.3**（支持 PEP 517） |
| 操作系统 | **Linux x86_64**（仓库自带 wheel 为 `linux_x86_64`） |
| 网络 | 不需要 Nexus；Python 依赖需要 PyPI 或镜像 |

> ✅ **不需要** pip.conf / Nexus 凭据 / `[tool.uv.sources]` editable
>
> ⚠️ macOS / Windows / ARM64 / Python 3.11 或 3.13 需要维护者额外提供
> `py3-none-any` wheel；当前 zip 中没有。

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

> ⚠️ v0.2.0 已废弃。本节保留仅为对比参考, 实际安装请走第 3 节本地 wheel 方式。

---

## 3. 创建虚拟环境（推荐）

```bash
cd lesson1
python3.12 -m venv .venv
source .venv/bin/activate          # Linux / macOS
# .venv\Scripts\activate           # Windows PowerShell

python -m pip install --upgrade pip  # 升级 pip (避免老版本解析问题)
```

---

## 4. 安装本地 wheel（核心步骤）

```bash
# 4.1 装 tongagents SDK (Agent / AgentSettings / Workflow / node_declare)
pip install ../wheels/tongagents-*.whl

# 4.2 装 tongagents_cli (CLI 工具, SDK 间接依赖)
pip install ../wheels/tongagents_cli-*.whl

# 4.3 装 lesson1 自身依赖 (pytest / python-dotenv)
pip install -e .
```

> ⚠️ **顺序很重要**: 先 wheel, 再 `-e .`。否则 lesson1 会去 PyPI 找 tongagents 失败。

---

## 5. 验证安装

```bash
# 5.1 验证 Python 模块
python -c "import tongagents; print('tongagents', tongagents.__version__)"

# 5.2 验证 SDK 核心 API
python -c "
from tongagents.agent import Agent, AgentSettings
from tongagents.workflow.simple_workflow import (
    node_declare, Workflow, NodeBase, NodeConfig,
)
print('All SDK core APIs importable.')
print('AgentSettings fields:', list(AgentSettings.model_fields.keys()))
"

# 5.3 验证本地示例可跑
python echo_agent.py
python agent_settings_demo.py
python node_declare_demo.py
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
python -m pytest tests/ -v
```

预期：32 个测试 PASS。

---

## 7. 升级 / 卸载

```bash
# 升级 wheel
pip install --upgrade ../wheels/tongagents-*.whl
pip install --upgrade ../wheels/tongagents_cli-*.whl

# 查看本地版本
pip show tongagents
pip show tongagents_cli

# 卸载
pip uninstall tongagents tongagents-cli tongagents-lesson1 -y
```

---

## 8. 常见问题

### 8.1 wheel 与本地 Python ABI 不匹配

错误: `tongagents-2.7.65-cp312-cp312-linux_x86_64.whl is not a supported wheel on this platform.`

解决:

```bash
# 检查 Python 版本
python --version   # 期望 3.12.x

# 当前仓库 zip 没有 py3-none-any wheel。
# 如果是 3.11 / 3.13 / ARM64 / macOS，请联系维护者获取对应版本的通用 wheel：
#   tongagents-<version>-py3-none-any.whl
#   tongagents_cli-<version>-py3-none-any.whl
pip uninstall tongagents tongagents-cli -y
# 然后安装维护者提供的 py3-none-any wheel
```

### 8.2 `ModuleNotFoundError: No module named 'tongagents'`

- 确认 wheel 已装: `pip show tongagents`
- 确认 wheel 在 `../wheels/` 目录下 (相对 lesson1)
- 用绝对路径试一下: `pip install /abs/path/to/wheels/tongagents-*.whl`

### 8.3 缺依赖 / `ImportError`

lesson1 自身依赖（pytest / python-dotenv）通过 `pip install -e .` 装。
tongagents 的运行时依赖（pydantic / litellm / fastapi 等）会在安装 wheel 时从 PyPI
（或镜像）自动解析安装；如果网络不可用，需要提前准备依赖 wheelhouse。

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
