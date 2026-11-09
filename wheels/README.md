# 本地 wheel 说明

本目录用于放置课程运行所需的 TongAgents 本地 wheel。

## 仓库自带内容

`tongagents-wheels.zip` 已包含：

```text
tongagents-2.7.65-cp312-cp312-linux_x86_64.whl
tongagents_cli-1.8.65-cp312-cp312-linux_x86_64.whl
tong_etcd_sdk-0.0.11-py3-none-any.whl
```

先解压，然后进入任意 lesson 执行 `uv sync`：

```bash
cd tongagents-course
unzip -o wheels/tongagents-wheels.zip -d wheels/
ls -lh wheels/

cd lesson1
uv sync
```

> ⚠️ `wheels/.gitignore` 会忽略解压出来的 `*.whl`，这是正常的：wheel 文件不重复提交，
> 统一通过 zip 分发。各 lesson 的 `pyproject.toml` 里 `[tool.uv.sources]` 会指向
> `../wheels/` 下这些 wheel 文件。

## 平台与 Python 版本

仓库自带的是 **Linux x86_64 + Python 3.12** 的 `cp312` wheel。

- Linux x86_64 + Python 3.12：可直接使用。
- Windows：推荐使用 **WSL2 + Ubuntu**，见
  <https://learn.microsoft.com/zh-cn/windows/wsl/install>。
- macOS / ARM64 / Python 3.11 或 3.13：当前 zip **没有**可用的
  `py3-none-any` wheel。macOS 推荐使用 Docker（linux/amd64），详见
  [`../docs/environment.md`](../docs/environment.md)。
- 安装前建议确认：

```bash
python3 --version   # 期望 3.12.x
uname -m            # 期望 x86_64
```

## 网络依赖

“本地 wheel”只表示 **不依赖 BigAI Nexus**，并不代表完全离线。

`tongagents` 的运行时依赖（`pydantic`、`litellm`、`fastapi`、`boto3`、`spacy` 等）
仍会从 PyPI 解析和下载。没有外网的学生需要配置可用的 PyPI 镜像，或由维护者额外提供
依赖 wheelhouse。

## 维护者单独分发 whl 时

如果维护者单独提供了 whl 文件，也可以直接复制：

```bash
cp /path/to/tongagents-*.whl wheels/
cp /path/to/tongagents_cli-*.whl wheels/
```
