FROM nvidia/cuda:9.0-cudnn7-devel-ubuntu16.04

ENV DOCKERBUILD_OPENCV_VERSION=3.3.0

RUN apt update
# Install basic packages
RUN apt install -y --no-install-recommends curl software-properties-common
# Install python3 packages
RUN apt install -y --no-install-recommends python3-dev python3-pip
# Install other development packages
RUN apt install -y --no-install-recommends cmake

# Install ffmpeg
RUN add-apt-repository -y ppa:jonathonf/ffmpeg-3 && apt update && apt install -y --no-install-recommends ffmpeg

# OpenCV: Download sources
RUN curl -L -o /opencv.tar.gz https://github.com/opencv/opencv/archive/${DOCKERBUILD_OPENCV_VERSION}.tar.gz \
      && tar xf /opencv.tar.gz && mv /opencv-${DOCKERBUILD_OPENCV_VERSION} /opencv
RUN curl -L -o /opencv_contrib.tar.gz https://github.com/opencv/opencv_contrib/archive/${DOCKERBUILD_OPENCV_VERSION}.tar.gz \
      && tar xf /opencv_contrib.tar.gz && mv /opencv_contrib-${DOCKERBUILD_OPENCV_VERSION} /opencv_contrib
# OpenCV: Install numpy before build
RUN pip3 install numpy
# OpenCV: make && install
RUN mkdir /opencv/build && cd /opencv/build && cmake \
      -D CMAKE_BUILD_TYPE=RELEASE \
      -D CMAKE_INSTALL_PREFIX=/usr/local \
      -D OPENCV_EXTRA_MODULES_PATH=/opencv_contrib/modules \
      -D WITH_CUDA=OFF \
      -D BUILD_EXAMPLES=OFF \
      -D INSTALL_PYTHON_EXAMPLES=OFF \
      ..
RUN cd /opencv/build && make -j$(nproc) && make install && ldconfig

# Clean up
RUN apt clean
RUN rm -rf opencv.tar.gz opencv/ opencv_contrib.tar.gz opencv_contrib/
