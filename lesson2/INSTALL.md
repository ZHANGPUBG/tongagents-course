# Lesson 2 安装文档 — 本地 wheel 方式 (无 Nexus 权限)

> **v0.2.0 重要变更**: 本示例 **不再依赖 BigAI Nexus 私有源**。仓库自带
> `wheels/tongagents-wheels.zip`，解压后用 `pip install ./wheels/*.whl` 安装即可。
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
| pip | **>= 21.3**（支持 PEP 517） |
| 操作系统 | **Linux x86_64**（仓库自带 wheel 为 `linux_x86_64`） |
| 网络 | 不需要 Nexus；Python 依赖需要 PyPI 或镜像 |

> ⚠️ macOS / Windows / ARM64 / Python 3.11 或 3.13 需要维护者额外提供
> `py3-none-any` wheel；当前 zip 中没有。

---

## 1. 准备 wheels 目录

```bash
cd tongagents-course    # 仓库根目录
unzip -o wheels/tongagents-wheels.zip -d wheels/
ls wheels/
# 期望看到 tongagents-*.whl + tongagents_cli-*.whl
```

---

## 2. 创建虚拟环境 + 装本地 wheel

```bash
cd lesson2
python3.12 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip

# 装本地 wheel (不走 nexus, 不走 [tool.uv.sources])
pip install ../wheels/tongagents-*.whl
pip install ../wheels/tongagents_cli-*.whl

# 装 lesson2 自身依赖
pip install -e .
```

---

## 3. 验证安装 + 跑示例 + 测试

```bash
# 验证
python -c "import tongagents; print('tongagents', tongagents.__version__)"

# 跑 6 个示例
python prompt_template.py
python cot_demo.py
python cot_self_consistency.py
python tot_demo.py
python rag_demo.py
python cot_prompt_agent.py

# 跑测试
python -m pytest tests/ -v
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
python -m pytest tests/test_cot.py -v
python -m pytest tests/test_rag.py -v
```

---

## 5. 升级 / 卸载

```bash
# 升级 wheel
pip install --upgrade ../wheels/tongagents-*.whl
pip install --upgrade ../wheels/tongagents_cli-*.whl

# 卸载
pip uninstall tongagents tongagents-cli tongagents-lesson2 -y
```

---

## 6. 参考链接

- **lesson1 INSTALL**: [../lesson1/INSTALL.md](../lesson1/INSTALL.md) — 详细安装文档
- **课程仓库**: `https://github.com/temp-bigai/tongagents-course`
- **tongagents 主项目**: `https://github.com/temp-bigai/Tong-Agent`
