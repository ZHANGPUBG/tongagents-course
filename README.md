# TongAgents Course

智能体实践课程代码库（从 TongAgent 到 OpenClaw）

11 个 Lesson · 8 个核心 + 3 个选修 + 1 个贯穿大作业

---

## 🚀 快速开始（uv 管理环境）

本课程统一使用 [uv](https://docs.astral.sh/uv/) 管理 Python 环境。每个示例目录都有
自己的 `pyproject.toml`，依赖中的 TongAgents SDK 指向仓库根目录 `wheels/` 下的本地
wheel，因此 `uv sync` 可以直接完成安装。

> **平台说明**：仓库自带 wheel 是 **Linux x86_64 + Python 3.12**。
> Windows 请使用 **WSL2**，macOS 请使用 **Docker（linux/amd64）**。
> 详细步骤见 [`docs/environment.md`](docs/environment.md)。
>
> - Windows WSL2 安装文档：<https://learn.microsoft.com/zh-cn/windows/wsl/install>
> - uv 安装文档：<https://docs.astral.sh/uv/getting-started/installation/>

### 0. 安装 uv

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
uv --version
```

### 1. 解压本地 wheel

```bash
cd tongagents-course    # 仓库根目录
unzip -o wheels/tongagents-wheels.zip -d wheels/
ls -lh wheels/
```

### 2. 跑任一示例

```bash
# 方式 A: cli-sample (REPL + 6 个本地工具)
cd cli-sample
uv sync
cp .env.example .env   # 填写 OPENAI_API_KEY / OPENAI_BASE_URL / OPENAI_MODEL
uv run python cli_sample.py

# 方式 B: lesson1 (3 个示例 + 32 测试)
cd ..
cd lesson1
uv sync
uv run python echo_agent.py
uv run python agent_settings_demo.py
uv run python node_declare_demo.py
uv run python -m pytest tests/ -v

# 方式 C: lesson2 (6 个示例 + 87 测试)
cd ..
cd lesson2
uv sync
uv run python prompt_template.py
uv run python cot_demo.py
uv run python cot_self_consistency.py
uv run python tot_demo.py
uv run python rag_demo.py
uv run python cot_prompt_agent.py
uv run python -m pytest tests/ -v
```

> 运行 `cli-sample` 前先 `cp .env.example .env`，并填写
> `OPENAI_API_KEY / OPENAI_BASE_URL / OPENAI_MODEL` 三项；只填 Key 会因缺少
> `OPENAI_BASE_URL` 启动失败。lesson1 / lesson2 的示例不需要真实 LLM Key。

每个 lesson 的详细步骤见各自的 `INSTALL.md`。平台/网络问题见
[`docs/environment.md`](docs/environment.md) 和 [`wheels/README.md`](wheels/README.md)。

---

## 课程结构

| Lesson | 主题 | 内容 |
|---|---|---|
| 01 | Agent 基本概念 | 演进 · 分类 · 循环 |
| 02 | 前 Agent 时期技术 | CoT · RAG |
| 03 | [Streaming 智能体](lesson3/README.md) | SDK · CLI · 流式事件 |
| 04 | 工具使用 / MCP / Sandbox | Function Call |
| 05 | [ReAct 论文检索综述](lesson5/README.md) | arXiv Tool · ReAct · 证据写作 |
| 06 | 记忆与 RAG 检索 | Vector · KG |
| 07 | 多智能体 / A2A / ANP | 协作 · 网络 |
| 08 | 评测与自演进 | Harness |
| 选修 | 安全 · 多模态 · 行业应用 | - |
| 大作业 | OpenClaw 多智能体项目 | - |

## TongAgents 示例

所有从 PPT 提取的 TongAgents SDK 代码按课程主题整理于各 lesson 目录。PPT 中明确出现的 SDK 代码集中在 Lesson 1、Lesson 2 和课程作业示例；其余 lesson 目录提供对应目录结构和 README，说明课程主题与后续扩展入口。

## 安装方式对比

| 方案 | 优点 | 缺点 | 适用 |
|---|---|---|---|
| **本地 wheel (本课程默认)** | 离线 / 无 Nexus 凭据 / 跨环境一致 | 需维护者分发 wheel | 外部开发者 |
| Nexus pip.conf | 自动解析依赖 | 需内网 + 凭据 | 内部开发者 |
| `[tool.uv.sources]` editable | 调试方便 | 需 uv + 内网 + 编辑源 | Tong-Agent 维护者 |

---

## 致谢

基于 Tong-Agent SDK 2.4.0+（https://github.com/temp-bigai/Tong-Agent）
