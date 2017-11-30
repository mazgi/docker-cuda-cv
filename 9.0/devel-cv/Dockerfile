FROM nvidia/cuda:9.0-cudnn7-devel-ubuntu16.04

ENV DOCKERBUILD_OPENCV_VERSION=3.3.0
ENV DOCKERBUILD_OPENMPI_VERSION=3.0.0
ENV DOCKERBUILD_CHAINER_VERSION=4.0.0b1
ENV DOCKERBUILD_CUPY_VERSION=4.0.0b1

RUN : 'Install basic packages' \
  && apt update && apt install -y --no-install-recommends curl software-properties-common pkg-config \
  && : 'Install python3 packages' \
  && apt install -y --no-install-recommends python3-dev python3-pip \
  && pip3 install --upgrade pip && pip3 install setuptools cython \
  && : 'Install NVIDIA packages' \
  && apt install -y --no-install-recommends libnccl-dev libcudnn7 \
  && : 'Install other development packages' \
  && apt install -y --no-install-recommends cmake autoconf automake libtool flex \
  && : 'Install OpenSSH server for Open MPI' \
  && apt-get install -y --no-install-recommends openssh-server \
  && mkdir /var/run/sshd \
  && sed -i 's/Port 22/Port 10022/' /etc/ssh/sshd_config \
  && mkdir /root/.ssh && chmod 700 /root/.ssh \
  && echo 'Host *' > /root/.ssh/config \
  && echo '  Port 10022' >> /root/.ssh/config \
  && echo '  StrictHostKeyChecking no' >> /root/.ssh/config \
  && echo '  UserKnownHostsFile /dev/null' >> /root/.ssh/config \
  && : 'Install ffmpeg' \
  && add-apt-repository -y ppa:jonathonf/ffmpeg-3 && apt update && apt install -y --no-install-recommends ffmpeg \
  && : 'Clean up' \
  && apt clean

RUN : 'OpenCV: Install numpy before build' \
  && pip3 install numpy \
  && : 'OpenCV: Download sources' \
  && curl -L -o /opencv.tar.gz https://github.com/opencv/opencv/archive/${DOCKERBUILD_OPENCV_VERSION}.tar.gz \
  && tar xf /opencv.tar.gz -C / && mv /opencv-${DOCKERBUILD_OPENCV_VERSION} /opencv \
  && curl -L -o /opencv_contrib.tar.gz https://github.com/opencv/opencv_contrib/archive/${DOCKERBUILD_OPENCV_VERSION}.tar.gz \
  && tar xf /opencv_contrib.tar.gz -C / && mv /opencv_contrib-${DOCKERBUILD_OPENCV_VERSION} /opencv_contrib \
  && : 'OpenCV: make && install' \
  && mkdir /opencv/build && cd /opencv/build && cmake \
  -D CMAKE_BUILD_TYPE=RELEASE \
  -D CMAKE_INSTALL_PREFIX=/usr/local \
  -D OPENCV_EXTRA_MODULES_PATH=/opencv_contrib/modules \
  -D WITH_CUDA=OFF \
  -D BUILD_EXAMPLES=OFF \
  -D INSTALL_PYTHON_EXAMPLES=OFF \
  .. \
  && cd /opencv/build && make -j$(nproc) && make install && ldconfig \
  && : 'Install Open MPI for ChainerMN' \
  && : 'see: https://chainermn.readthedocs.io/en/latest/installation/guide.html' \
  && curl -L -o /openmpi.tar.gz https://github.com/open-mpi/ompi/archive/v${DOCKERBUILD_OPENMPI_VERSION}.tar.gz \
  && tar xf /openmpi.tar.gz -C / && mv /ompi-${DOCKERBUILD_OPENMPI_VERSION} /openmpi \
  && cd /openmpi && ./autogen.pl && ./configure --with-cuda \
  && cd /openmpi && make -j$(nproc) && make install && ldconfig \
  && : 'Clean up' \
  && rm -rf /opencv.tar.gz /opencv /opencv_contrib.tar.gz /opencv_contrib /openmpi.tar.gz /openmpi \
  && : 'Create symlink' \
  && cd /usr/local/cuda/lib64/stubs/ && ln -s libcuda.so libcuda.so.1

ENV CUDA_PATH=/usr/local/cuda
ENV PATH=$CUDA_PATH/bin:$PATH
ENV CPATH=$CUDA_PATH/include:/usr/local/include:$CPATH
ENV LD_LIBRARY_PATH=$CUDA_PATH/lib64:$CUDA_PATH/lib64/stubs:/usr/local/lib:$LD_LIBRARY_PATH

RUN : 'Install Chainer' \
  && pip3 install cupy==${DOCKERBUILD_CUPY_VERSION} chainer==${DOCKERBUILD_CHAINER_VERSION} chainermn

# for sshd
EXPOSE 10022
