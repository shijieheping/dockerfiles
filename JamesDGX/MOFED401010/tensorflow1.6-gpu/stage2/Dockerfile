FROM aaii/JamesDGX:MOFED401010_tensorflow1.6-gpu
LABEL maintainer "PengzhiZhu <ldqhyfzpz@qq.com>"
MAINTAINER PengzhiZhu <ldqhyfzpz@qq.com>

RUN ln -s /usr/local/cuda/lib64/stubs/libcuda.so /usr/local/cuda/lib64/stubs/libcuda.so.1 && \
    LD_LIBRARY_PATH=/usr/local/cuda/lib64/stubs:${LD_LIBRARY_PATH} \
    tensorflow/tools/ci_build/builds/configured GPU \
    bazel build -c opt --config=cuda \
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
RUN git clone https://github.com/tensorflow/benchmarks.git . && \
    git reset --hard 4b337c13b1d71c67a9097779a2d92a9e1cc7ba2a && \
    pip --no-cache-dir install \
        portpicker 

# For Singularity
RUN touch /usr/bin/nvidia-smi

# TensorBoard
EXPOSE 6006
# IPython
EXPOSE 8888
