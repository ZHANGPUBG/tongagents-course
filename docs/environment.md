# 环境准备（Windows / macOS / Linux）

本课程统一使用 **uv** 管理 Python 环境。仓库自带的 wheel 是
**Linux x86_64 + Python 3.12**（`cp312`），因此：

| 你的系统 | 推荐方案 | 说明 |
|---|---|---|
| Linux x86_64 | 原生运行 | 最简单，直接安装 uv 即可 |
| Windows 10/11 | **WSL2 + Ubuntu** | 在 WSL 里按 Linux 流程运行 |
| macOS Intel | **Docker Desktop（linux/amd64）** | 使用仓库提供的 `docker/Dockerfile` |
| macOS Apple Silicon | **Docker Desktop（linux/amd64 + Rosetta）** | 需要开启 x86_64 模拟 |
| 不想本地装环境 | GitHub Codespaces / 远程 Linux | 需要能访问 GitHub |

> 当前仓库没有 macOS / Windows 原生 wheel。直接在这些系统上
> `uv sync` 会因为 wheel 平台不匹配而失败，请使用下面的 WSL2 或 Docker 方案。

---

## 0. 安装 uv

所有平台通用，官方安装文档：

- <https://docs.astral.sh/uv/getting-started/installation/>

Linux / WSL / macOS 常用安装命令：

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

安装后确认：

```bash
uv --version
python3.12 --version   # 需要 3.12.x
```

仓库根目录有 `.python-version`（`3.12`），各示例目录有 `uv.lock`。
`uv sync` 会自动选择 Python 3.12 并安装锁定版本。

> Windows 原生 PowerShell 也可以安装 uv，但本课程仍建议在 WSL2 里运行，
> 因为 wheel 是 Linux x86_64。

---

## 1. Linux x86_64

```bash
# 安装 Python 3.12（Ubuntu / Debian）
sudo apt update
sudo apt install -y python3.12 python3.12-venv unzip curl

# 安装 uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# 解压课程 wheel
cd tongagents-course
unzip -o wheels/tongagents-wheels.zip -d wheels/

# 任选一个示例
cd cli-sample
uv sync
cp .env.example .env   # 填写 OPENAI_API_KEY / OPENAI_BASE_URL / OPENAI_MODEL
uv run python cli_sample.py
```

---

## 2. Windows：使用 WSL2

Windows 推荐使用 **WSL2 + Ubuntu**。微软官方安装文档（中文）：

- <https://learn.microsoft.com/zh-cn/windows/wsl/install>

英文版：

- <https://learn.microsoft.com/windows/wsl/install>

### 快速步骤

1. 以管理员身份打开 PowerShell：

   ```powershell
   wsl --install -d Ubuntu-24.04
   ```

2. 重启电脑，按提示设置 Ubuntu 用户名和密码。
3. 打开 Ubuntu 终端，安装 Python 3.12、unzip 和 uv：

   ```bash
   sudo apt update
   sudo apt install -y python3.12 python3.12-venv unzip curl
   curl -LsSf https://astral.sh/uv/install.sh | sh
   ```

4. **把仓库放在 WSL 的文件系统里**（例如 `~/tongagents-course`），不要放在
   `/mnt/c/...` 下，否则文件 IO 会很慢：

   ```bash
   cd ~
   git clone https://github.com/temp-bigai/tongagents-course.git
   cd tongagents-course
   unzip -o wheels/tongagents-wheels.zip -d wheels/
   ```

5. 按 Linux 流程运行：

   ```bash
   cd cli-sample
   uv sync
   cp .env.example .env
   uv run python cli_sample.py
   ```

> WSL2 需要 Windows 10 2004+ 或 Windows 11，并在 BIOS 中开启虚拟化。
> 如果 `wsl --install` 失败，请先按微软文档启用“虚拟机平台”和 WSL 功能。

---

## 3. macOS：使用 Docker（linux/amd64）

macOS 原生无法直接使用仓库里的 Linux wheel。推荐用 Docker Desktop 运行
**linux/amd64** 容器；仓库提供了 `docker/Dockerfile`。

### 3.1 安装 Docker Desktop

- <https://www.docker.com/products/docker-desktop/>

Apple Silicon（M1/M2/M3/M4）用户请在 Docker Desktop 设置中开启：

> Settings → General → **Use Rosetta for x86_64/amd64 emulation on Apple Silicon**

否则 linux/amd64 容器会非常慢，甚至无法启动。

### 3.2 构建镜像

在仓库根目录执行：

```bash
docker build --platform linux/amd64 -f docker/Dockerfile -t tongagents-course .
```

### 3.3 运行

```bash
docker run --rm -it --platform linux/amd64 \
  -v "$PWD:/workspace" -w /workspace \
  tongagents-course
```

进入容器后就是 Linux x86_64 + Python 3.12 + uv 环境：

```bash
cd cli-sample
uv sync
cp .env.example .env
uv run python cli_sample.py
```

> 如果使用 Apple Silicon 且没有开启 Rosetta，`docker build` 可能会卡住或报
> `exec format error`。

### 3.4 macOS 的其他选择

- **GitHub Codespaces**：在 GitHub 上打开本仓库，选择 Codespaces，得到一台
  Linux x86_64 开发机，然后按 Linux 流程运行。适合不想本地装 Docker 的同学。
- **Linux 虚拟机**：UTM / Parallels / VMware 安装 Ubuntu 24.04 x86_64，
  然后按 Linux 流程运行。
- **向维护者申请 macOS wheel**：当前 zip 中没有 `py3-none-any` wheel；如果课程
  后续提供，可以直接在 macOS 原生运行。

---

## 4. 验证环境

进入任意 lesson 后：

```bash
uv sync
uv run python -c "import tongagents; print('tongagents', tongagents.__version__)"
```

预期：

```text
tongagents 2.7.65
```

运行 lesson1 的测试：

```bash
cd lesson1
uv sync
uv run python -m pytest tests/ -q
```

预期：`32 passed`。

---

## 5. 常见问题

### Q: `uv sync` 报找不到 wheel

先解压：

```bash
cd tongagents-course
unzip -o wheels/tongagents-wheels.zip -d wheels/
```

### Q: `uv sync` 很慢 / 需要联网

本地 wheel 只解决 TongAgents SDK 本体；`pydantic / litellm / fastapi / boto3`
等依赖仍会从 PyPI 下载。可以配置 PyPI 镜像，例如：

```bash
export UV_DEFAULT_INDEX=https://pypi.tuna.tsinghua.edu.cn/simple
```

### Q: Windows 上直接 `uv sync` 失败

这是预期行为：wheel 是 Linux x86_64。请使用 WSL2 或 Docker。

### Q: macOS 上直接 `uv sync` 失败

同上。请使用 Docker（linux/amd64）或 GitHub Codespaces。
