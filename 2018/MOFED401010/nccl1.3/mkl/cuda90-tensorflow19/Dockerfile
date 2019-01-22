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

# Why golang openjdk-8-jdk openjdk-8-jre-headless ?
#	golang \
#       openjdk-8-jdk \
#       openjdk-8-jre-headless \
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

#Why Pillow and h5py? For jupyter?
#Mock is needed by r1.8 and beyond
RUN pip --no-cache-dir install \
        ipykernel \
        jupyter \
        matplotlib \
        numpy \
        scipy \
        sklearn \
        pandas \
	mock \
	wheel \
        && \
    python -m ipykernel.kernelspec

# RUN ln -s -f /usr/bin/python3 /usr/bin/python#

# Set up our notebook config.
#COPY jupyter_notebook_config.py /root/.jupyter/	no need

# Jupyter has issues with being run directly:
#   https://github.com/ipython/ipython/issues/7062
# We just add a little wrapper script.
#COPY run_jupyter.sh /	no need

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
# Why curl rather than wget? For in-docker build?
ENV BAZEL_VERSION 0.11.0
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
RUN git clone --branch=r1.9 --depth=1 https://github.com/tensorflow/tensorflow.git .

# Configure the build for our CUDA configuration.
ENV CI_BUILD_PYTHON python
ENV LD_LIBRARY_PATH /usr/local/cuda/extras/CUPTI/lib64:$LD_LIBRARY_PATH
ENV TF_NEED_CUDA 1
ENV TF_NEED_JEMALLOC 0
ENV TF_NEED_GCP 0
ENV TF_NEED_HDFS 0
ENV TF_NEED_S3 0
ENV TF_NEED_KAFKA 0
ENV TF_ENABLE_XLA 0
ENV TF_NEED_GDR 1
ENV TF_NEED_VERBS 1
ENV TF_NEED_MPI 0
ENV TF_CUDA_COMPUTE_CAPABILITIES 3.5,6.0,7.0
ENV TF_CUDA_VERSION 9.0
ENV TF_CUDNN_VERSION 7
ENV CUDNN_INSTALL_PATH /usr/lib/x86_64-linux-gnu
ENV CC_OPT_FLAGS '-march=native'

RUN ln -s /usr/local/cuda/lib64/stubs/libcuda.so /usr/local/cuda/lib64/stubs/libcuda.so.1 && \
    LD_LIBRARY_PATH=/usr/local/cuda/lib64/stubs:${LD_LIBRARY_PATH} \
    tensorflow/tools/ci_build/builds/configured GPU \
    bazel build -c opt \
		--config=cuda \
		--config=mkl \
                --config="opt" \
                --copt="-march=broadwell" \
                --copt="-O3" \
		--cxxopt="-D_GLIBCXX_USE_CXX11_ABI=0" \
	        tensorflow/tools/pip_package:build_pip_package && \
    mkdir /pip_pkg && \
    bazel-bin/tensorflow/tools/pip_package/build_pip_package /pip_pkg && \
    pip --no-cache-dir install --upgrade /pip_pkg/tensorflow-*.whl && \
    rm -rf /pip_pkg && \
    rm -rf /root/.cache
# Clean up pip wheel and Bazel cache when done.

WORKDIR /root

# Benchmarks
#RUN git clone https://github.com/tensorflow/benchmarks.git . && \
#    git reset --hard 4b337c13b1d71c67a9097779a2d92a9e1cc7ba2a && \
WORKDIR /benchmarks
RUN git clone --origin=3b90c14fb2bf02ca5d27c188aee878663229a0a7 https://github.com/tensorflow/benchmarks.git . && \
    git reset --hard 3b90c14fb2bf02ca5d27c188aee878663229a0a7 && \
    pip --no-cache-dir install \
        portpicker 

# For Singularity
RUN touch /usr/bin/nvidia-smi

# TensorBoard
EXPOSE 6006
# IPython
EXPOSE 8888
