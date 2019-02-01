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

ENV NCCL_VERSION 2.3.5

# Install NCCL2 v2.3.5-5
RUN apt-get purge -y --allow-change-held-packages \
	"*nccl*" && \
	apt-get update && apt-get install -y --no-install-recommends \
        git \
	devscripts && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
	git clone --branch=v$NCCL_VERSION-5 https://github.com/NVIDIA/nccl.git && \
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

# Download and install nccl-tests
RUN mkdir /nccl-tests -p && cd /nccl-tests && \
    git clone https://github.com/NVIDIA/nccl-tests.git . && \
    make MPI=1 NCCL_HOME=/usr/lib/x86_64-linux-gnu CUDA_HOME=/usr/local/cuda-10.0 MPI_HOME=/mpi/ompi4-cuda DST_DIR=/ompi4tests/NCCL2 -j $(nproc) && \
    cd / && rm -fr /nccl-tests

ENV OSU_VERSION=5.5
# Download and install CUDA osu-micro-benchmarks
RUN cd /mpi && \
    wget http://mvapich.cse.ohio-state.edu/download/mvapich/osu-micro-benchmarks-$OSU_VERSION.tar.gz && \
    tar xf osu-micro-benchmarks-$OSU_VERSION.tar.gz && rm osu-micro-benchmarks-$OSU_VERSION.tar.gz -f && \
    cd osu-micro-benchmarks-$OSU_VERSION && \
    CC=/mpi/ompi4-cuda/bin/mpicc CXX=/mpi/ompi4-cuda/bin/mpicxx ./configure --prefix=/ompi4tests/osu --enable-cuda --with-cuda=/usr/local/cuda-10.0 && \
    make -j $(nproc) install && \
    cd / && rm -fr /mpi/osu-micro-benchmarks-$OSU_VERSION*

RUN apt-get update && apt-get install -y --no-install-recommends \
        build-essential \
        cuda-command-line-tools-9-0 \
        curl \
        git \
        libcurl3-dev \
        libfreetype6-dev \
        libhdf5-serial-dev \
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
    find /usr/local/cuda-10.0/lib64/ -type f -name 'lib*_static.a' -not -name 'libcudart_static.a' -delete && \
    rm /usr/lib/x86_64-linux-gnu/libcudnn_static_v7.a

RUN apt-get update && \
        apt-get install nvinfer-runtime-trt-repo-ubuntu1604-5.0.0-rc-cuda10.0 && \
        apt-get update && \
        apt-get install libnvinfer5=5.0.0-1+cuda10.0 && \
        apt-get install libnvinfer-dev=5.0.0-1+cuda10.0

# Link NCCL libray and header where the build script expects them.
RUN mkdir /usr/local/cuda-10.0/lib &&  \
    ln -s /usr/lib/x86_64-linux-gnu/libnccl.so.2 /usr/local/cuda/lib/libnccl.so.2 && \
    ln -s /usr/include/nccl.h /usr/local/cuda/include/nccl.h

RUN curl -fSsL -O https://bootstrap.pypa.io/get-pip.py && \
    python get-pip.py && \
    rm get-pip.py

RUN pip --no-cache-dir install \
        Pillow \
        h5py \
        ipykernel \
        jupyter \
        keras_applications \
        keras_preprocessing \
        matplotlib \
        mock \
        numpy \
        scipy \
        sklearn \
        pandas \
        && \
    python -m ipykernel.kernelspec

# RUN ln -s -f /usr/bin/python3 /usr/bin/python#

# Set up our notebook config.
#COPY jupyter_notebook_config.py /root/.jupyter/

# Set up Bazel.

# Running bazel inside a `docker build` command causes trouble, cf:
#   https://github.com/bazelbuild/bazel/issues/134
# The easiest solution is to set up a bazelrc file forcing --batch.
RUN echo "startup --batch" >>/etc/bazel.bazelrc
# Similarly, we need to workaround sandboxing issues:
#   https://github.com/bazelbuild/bazel/issues/418
RUN echo "build --spawn_strategy=standalone --genrule_strategy=standalone" \
    >>/etc/bazel.bazelrc
