# Lesson 2 安装文档 — 本地 wheel 方式 (无 Nexus 权限)

> **v0.2.0 重要变更**: 本示例 **不再依赖 BigAI Nexus 私有源**。仓库自带
> `wheels/tongagents-wheels.zip`，解压后执行 `uv sync` 即可。
>
> ⚠️ “本地 wheel”只表示不需要 Nexus；`pydantic / litellm / fastapi / boto3` 等
> Python 依赖仍会从 PyPI（或镜像）下载。详见 [`../wheels/README.md`](../wheels/README.md)。

> Lesson 2 复用 Lesson 1 的全部安装流程 + wheel 文件 (同一份 `wheels/` 目录),
> 因此本节简短描述, 详细步骤见 [../lesson1/INSTALL.md](../lesson1/INSTALL.md)。

---

## 0. 环境先决条件

| 项目 | 要求 |
|---|---|
| Python | **3.12**（仓库自带 wheel 为 `cp312`） |
| uv | 最新版（<https://docs.astral.sh/uv/>） |
| 操作系统 | **Linux x86_64**（仓库自带 wheel 为 `linux_x86_64`） |
| 网络 | 不需要 Nexus；Python 依赖需要 PyPI 或镜像 |

> ⚠️ Windows 请使用 WSL2，macOS 请使用 Docker（linux/amd64），详见
> [`../docs/environment.md`](../docs/environment.md)。ARM64 / Python 3.11 或 3.13
> 需要维护者额外提供 `py3-none-any` wheel；当前 zip 中没有。

---

## 1. 准备 wheels 目录

```bash
cd tongagents-course    # 仓库根目录
unzip -o wheels/tongagents-wheels.zip -d wheels/
ls wheels/
# 期望看到 tongagents-*.whl + tongagents_cli-*.whl
```

---

## 2. 创建 / 同步环境（uv）

```bash
# 安装 uv（如果还没有）
curl -LsSf https://astral.sh/uv/install.sh | sh

cd lesson2
uv sync
```

`uv sync` 会读取 `pyproject.toml` 的 `[tool.uv.sources]`，安装仓库本地 wheel，
并创建 `.venv`，使用仓库中的 `uv.lock` 锁定依赖。不需要手动 `pip install`。

---

## 3. 验证安装 + 跑示例 + 测试

```bash
# 验证
uv run python -c "import tongagents; print('tongagents', tongagents.__version__)"

# 跑 6 个示例
uv run python prompt_template.py
uv run python cot_demo.py
uv run python cot_self_consistency.py
uv run python tot_demo.py
uv run python rag_demo.py
uv run python cot_prompt_agent.py

# 跑测试
uv run python -m pytest tests/ -v
```

预期：6 个示例跑通 + 87 个测试 PASS。

---

## 4. 故障排查

详见 [../lesson1/INSTALL.md#8-常见问题](../lesson1/INSTALL.md#8-常见问题)。

lesson2 特有的常见问题:

### 4.1 ToT / RAG demo 需要更多内存

ToT 树搜索 + RAG 向量计算会占用较多内存, 推荐至少 2GB 可用内存。

### 4.2 测试耗时

lesson2 共 87 个测试, 全部跑完约 5-10 秒。如需加速, 可单独跑某个文件:

```bash
uv run python -m pytest tests/test_cot.py -v
uv run python -m pytest tests/test_rag.py -v
```

---

## 5. 升级 / 卸载

```bash
# 升级依赖（重新解析 uv.lock）
uv sync --upgrade

# 彻底删除本地环境（保留仓库中的 uv.lock）
rm -rf .venv
```

---

## 6. 参考链接

- **lesson1 INSTALL**: [../lesson1/INSTALL.md](../lesson1/INSTALL.md) — 详细安装文档
- **课程仓库**: `https://github.com/temp-bigai/tongagents-course`
- **tongagents 主项目**: `https://github.com/temp-bigai/Tong-Agent`
