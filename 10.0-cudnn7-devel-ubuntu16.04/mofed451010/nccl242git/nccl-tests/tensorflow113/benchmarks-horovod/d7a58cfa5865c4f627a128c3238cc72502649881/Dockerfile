FROM nvidia/cuda:10.0-cudnn7-devel-ubuntu16.04
LABEL maintainer "PengzhiZhu <ldqhyfzpz@qq.com>"
MAINTAINER PengzhiZhu <ldqhyfzpz@qq.com>

WORKDIR /

ENV MOFED_VERSION=4.5-1.0.1.0
# Pick up some MOFED and TF dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
        wget \
        perl \
        lsb-release && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN wget http://content.mellanox.com/ofed/MLNX_OFED-$MOFED_VERSION/MLNX_OFED_LINUX-$MOFED_VERSION-ubuntu16.04-x86_64.tgz && \
        tar -xzvf MLNX_OFED_LINUX-$MOFED_VERSION-ubuntu16.04-x86_64.tgz && \
        yes y | head -1 | MLNX_OFED_LINUX-$MOFED_VERSION-ubuntu16.04-x86_64/mlnxofedinstall --user-space-only --without-fw-update --all -vvv --without-mlnx-ofed-kernel-utils && \
	dpkg -i MLNX_OFED_LINUX-4.5-1.0.1.0-ubuntu16.04-x86_64/DEBS/openmpi_4.0.0rc5-1.45101_all.deb && \
        cd .. && \
        rm -rf MLNX_OFED_LINUX-$MOFED_VERSION-ubuntu16.04-x86_64 && \
        rm -rf *.tgz && \
    rm -rf /var/lib/apt/lists/*

ENV NCCL_VERSION 2.4.2

# Install NCCL2 v2.4.2-1
RUN apt-get purge -y --allow-change-held-packages \
	"*nccl*" && \
	apt-get update && apt-get install -y --no-install-recommends \
        git \
	devscripts && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
	git clone https://github.com/NVIDIA/nccl.git && \
	cd nccl && git reset --hard d7a58cfa5865c4f627a128c3238cc72502649881 && make pkg.debian.build -j && \
	dpkg -i build/pkg/deb/libnccl*.deb && \
	cd .. && \
	rm -rf nccl 

# Install RSH and SSH clients for MPI
RUN apt-get update && apt-get install -y --no-install-recommends \
        rsh-client \
        ssh && \
    rm -rf /var/lib/apt/lists/*

# Download and install nccl-tests
RUN mkdir /nccl-tests -p && cd /nccl-tests && \
    git clone https://github.com/NVIDIA/nccl-tests.git . && \
    make MPI=1 NCCL_HOME=/usr/lib/x86_64-linux-gnu CUDA_HOME=/usr/local/cuda-10.0 MPI_HOME=/usr/mpi/gcc/openmpi-4.0.0rc5/ DST_DIR=/ompi4tests/NCCL2 -j $(nproc) && \
    cd / && rm -fr /nccl-tests

RUN apt-get update && apt-get install -y --no-install-recommends \
        python-pip && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN pip install setuptools && pip install tensorflow-gpu

# Benchmarks
WORKDIR /benchmarks
RUN git clone --branch=cnn_tf_v1.13_compatible https://github.com/tensorflow/benchmarks.git . && \
    git reset --hard cnn_tf_v1.13_compatible

RUN apt-get update && apt-get install -y --no-install-recommends \
        python-dev && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN ln -s /usr/local/cuda/lib64/stubs/libcuda.so /usr/local/cuda/lib64/stubs/libcuda.so.1 && \
    PATH=$PATH:"/usr/mpi/gcc/openmpi-4.0.0rc5/bin" \
    LD_LIBRARY_PATH=/usr/local/cuda/lib64/stubs:${LD_LIBRARY_PATH} \
    HOROVOD_GPU_ALLREDUCE=NCCL \
    HOROVOD_NCCL_HOME=/usr/lib/x86_64-linux-gnu \
    pip install --no-cache-dir horovod

# For Singularity
RUN touch /usr/bin/nvidia-smi

# How to open a port
#EXPOSE 8888
