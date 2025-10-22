FROM nvidia/cuda:12.8.0-cudnn-runtime-ubuntu22.04
ENV DEBIAN_FRONTEND=noninteractive PIP_DISABLE_PIP_VERSION_CHECK=1 PYTHONDONTWRITEBYTECODE=1 PYTHONUNBUFFERED=1
RUN apt-get update && apt-get install -y --no-install-recommends python3 python3-pip git ca-certificates && rm -rf /var/lib/apt/lists/*
WORKDIR /workspace
RUN python3 -m pip install --upgrade pip \
 && pip install torch==2.2.2+cu121 torchvision==0.17.2+cu121 --index-url https://download.pytorch.org/whl/cu121
COPY segnet-requirements.txt /tmp/segnet-requirements.txt
RUN pip install --no-cache-dir -r /tmp/segnet-requirements.txt
EXPOSE 8888
CMD ["jupyter","lab","--ip=0.0.0.0","--no-browser","--NotebookApp.token=","--NotebookApp.password=","--allow-root"]