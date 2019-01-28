FROM nvidia/cuda:9.2-cudnn7-devel-ubuntu16.04
LABEL maintainer "PengzhiZhu <ldqhyfzpz@qq.com>"
MAINTAINER PengzhiZhu <ldqhyfzpz@qq.com>

WORKDIR /

ENV MOFED_VERSION 4.5-1.0.1.0

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

RUN apt-get purge ucx -y --no-install-recommends && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

ENV OMPI_VERSION=1.8.8
# Download and install OpenMPI v1.8.8
RUN apt-get update && apt-get install -y --no-install-recommends \
        gfortran && \
    mkdir /mpi -p && \
    cd /mpi && \
    git clone --branch=v$OMPI_VERSION --depth=1 https://github.com/open-mpi/ompi.git ./ompi$OMPI_VERSION && \
    cd /mpi/ompi$OMPI_VERSION && ./autogen.pl && \
    mkdir build && cd build && \
    CC=/usr/bin/gcc CXX=/usr/bin/g++ FC=/usr/bin/gfortran ../configure --with-verbs --with-cuda=/usr/local/cuda-9.2 --prefix=/mpi/ompi188-cuda && \
    make -j $(nproc) install && \
    cp Makefile config.log /mpi/ompi188-cuda && \
    cd / && rm -fr /mpi/ompi$OMPI_VERSION

ENV OMPI_VERSION=2.1.6
# Download and install OpenMPI v2.1.6
RUN apt-get update && apt-get install -y --no-install-recommends \
        gfortran && \
    mkdir /mpi -p && \
    cd /mpi && \
    git clone --branch=v$OMPI_VERSION --depth=1 https://github.com/open-mpi/ompi.git ./ompi$OMPI_VERSION && \
    cd /mpi/ompi$OMPI_VERSION && ./autogen.pl && \
    mkdir build && cd build && \
    CC=/usr/bin/gcc CXX=/usr/bin/g++ FC=/usr/bin/gfortran ../configure --with-verbs --with-cuda=/usr/local/cuda-9.2 --prefix=/mpi/ompi216-cuda && \
    make -j $(nproc) install && \
    cp Makefile config.log /mpi/ompi216-cuda && \
    cd / && rm -fr /mpi/ompi$OMPI_VERSION

ENV OMPI_VERSION=3.1.0
# Download and install OpenMPI v3.1.0
RUN apt-get update && apt-get install -y --no-install-recommends \
        gfortran && \
    mkdir /mpi -p && \
    cd /mpi && \
    git clone --branch=v$OMPI_VERSION --depth=1 https://github.com/open-mpi/ompi.git ./ompi$OMPI_VERSION && \
    cd /mpi/ompi$OMPI_VERSION && ./autogen.pl && \
    mkdir build && cd build && \
    CC=/usr/bin/gcc CXX=/usr/bin/g++ FC=/usr/bin/gfortran ../configure --with-verbs --with-cuda=/usr/local/cuda-9.2 --prefix=/mpi/ompi310-cuda && \
    make -j $(nproc) install && \
    cp Makefile config.log /mpi/ompi310-cuda && \
    cd / && rm -fr /mpi/ompi$OMPI_VERSION

ENV OMPI_VERSION=3.1.1
# Download and install OpenMPI v3.1.1
RUN apt-get update && apt-get install -y --no-install-recommends \
        gfortran && \
    mkdir /mpi -p && \
    cd /mpi && \
    git clone --branch=v$OMPI_VERSION --depth=1 https://github.com/open-mpi/ompi.git ./ompi$OMPI_VERSION && \
    cd /mpi/ompi$OMPI_VERSION && ./autogen.pl && \
    mkdir build && cd build && \
    CC=/usr/bin/gcc CXX=/usr/bin/g++ FC=/usr/bin/gfortran ../configure --with-verbs --with-cuda=/usr/local/cuda-9.2 --prefix=/mpi/ompi311-cuda && \
    make -j $(nproc) install && \
    cp Makefile config.log /mpi/ompi311-cuda && \
    cd / && rm -fr /mpi/ompi$OMPI_VERSION

ENV OMPI_VERSION=4.0.0
# Download and install OpenMPI v4.0.0
RUN apt-get update && apt-get install -y --no-install-recommends \
        gfortran && \
    mkdir /mpi -p && \
    cd /mpi && \
    git clone --branch=v$OMPI_VERSION --depth=1 https://github.com/open-mpi/ompi.git ./ompi$OMPI_VERSION && \
    cd /mpi/ompi$OMPI_VERSION && ./autogen.pl && \
    mkdir build && cd build && \
    CC=/usr/bin/gcc CXX=/usr/bin/g++ FC=/usr/bin/gfortran ../configure --with-verbs --with-cuda=/usr/local/cuda-9.2 --prefix=/mpi/ompi400-cuda && \
    make -j $(nproc) install && \
    cp Makefile config.log /mpi/ompi400-cuda && \
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
