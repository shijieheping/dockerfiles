FROM nvidia/cuda:9.0-cudnn7-devel-ubuntu16.04
LABEL maintainer "PengzhiZhu <ldqhyfzpz@qq.com>"
MAINTAINER PengzhiZhu <ldqhyfzpz@qq.com>

WORKDIR /

ENV MOFED_VERSION=4.0-1.0.1.0
# Pick up some MOFED and TF dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
        wget \
        perl \
        lsb-release && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN wget http://content.mellanox.com/ofed/MLNX_OFED-$MOFED_VERSION/MLNX_OFED_LINUX-$MOFED_VERSION-ubuntu16.04-x86_64.tgz && \
        tar -xzvf MLNX_OFED_LINUX-$MOFED_VERSION-ubuntu16.04-x86_64.tgz && \
        yes y | head -1 | MLNX_OFED_LINUX-$MOFED_VERSION-ubuntu16.04-x86_64/mlnxofedinstall --user-space-only --without-fw-update --all -vvv && \
	dpkg -i MLNX_OFED_LINUX-$MOFED_VERSION-ubuntu16.04-x86_64/DEBS/openmpi_*_all.deb && \
        cd .. && \
        rm -rf MLNX_OFED_LINUX-$MOFED_VERSION-ubuntu16.04-x86_64 && \
        rm -rf *.tgz && \
    rm -rf /var/lib/apt/lists/*

ENV NCCL_VERSION 2.3.7
# Install NCCL2 v2.3.7-1
RUN apt-get purge -y --allow-change-held-packages \
	"*nccl*" && \
	apt-get update && apt-get install -y --no-install-recommends \
        git \
	devscripts && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
	apt-get update && apt-get install -y --no-install-recommends \
        libnccl2=$NCCL_VERSION-1+cuda9.0 \
	libnccl-dev=$NCCL_VERSION-1+cuda9.0 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
#	git clone --branch=v$NCCL_VERSION-1 https://github.com/NVIDIA/nccl.git && \
#	cd nccl && make pkg.debian.build -j && \
#	dpkg -i build/pkg/deb/libnccl*.deb && \
#	cd .. && \
#	rm -rf nccl

#RUN apt-get update && apt-get install -y --no-install-recommends \
#        python3-pip && \
#    apt-get clean && \
#    rm -rf /var/lib/apt/lists/*
RUN apt-get update && apt-get install -y --no-install-recommends \
        python3 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN wget https://bootstrap.pypa.io/get-pip.py && \
    python3 get-pip.py && \
    rm get-pip.py

RUN pip3 install setuptools && pip3 install tensorflow-gpu==1.12.0

# Benchmarks
WORKDIR /benchmarks
RUN git clone --branch=cnn_tf_v1.12_compatible https://github.com/tensorflow/benchmarks.git . && \
    git reset --hard cnn_tf_v1.12_compatible
