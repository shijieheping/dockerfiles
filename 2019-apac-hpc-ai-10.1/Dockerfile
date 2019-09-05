FROM nvidia/cuda:10.1-cudnn7-devel-ubuntu16.04
LABEL maintainer "PengzhiZhu <ldqhyfzpz@qq.com>"
MAINTAINER PengzhiZhu <ldqhyfzpz@qq.com>

WORKDIR /

ENV MOFED_VERSION=4.4-2.0.7.0
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
	git clone --branch=v$NCCL_VERSION-1 https://github.com/NVIDIA/nccl.git && \
	cd nccl && make pkg.debian.build -j && \
	dpkg -i build/pkg/deb/libnccl*.deb && \
	cd .. && \
	rm -rf nccl

# Pick up some MOFED and TF dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
        libnuma-dev && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

#Install ucx
RUN cd / && \
git clone --branch=v1.6.x  https://github.com/openucx/ucx.git && \
cd /ucx && \
./autogen.sh --force && \
./contrib/configure-release \
--with-cuda=/usr/local/cuda-10.1 \
--with-rocm \
--with-verbs \
--with-rc \
--with-ud \
--with-dc \
--with-mlx5-dv \
--with-dm \
--with-cm \
--with-rdmacm &&\
make && \
make install

RUN apt-get update && apt-get install -y --no-install-recommends \
	git gfortran \
	ca-certificates
RUN cd / && \
wget https://www.open-mpi.org/software/ompi/v4.0/downloads/openmpi-4.0.0.tar.gz && \
tar zxf openmpi-4.0.0.tar.gz && \
cd openmpi-4.0.0  && \
./configure \
--enable-orterun-prefix-by-default \
--enable-openib-control-hdr-padding \
--enable-openib-rdmacm-ibaddr \
--enable-openib-rdmacm \
--enable-btl-portals4-flow-control \
--enable-opal-btl-usnic-unit-tests \
--enable-timing \
--with-cuda \
--with-slurm \
--with-verbs \
--with-ucx \
--disable-getpwuid \
CXX=g++ FORTRAN=gfortran --enable-mpi-cxx --enable-mpi-fortran --enable-cxx-exceptions && \
make -j $(nproc) all && \
make install

# Install RSH and SSH clients for MPI
RUN apt-get update && apt-get install -y --no-install-recommends \
        rsh-client \
        ssh && \
    rm -rf /var/lib/apt/lists/*

# Download and install nccl-tests
RUN mkdir /nccl-tests -p && cd /nccl-tests && \
    git clone https://github.com/NVIDIA/nccl-tests.git . && \
    make MPI=1 NCCL_HOME=/usr/lib/x86_64-linux-gnu CUDA_HOME=/usr/local/cuda-10.1 MPI_HOME=/usr/local DST_DIR=/ompi4tests/NCCL2 -j $(nproc) && \
    cd / && rm -fr /nccl-tests

RUN apt-get update && apt-get install -y --no-install-recommends \
        python3-pip && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN pip3 install setuptools && pip3 install tensorflow-gpu==1.13.1

# Benchmarks
WORKDIR /benchmarks
RUN git clone --branch=cnn_tf_v1.13_compatible https://github.com/tensorflow/benchmarks.git . && \
    git reset --hard cnn_tf_v1.13_compatible

RUN apt-get update && apt-get install -y --no-install-recommends \
        libpython3.5-dev && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN ln -s /usr/local/cuda/lib64/stubs/libcuda.so /usr/local/cuda/lib64/stubs/libcuda.so.1 && \
    PATH=$PATH:"/usr/local/bin" \
    LD_LIBRARY_PATH=/usr/local/cuda/lib64/stubs:${LD_LIBRARY_PATH} \
    HOROVOD_GPU_ALLREDUCE=NCCL \
    HOROVOD_NCCL_HOME=/usr/lib/x86_64-linux-gnu \
    pip3 install --no-cache-dir horovod

# For Singularity
RUN touch /usr/bin/nvidia-smi

# How to open a port
#EXPOSE 8888
