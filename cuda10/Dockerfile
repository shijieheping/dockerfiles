FROM nvidia/cuda:10.0-cudnn7-devel-ubuntu16.04
LABEL maintainer "PengzhiZhu <ldqhyfzpz@qq.com>"
MAINTAINER PengzhiZhu <ldqhyfzpz@qq.com>

WORKDIR /

RUN apt-get update && apt-get install -y --no-install-recommends \
        wget curl python-dev unzip git \
        && \
    rm -rf /var/lib/apt/lists/* && \
    find /usr/local/cuda-10.0/lib64/ -type f -name 'lib*_static.a' -not -name 'libcudart_static.a' -delete && \
    rm /usr/lib/x86_64-linux-gnu/libcudnn_static_v7.a

RUN apt-get update && \
        apt-get install nvinfer-runtime-trt-repo-ubuntu1604-5.0.0-rc-cuda10.0 && \
        apt-get update && \
        apt-get install libnvinfer-dev libnvinfer5 && \
    rm -rf /var/lib/apt/lists/*

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
        keras_applications==1.0.5 \
        keras_preprocessing==1.0.3 \
        matplotlib \
        mock \
        numpy \
        scipy \
        sklearn \
        pandas \
        && \
    python -m ipykernel.kernelspec

# Set up Bazel.
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
RUN git clone --branch=r1.11 --depth=1 https://github.com/tensorflow/tensorflow.git .

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
ENV TF_NEED_GDR 0
ENV TF_NEED_VERBS 0
ENV TF_NEED_MPI 0
ENV TF_CUDNN_VERSION 7
ENV TF_NEED_OPENCL_SYCL 0
ENV TF_CUDA_CLANG 0
ENV TF_NCCL_VERSION 2
ENV CUDNN_INSTALL_PATH /usr/lib/x86_64-linux-gnu
ENV CC_OPT_FLAGS '-march=native'

ENV TF_NEED_TENSORRT 1
ENV TF_CUDA_COMPUTE_CAPABILITIES=6.1,7.5
ENV TF_CUDA_VERSION=10.0


RUN ln -s /usr/local/cuda/lib64/stubs/libcuda.so /usr/local/cuda/lib64/stubs/libcuda.so.1 && \
    LD_LIBRARY_PATH=/usr/local/cuda/lib64/stubs:${LD_LIBRARY_PATH} \
    tensorflow/tools/ci_build/builds/configured GPU \
    bazel build -c opt \
	--copt=-mavx --config=cuda \
	--config="opt" \
	--copt="-march=broadwell" \
        --copt="-O3" \
	--cxxopt="-D_GLIBCXX_USE_CXX11_ABI=0" \
        tensorflow/tools/pip_package:build_pip_package && \
    rm /usr/local/cuda/lib64/stubs/libcuda.so.1 && \
    bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/pip && \
    pip --no-cache-dir install --upgrade /tmp/pip/tensorflow-*.whl && \
    rm -rf /tmp/pip && \
    rm -rf /root/.cache
# Clean up pip wheel and Bazel cache when done.

# Benchmarks
WORKDIR /benchmarks
#RUN git clone --origin=3b90c14fb2bf02ca5d27c188aee878663229a0a7 https://github.com/tensorflow/benchmarks.git . && \
#    git reset --hard 3b90c14fb2bf02ca5d27c188aee878663229a0a7 && \
RUN git clone https://github.com/tensorflow/benchmarks.git . && \
    git reset --hard 4c7b09ad87bbfc4b1f89650bcee40b3fc5e7dfed && \
    pip --no-cache-dir install \
        portpicker 

# For Singularity
RUN touch /usr/bin/nvidia-smi

# TensorBoard
EXPOSE 6006
# IPython
EXPOSE 8888
