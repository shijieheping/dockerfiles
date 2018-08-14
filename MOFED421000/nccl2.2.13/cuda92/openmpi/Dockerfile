FROM nvidia/cuda:9.2-cudnn7-devel-ubuntu16.04
LABEL maintainer "PengzhiZhu <ldqhyfzpz@qq.com>"
MAINTAINER PengzhiZhu <ldqhyfzpz@qq.com>

ENV MOFED_VERSION 4.2-1.0.0.0

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

#RUN    find /usr/local/cuda-9.2/lib64/ -type f -name 'lib*_static.a' -not -name 'libcudart_static.a' -delete && \
#    rm /usr/lib/x86_64-linux-gnu/libcudnn_static_v7.a

# docker run --network host -ti -p 8080:80 -v /root/httpd:/usr/local/apache2/htdocs httpd
RUN cd / && wget http://localhost/nccl_2.2.13-1+cuda9.2_x86_64.txz && \
    tar -xf nccl_2.2.13-1+cuda9.2_x86_64.txz && \
    rm -f nccl_2.2.13-1+cuda9.2_x86_64.txz

# RUN ln -s -f /usr/bin/python3 /usr/bin/python#

WORKDIR /root

# For Singularity
RUN touch /usr/bin/nvidia-smi

# How to open a port
#EXPOSE 8888
