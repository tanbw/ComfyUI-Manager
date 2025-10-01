# 使用官方的 NVIDIA CUDA 镜像作为基础，cu121 对应 CUDA 12.1
# devel 版本包含了构建工具，对于编译 Python 库更稳定
FROM docker.1ms.run/nvidia/cuda:12.1.1-devel-ubuntu22.04
# 声明可以从外部接收的构建参数
ARG HTTP_PROXY
ARG HTTPS_PROXY
ARG NO_PROXY

# 将接收到的参数设置为整个构建过程和最终容器的环境变量
# 设置小写和大写两个版本以兼容各种工具

# 设置工作目录
WORKDIR /app

# 设置环境变量，防止 apt-get 在构建时弹出交互窗口
ENV DEBIAN_FRONTEND=noninteractive

# 1. 安装基础依赖：git 用于克隆仓库，python3-venv 用于创建虚拟环境
RUN apt-get update && \
    apt-get install -y --no-install-recommends git python3.10-venv python3-pip libglib2.0-0 libgl1-mesa-glx && \
    rm -rf /var/lib/apt/lists/*

# 2. 克隆 ComfyUI 和 ComfyUI-Manager 仓库
RUN git clone https://ghfast.top/github.com/comfyanonymous/ComfyUI.git && \
    cd ComfyUI/custom_nodes && \
    git clone https://ghfast.top/github.com/ltdrdata/ComfyUI-Manager.git

# 进入 ComfyUI 目录
WORKDIR /app/ComfyUI

# 3. 创建并激活 Python 虚拟环境，然后安装所有依赖
# 注意：在 Dockerfile 的 RUN 指令中，不能使用 'source'。
# 我们直接使用 venv 内的 python 和 pip 可执行文件来确保所有包装在虚拟环境中。

RUN python3 -m venv venv && \
    ./venv/bin/python -m pip install --upgrade pip && \
    ./venv/bin/python -m pip install torch torchvision torchaudio --extra-index-url https://mirrors.aliyun.com/pytorch-wheels/cu121/
RUN ./venv/bin/python -m pip install -r requirements.txt 
RUN ./venv/bin/python -m pip install -r custom_nodes/ComfyUI-Manager/requirements.txt
RUN ./venv/bin/python -m pip install https://ghfast.top/github.com/Dao-AILab/flash-attention/releases/download/v2.8.3/flash_attn-2.8.3+cu12torch2.8cxx11abiTRUE-cp310-cp310-linux_x86_64.whl

# 暴露 ComfyUI 将要监听的端口
EXPOSE 7865

# 4. 设置容器启动时要执行的默认命令
# 这等同于运行 run_gpu.sh 的效果，并直接加入了您需要的参数
# --listen 0.0.0.0 让服务可以被外部访问
# --port 7865 指定端口
CMD ["./venv/bin/python", "main.py", "--listen", "0.0.0.0", "--port", "7865", "--preview-method", "auto"]
