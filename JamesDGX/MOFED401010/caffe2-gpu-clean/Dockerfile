FROM nvidia/cuda:9.0-cudnn7-devel-ubuntu16.04
LABEL maintainer "PengzhiZhu <ldqhyfzpz@qq.com>"
MAINTAINER PengzhiZhu <ldqhyfzpz@qq.com>

ENV MOFED_VERSION 4.0-1.0.1.0

WORKDIR /

# Pick up some MOFED and TF dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
        wget \
        perl \
        lsb-release && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Download and install Mellanox OFED $MOFED_VERSION
RUN wget http://content.mellanox.com/ofed/MLNX_OFED-$MOFED_VERSION/MLNX_OFED_LINUX-$MOFED_VERSION-ubuntu16.04-x86_64.tgz && \
        tar -xzvf MLNX_OFED_LINUX-$MOFED_VERSION-ubuntu16.04-x86_64.tgz && \
        yes y | head -1 | MLNX_OFED_LINUX-$MOFED_VERSION-ubuntu16.04-x86_64/mlnxofedinstall --user-space-only --without-fw-update --all -vvv && \
        cd .. && \
        rm -rf MLNX_OFED_LINUX-$MOFED_VERSION-ubuntu16.04-x86_64 && \
        rm -rf *.tgz && \
    rm -rf /var/lib/apt/lists/*

RUN apt-get update && apt-get install -y --no-install-recommends \
        build-essential \
        curl \
        git \
        libcurl3-dev \
        libfreetype6-dev \
        libpng12-dev \
        libzmq3-dev \
        pkg-config \
        python-dev \
        rsync \
        software-properties-common \
        unzip \
        zip \
        zlib1g-dev \
        wget \
        && \
    rm -rf /var/lib/apt/lists/* && \
    find /usr/local/cuda-9.0/lib64/ -type f -name 'lib*_static.a' -not -name 'libcudart_static.a' -delete && \
    rm /usr/lib/x86_64-linux-gnu/libcudnn_static_v7.a

RUN curl -fSsL -O https://bootstrap.pypa.io/get-pip.py && \
    python get-pip.py && \
    rm get-pip.py

RUN pip --no-cache-dir install \
        ipykernel \
        jupyter \
        matplotlib \
        numpy \
        scipy \
        sklearn \
        pandas \
        && \
    python -m ipykernel.kernelspec


# caffe2 install with gpu support

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    cmake \
    git \
    libgflags-dev \
    libgoogle-glog-dev \
    libgtest-dev \
    libiomp-dev \
    libleveldb-dev \
    liblmdb-dev \
    libopencv-dev \
    libopenmpi-dev \
    libprotobuf-dev \
    libsnappy-dev \
    openmpi-bin \
    openmpi-doc \
    protobuf-compiler \
    python-dev \
    python-numpy \
    python-pip \
    python-pydot \
    python-setuptools \
    python-scipy \
    wget \
    redis-tools \
    libhiredis-dev \
    && rm -rf /var/lib/apt/lists/*

RUN pip install --no-cache-dir \
    flask \
    future \
    graphviz \
    hypothesis \
    jupyter \
    matplotlib \
    numpy \
    protobuf \
    pydot \
    python-nvd3 \
    pyyaml \
    requests \
    scikit-image \
    scipy \
    setuptools \
    six \
    tornado

########## INSTALLATION STEPS ###################
RUN git clone --branch master --recursive https://github.com/pytorch/pytorch.git
RUN cd pytorch && mkdir build && cd build \
    && cmake .. \
    -DCMAKE_INSTALL_PREFIX=/pytorch-caffe2 \
    -DCUDA_ARCH_NAME=Manual \
    -DCUDA_ARCH_BIN="60 70" \
    -DCUDA_ARCH_PTX="70" \
    -DUSE_ACL=OFF \
    -DUSE_ASAN=OFF \
    -DUSE_ATEN=OFF \
    -DUSE_FFMPEG=OFF \
    -DUSE_LITE_PROTO=OFF \
    -DUSE_NATIVE_ARCH=OFF \
    -DUSE_NERVANA_GPU=OFF \
    -DUSE_NNAPI=OFF \
    -DUSE_OBSERVERS=OFF \
    -DUSE_OPENCL=OFF \
    -DUSE_OPENMP=OFF \
    -DUSE_PROF=OFF \
    -DUSE_ROCKSDB=OFF \
    -DUSE_SNPE=OFF \
    -DUSE_TENSORRT=OFF \
    -DUSE_ZMQ=OFF \
    -DUSE_ZSTD=OFF \
    -DUSE_LEVELDB=OFF \
    -DUSE_NNPACK=OFF \
    -DUSE_MPI=OFF \
    -DUSE_MOBILE_OPENGL=OFF \
    -DUSE_METAL=OFF \
    -DUSE_IDEEP=OFF \
    -DUSE_OPENCV=ON \
    -DBUILD_BINARY=ON \
    -DBUILD_CUSTOM_PROTOBUF=ON \
    -DBUILD_PYTHON=ON \
    -DBUILD_SHARED_LIBS=ON \
    -DBUILD_TEST=ON \
    -DUSE_NUMA=ON \
    -DUSE_NCCL=ON \
    -DUSE_CUDA=ON \
    -DUSE_GFLAGS=ON \
    -DUSE_GLOG=ON \
    -DUSE_GLOO=ON \
    -DUSE_LMDB=ON \
    -DUSE_MKLML=ON \
    -DUSE_REDIS=ON \
    -DUSE_IBVERBS=ON \
    && make -j"$(nproc)" install \
    && ldconfig \
    && make clean \
    && cd .. \
    && rm -rf build \
    && cd / \
    && rm -rf pytorch
    

ENV PYTHONPATH /pytorch-caffe2/lib/python2.7/dist-packages/:$PYTHONPATH
ENV LD_LIBRARY_PATH /pytorch-caffe2/lib/:$LD_LIBRARY_PATH

# For Singularity
RUN touch /usr/bin/nvidia-smi
