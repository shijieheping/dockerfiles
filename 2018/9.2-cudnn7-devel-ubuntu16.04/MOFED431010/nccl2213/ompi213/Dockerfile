FROM nvidia/cuda:9.2-cudnn7-devel-ubuntu16.04
LABEL maintainer "PengzhiZhu <ldqhyfzpz@qq.com>"
MAINTAINER PengzhiZhu <ldqhyfzpz@qq.com>

ENV MOFED_VERSION 4.3-1.0.1.0

#RUN    find /usr/local/cuda-9.2/lib64/ -type f -name 'lib*_static.a' -not -name 'libcudart_static.a' -delete && \
#    rm /usr/lib/x86_64-linux-gnu/libcudnn_static_v7.a
# RUN ln -s -f /usr/bin/python3 /usr/bin/python#

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

# Download nccl_2.2.13
# docker run --network host -ti -p 8080:80 -v /root/httpd:/usr/local/apache2/htdocs httpd
ENV NCCL_VERSION=2.2.13
RUN apt-get purge -y \
        "*nccl*" && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
    cd / && wget http://localhost/nccl_$NCCL_VERSION-1+cuda9.2_x86_64.txz && \
    tar -xf nccl_$NCCL_VERSION-1+cuda9.2_x86_64.txz && \
    rm -f nccl_$NCCL_VERSION-1+cuda9.2_x86_64.txz

# Download and install OpenMPI v2.1.3
ENV OMPI_VERSION=2.1.3
RUN apt-get update && apt-get install -y --no-install-recommends \
	git && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
RUN mkdir /mpi -p && \
    cd /mpi && \
    git clone --branch=v$OMPI_VERSION --depth=1 https://github.com/open-mpi/ompi.git ./ompi$OMPI_VERSION && \
    cd /mpi/ompi$OMPI_VERSION && ./autogen.pl && \
    CC=/usr/bin/gcc CXX=/usr/bin/g++ FC=/usr/bin/gfortran ./configure --with-verbs --with-cuda=/usr/local/cuda-9.2 --prefix=/mpi/ompi3-cuda && \
    make -j $(nproc) install && \
    cp Makefile config.log /mpi/ompi3-cuda && \
    cd / && rm -fr /mpi/ompi$OMPI_VERSION

# Install RSH and SSH clients for MPI
RUN apt-get update && apt-get install -y --no-install-recommends \
        rsh-client \
        ssh && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /

# For Singularity
RUN touch /usr/bin/nvidia-smi

# How to open a port
#EXPOSE 8888
