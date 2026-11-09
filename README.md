# TongAgents Course

智能体实践课程代码库（从 TongAgent 到 OpenClaw）

11 个 Lesson · 8 个核心 + 3 个选修 + 1 个贯穿大作业

---

## 🚀 快速开始（开发者，本地 wheel 方式）

> **v0.2.0 重要变更**: 本课程**不依赖 BigAI Nexus 内网**。仓库自带
> `wheels/tongagents-wheels.zip`，解压后用 `pip install ./wheels/*.whl` 安装即可。
>
> ⚠️ “本地 wheel”只表示不需要 Nexus；`pydantic / litellm / fastapi / boto3` 等
> Python 依赖仍会从 PyPI（或镜像）下载。详见 [`wheels/README.md`](wheels/README.md)。

### 环境要求

- **Python 3.12**（仓库自带 wheel 为 `cp312`）
- **Linux x86_64**（仓库自带 wheel 为 `linux_x86_64`）
- 可访问 PyPI 或已配置 PyPI 镜像

> macOS / Windows / ARM64 / Python 3.11 或 3.13 需要维护者额外提供
> `py3-none-any` wheel；当前 zip 中没有。

### 一次性准备 wheels 目录

```bash
cd tongagents-course    # 仓库根目录

# 仓库已自带 zip，先解压
unzip -o wheels/tongagents-wheels.zip -d wheels/

ls -lh wheels/
# 期望看到:
# tongagents-2.7.65-cp312-cp312-linux_x86_64.whl       (17M, Linux x86_64)
# tongagents_cli-1.8.65-cp312-cp312-linux_x86_64.whl   (5.0M, Linux x86_64)

# 如果维护者单独发了 whl，也可以直接复制到 wheels/
# cp /path/to/tongagents-*.whl wheels/
# cp /path/to/tongagents_cli-*.whl wheels/
```

### 跑任一示例

```bash
# 方式 A: cli-sample (REPL + 6 个本地工具)
cd cli-sample
python3.12 -m venv .venv && source .venv/bin/activate
pip install ../wheels/tongagents-*.whl ../wheels/tongagents_cli-*.whl
pip install -e .
python cli_sample.py

# 方式 B: lesson1 (3 个示例 + 32 测试)
cd lesson1
python3.12 -m venv .venv && source .venv/bin/activate
pip install ../wheels/tongagents-*.whl ../wheels/tongagents_cli-*.whl
pip install -e .
python echo_agent.py
python agent_settings_demo.py
python node_declare_demo.py
python -m pytest tests/ -v

# 方式 C: lesson2 (6 个示例 + 87 测试)
cd lesson2
python3.12 -m venv .venv && source .venv/bin/activate
pip install ../wheels/tongagents-*.whl ../wheels/tongagents_cli-*.whl
pip install -e .
python prompt_template.py
python cot_demo.py
python cot_self_consistency.py
python tot_demo.py
python rag_demo.py
python cot_prompt_agent.py
python -m pytest tests/ -v
```

> 运行 `cli-sample` 前先 `cp .env.example .env`，并填写
> `OPENAI_API_KEY / OPENAI_BASE_URL / OPENAI_MODEL` 三项；只填 Key 会因缺少
> `OPENAI_BASE_URL` 启动失败。lesson1 / lesson2 的示例不需要真实 LLM Key。

每个 lesson 的详细步骤见各自的 `INSTALL.md`。

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
