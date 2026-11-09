# 用于 macOS / 非 Linux 环境的课程容器
# 构建（在仓库根目录）:
#   docker build --platform linux/amd64 -f docker/Dockerfile -t tongagents-course .
# 运行:
#   docker run --rm -it --platform linux/amd64 -v "$PWD:/workspace" -w /workspace tongagents-course
#
# Apple Silicon 需要在 Docker Desktop 中开启:
#   Settings -> General -> Use Rosetta for x86_64/amd64 emulation on Apple Silicon

FROM --platform=linux/amd64 python:3.12-slim

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        git \
        unzip \
    && rm -rf /var/lib/apt/lists/*

# 安装 uv（课程统一使用 uv 管理环境）
RUN curl -LsSf https://astral.sh/uv/install.sh | sh
ENV PATH="/root/.local/bin:${PATH}"

WORKDIR /workspace

CMD ["bash"]
