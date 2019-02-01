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

# Download and install Mellanox OFED $MOFED_VERSION
RUN wget http://content.mellanox.com/ofed/MLNX_OFED-$MOFED_VERSION/MLNX_OFED_LINUX-$MOFED_VERSION-ubuntu16.04-x86_64.tgz && \
        tar -xzvf MLNX_OFED_LINUX-$MOFED_VERSION-ubuntu16.04-x86_64.tgz && \
        yes y | head -1 | MLNX_OFED_LINUX-$MOFED_VERSION-ubuntu16.04-x86_64/mlnxofedinstall --user-space-only --without-fw-update --all -vvv --without-mlnx-ofed-kernel-utils && \
        cd .. && \
        rm -rf MLNX_OFED_LINUX-$MOFED_VERSION-ubuntu16.04-x86_64 && \
        rm -rf *.tgz && \
    rm -rf /var/lib/apt/lists/*

ENV NCCL_VERSION=2.3.7
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

ENV OMPI_VERSION=4.0.0
# Download and install OpenMPI v4.0.0
RUN apt-get update && apt-get install -y --no-install-recommends \
        gfortran && \
    mkdir /mpi -p && \
    cd /mpi && \
    git clone --branch=v$OMPI_VERSION --depth=1 https://github.com/open-mpi/ompi.git ./ompi$OMPI_VERSION && \
    cd /mpi/ompi$OMPI_VERSION && ./autogen.pl && \
    CC=/usr/bin/gcc CXX=/usr/bin/g++ FC=/usr/bin/gfortran ./configure --with-verbs --with-cuda=/usr/local/cuda-10.0 --prefix=/mpi/ompi4-cuda && \
    make -j $(nproc) install && \
    cp Makefile config.log /mpi/ompi4-cuda && \
    cd / && rm -fr /mpi/ompi$OMPI_VERSION

# Install RSH and SSH clients for MPI
RUN apt-get update && apt-get install -y --no-install-recommends \
        rsh-client \
        ssh && \
    rm -rf /var/lib/apt/lists/*

# For Singularity
RUN touch /usr/bin/nvidia-smi

# How to open a port
#EXPOSE 8888
