FROM nvidia/cudagl:11.4.0-devel-ubuntu20.04

ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=America/Chicago
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    wget curl software-properties-common

# Install python3.9
RUN add-apt-repository ppa:deadsnakes/ppa
RUN apt update && apt install -y python3.9 python3.9-dev python3.9-distutils
RUN ln -s `which python3.9` /usr/local/bin/python
RUN ln -s `which python3.9` /usr/bin/python
RUN wget https://bootstrap.pypa.io/get-pip.py
RUN python get-pip.py && rm get-pip.py

ENV PIP_NO_CACHE_DIR=1
RUN apt-get update && apt-get install -y --no-install-recommends \
    tree \
    zsh  \
    git \
    git-lfs \
    cmake \
    build-essential \
    libboost-program-options-dev \
    libboost-filesystem-dev \
    libboost-graph-dev \
    libboost-system-dev \
    libboost-test-dev \
    libeigen3-dev \
    libsuitesparse-dev \
    libfreeimage-dev \
    libmetis-dev \
    libgoogle-glog-dev \
    libgflags-dev \
    libglew-dev \
    qtbase5-dev \
    libqt5opengl5-dev \
    libcgal-dev \
    libcgal-qt5-dev

RUN pip install --upgrade pip
RUN pip install pytest pytest-forked lz4 pyyaml
RUN pip install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cu116

# Install typical dependencies
RUN pip install einops sklearn tqdm scipy h5py matplotlib


# install system dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    apt-utils \
    ffmpeg \
    libffi-dev \
    libfreetype6-dev \
    libgl1-mesa-dev \
    libgl1-mesa-glx \
    libosmesa6-dev \
    libglfw3 \
    gcc \
    pciutils \
    xserver-xorg \
    xserver-xorg-video-fbdev \
    xauth

# install GLX-Gears (for debugging)
RUN apt-get update && apt-get install -y \
    mesa-utils \
    python3-setuptools \
    && rm -rf /var/lib/apt/lists/*



### Install mujoco ###
RUN curl -o /usr/local/bin/patchelf https://s3-us-west-2.amazonaws.com/openai-sci-artifacts/manual-builds/patchelf_0.9_amd64.elf \
    && chmod +x /usr/local/bin/patchelf

# ENV LD_LIBRARY_PATH /root/.mujoco/mujoco210/bin:${LD_LIBRARY_PATH}
ENV LD_LIBRARY_PATH /usr/local/nvidia/lib64:${LD_LIBRARY_PATH}
COPY vendor/Xdummy /usr/local/bin/Xdummy
RUN chmod +x /usr/local/bin/Xdummy

# Workaround for https://bugs.launchpad.net/ubuntu/+source/nvidia-graphics-drivers-375/+bug/1674677
COPY ./vendor/10_nvidia.json /usr/share/glvnd/egl_vendor.d/10_nvidia.json

# NOTE: Dockerfile is so counter-intuitive.
# `COPY ./vendor .` or `COPY ./vendor ./` copies the contents of ./vendor to .
# https://stackoverflow.com/a/72376058/7057866
WORKDIR /opt/mujoco_py
COPY ./vendor ./vendor

RUN mkdir -p /usr/local/etc/mujoco \
    && wget https://mujoco.org/download/mujoco210-linux-x86_64.tar.gz -O mujoco.tar.gz \
    && tar -xf mujoco.tar.gz -C /usr/local/etc/mujoco \
    && rm mujoco.tar.gz
ENV MUJOCO_PY_MUJOCO_PATH="/usr/local/etc/mujoco/mujoco210"
ENV LD_LIBRARY_PATH="/usr/local/etc/mujoco/mujoco210/bin"
# Compile-time library location
ENV LIBRARY_PATH=$LD_LIBRARY_PATH
RUN ln -s /usr/local/etc/mujoco $HOME/.mujoco \
    && mkdir -p /opt/mujoco_py/mujoco_py/binaries \
    && ln -s /usr/local/etc/mujoco /opt/mujoco_py/mujoco_py/binaries/linux

# From requirements-ext.txt
RUN pip install \
    "numpy>=1.20.0" \
    "numba>=0.52.0,<=0.53.1" \
    "scipy>=1.2.3" \
    "free-mujoco-py==2.1.6"

# This compiles /usr/local/lib/python3.8/dist-packages/mujoco_py/cymj.pyx
RUN python -c 'import mujoco_py'

# Necessary for Singularity
RUN chmod -R a+w /usr/local/lib/python3.9/dist-packages/mujoco_py/generated


# Install wandb and its (optional) dependencies
RUN pip install \
    wandb \
    kornia \
    ffmpeg \
    imageio \
    imageio-ffmpeg \
    moviepy

ENTRYPOINT ["/opt/mujoco_py/vendor/Xdummy-entrypoint"]
