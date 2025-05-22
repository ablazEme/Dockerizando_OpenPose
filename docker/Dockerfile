FROM nvidia/cuda:11.7.1-cudnn8-devel-ubuntu20.04

# Environment variables
ENV DEBIAN_FRONTEND=noninteractive
ENV NVIDIA_VISIBLE_DEVICES=all
ENV NVIDIA_DRIVER_CAPABILITIES=compute,utility,graphics
ENV CUDA_HOME=/usr/local/cuda
ENV LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH

# Install dependencies with pinned versions
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        git cmake g++ wget unzip \
        libopencv-dev libgoogle-glog-dev \
        libboost-all-dev libhdf5-dev \
        libatlas-base-dev python3-dev \
        python3-pip python3-setuptools \
        libprotobuf-dev protobuf-compiler \
        cmake-qt-gui ffmpeg && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Install Python packages separately
RUN pip3 install numpy opencv-python

# Build OpenPose with Python support
WORKDIR /usr/local
RUN set -xe && \
    git clone https://github.com/CMU-Perceptual-Computing-Lab/openpose && \
    cd /usr/local/openpose && \
    bash ./scripts/ubuntu/install_deps.sh

#RUN cd /usr/local/openpose && \
#    bash ./models/getModels.sh

#Copy models (if needed) (los caffeees)
COPY ./models /usr/local/openpose/models

RUN ls -la /usr/local/openpose/models

# Build with RTX 3050 optimizations
WORKDIR /usr/local/openpose/build
RUN cmake -DBUILD_PYTHON=ON \
    -DGPU_MODE=CUDA \
    -DUSE_CUDNN=OFF \
    -DCUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda \
    -DCUDNN_INCLUDE=/usr/include \
    -DCUDNN_LIBRARY=/usr/lib/x86_64-linux-gnu/libcudnn.so \
    -DBUILD_EXAMPLES=ON \
    -DBUILD_DOCS=OFF \
    -DPYTHON_EXECUTABLE=$(which python3) \
    -DPYTHON_INCLUDE_DIR=$(python3 -c "from distutils.sysconfig import get_python_inc; print(get_python_inc())") \
    -DPYTHON_LIBRARY=$(python3 -c "import distutils.sysconfig as sysconfig; print(sysconfig.get_config_var('LIBDIR') + '/' + sysconfig.get_config_var('INSTSONAME'))") .. && \
    make -j$(($(nproc)-1)) && \
    make install && \
    cp -r python/openpose/* /usr/local/openpose/python/openpose/
 
# Python setup
ENV PYTHONPATH=/usr/local/openpose/python:$PYTHONPATH
RUN python3 -c "import sys; sys.path.append('/usr/local/openpose/build/python'); from openpose import pyopenpose; print('OpenPose Python bindings loaded successfully')"