# Install the most recent bazel release.
ENV BAZEL_VERSION 0.15.0
WORKDIR /
RUN mkdir /bazel && \
    cd /bazel && \
    curl -H "User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.133 Safari/537.36" -fSsL -O https://github.com/bazelbuild/bazel/releases/download/$BAZEL_VERSION/bazel-$BAZEL_VERSION-installer-linux-x86_64.sh && \
    curl -H "User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.133 Safari/537.36" -fSsL -o /bazel/LICENSE.txt https://raw.githubusercontent.com/bazelbuild/bazel/master/LICENSE && \
    chmod +x bazel-*.sh && \
    ./bazel-$BAZEL_VERSION-installer-linux-x86_64.sh && \
    cd / && \
    rm -f /bazel/bazel-$BAZEL_VERSION-installer-linux-x86_64.sh

# Download and build TensorFlow.
WORKDIR /tensorflow
RUN git clone --branch=r1.12 --depth=1 https://github.com/tensorflow/tensorflow.git .

# Configure the build for our CUDA configuration.
ENV CI_BUILD_PYTHON python
ENV LD_LIBRARY_PATH /usr/local/cuda/extras/CUPTI/lib64:$LD_LIBRARY_PATH
ENV TF_NEED_CUDA 1
ENV TF_NEED_TENSORRT 1
ENV TF_CUDA_COMPUTE_CAPABILITIES=7.0
ENV TF_CUDA_VERSION=10.0
ENV TF_CUDNN_VERSION=7
ENV TF_NCCL_VERSION=2

ENV TF_NEED_GCP 0
ENV TF_NEED_HDFS 0
ENV TF_NEED_S3 0
ENV TF_NEED_KAFKA 0
ENV TF_NEED_MPI 0
#ENV TF_CUDA_CLANG 0
#ENV TF_NEED_OPENCL_SYCL 0
#ENV TF_NEED_JEMALLOC 0
#ENV TF_NEED_IGNITE 0
#ENV TF_ENABLE_XLA 0
ENV TF_NEED_GDR 0
ENV TF_NEED_VERBS 0
ENV NCCL_INSTALL_PATH /usr/lib/x86_64-linux-gnu
ENV CUDNN_INSTALL_PATH /usr/lib/x86_64-linux-gnu
#ENV CC_OPT_FLAGS '-march=native'

RUN ln -s /usr/local/cuda/lib64/stubs/libcuda.so /usr/local/cuda/lib64/stubs/libcuda.so.1 && \
    LD_LIBRARY_PATH=/usr/local/cuda/lib64/stubs:${LD_LIBRARY_PATH} \
    tensorflow/tools/ci_build/builds/configured GPU \
    bazel build -c opt \
        --config="opt" \
	--config=cuda \
	--copt=-mavx \
        --copt="-march=broadwell" \
        --copt="-march=native" \
        --copt="-O3" \
	--cxxopt="-D_GLIBCXX_USE_CXX11_ABI=0" \
        tensorflow/tools/pip_package:build_pip_package && \
    rm /usr/local/cuda/lib64/stubs/libcuda.so.1 && \
    bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/pip && \
    pip --no-cache-dir install --upgrade /tmp/pip/tensorflow-*.whl && \
    rm -rf /tmp/pip && \
    rm -rf /root/.cache
# Clean up pip wheel and Bazel cache when done.
#	--config=gdr \
#	--config=verbs \
#       --copt="-march=avx2" \

# Benchmarks
#RUN git clone https://github.com/tensorflow/benchmarks.git . && \
#    git reset --hard 4b337c13b1d71c67a9097779a2d92a9e1cc7ba2a && \
#RUN git clone --origin=cnn_tf_v1.12_compatible https://github.com/tensorflow/benchmarks.git . && \
#    git reset --hard cnn_tf_v1.12_compatible
WORKDIR /benchmarks
RUN git clone https://github.com/tensorflow/benchmarks.git . && \
    git reset --hard 1c8a3490c0e8351f49101a987bf9feba7212faf5

RUN ln -s /usr/local/cuda/lib64/stubs/libcuda.so /usr/local/cuda/lib64/stubs/libcuda.so.1 && \
	PATH=$PATH:"/mpi/ompi4-cuda/bin" \
	LD_LIBRARY_PATH=/usr/local/cuda/lib64/stubs:${LD_LIBRARY_PATH} \
	HOROVOD_GPU_ALLREDUCE=NCCL \
	HOROVOD_NCCL_HOME=/usr/lib/x86_64-linux-gnu \
	pip install --no-cache-dir horovod && \
	rm /usr/local/cuda/lib64/stubs/libcuda.so.1
	
ENV LD_LIBRARY_PATH /usr/lib/x86_64-linux-gnu:$LD_LIBRARY_PATH

# For Singularity
RUN touch /usr/bin/nvidia-smi

# How to open a port
#EXPOSE 8888
