FROM nvidia/cuda:9.2-cudnn7-devel-ubuntu16.04
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
	git clone --origin=v$NCCL_VERSION-1 https://github.com/NVIDIA/nccl.git && \
	cd nccl && make pkg.debian.build -j && \
	dpkg -i build/pkg/deb/libnccl*.deb && \
	cd .. && \
	rm -rf nccl 

ENV OMPI_VERSION=4.0.0rc5
# Download and install OpenMPI v4.0.0rc5
RUN apt-get update && apt-get install -y --no-install-recommends \
        gfortran && \
    mkdir /mpi -p && \
    cd /mpi && \
    git clone --origin=v$OMPI_VERSION --depth=1 https://github.com/open-mpi/ompi.git ./ompi$OMPI_VERSION && \
    cd /mpi/ompi$OMPI_VERSION && ./autogen.pl && \
    CC=/usr/bin/gcc CXX=/usr/bin/g++ FC=/usr/bin/gfortran ./configure --with-verbs --with-cuda=/usr/local/cuda-9.2 --prefix=/mpi/ompi4-cuda && \
    make -j $(nproc) install && \
    cp Makefile config.log /mpi/ompi4-cuda && \
    cd / && rm -fr /mpi/ompi$OMPI_VERSION

# Install RSH and SSH clients for MPI
RUN apt-get update && apt-get install -y --no-install-recommends \
        rsh-client \
        ssh && \
    rm -rf /var/lib/apt/lists/*

# Download and install nccl-tests
RUN mkdir /nccl-tests -p && cd /nccl-tests && \
    git clone https://github.com/NVIDIA/nccl-tests.git . && \
    make MPI=1 NCCL_HOME=/usr/lib/x86_64-linux-gnu CUDA_HOME=/usr/local/cuda-9.2 MPI_HOME=/mpi/ompi4-cuda DST_DIR=/ompi4tests/NCCL2 -j $(nproc) && \
    cd / && rm -fr /nccl-tests

ENV OSU_VERSION=5.5
# Download and install CUDA osu-micro-benchmarks
RUN cd /mpi && \
    wget http://mvapich.cse.ohio-state.edu/download/mvapich/osu-micro-benchmarks-$OSU_VERSION.tar.gz && \
    tar xf osu-micro-benchmarks-$OSU_VERSION.tar.gz && rm osu-micro-benchmarks-$OSU_VERSION.tar.gz -f && \
    cd osu-micro-benchmarks-$OSU_VERSION && \
    CC=/mpi/ompi4-cuda/bin/mpicc CXX=/mpi/ompi4-cuda/bin/mpicxx ./configure --prefix=/ompi4tests/osu --enable-cuda --with-cuda=/usr/local/cuda-9.2 && \
    make -j $(nproc) install && \
    cd / && rm -fr /mpi/osu-micro-benchmarks-$OSU_VERSION*

# For Singularity
RUN touch /usr/bin/nvidia-smi

# How to open a port
#EXPOSE 8888
