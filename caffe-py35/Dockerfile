FROM nvidia/cuda:8.0-cudnn6-devel-ubuntu16.04
LABEL maintainer zhaostu4@gmail.com

RUN mkdir /root/.pip/ && echo "[global]">>/root/.pip/pip.conf && echo "index-url = https://pypi.tuna.tsinghua.edu.cn/simple">>/root/.pip/pip.conf

RUN apt-get update && \
        apt-get install -y --no-install-recommends \
        devscripts \
	dh-make \
	linux-headers-generic\
	fakeroot \
	build-essential \
        cmake \
        git \
        wget \
        libatlas-base-dev \
        libboost-all-dev \
        libgflags-dev \
        libgoogle-glog-dev \
        libhdf5-serial-dev \
        libleveldb-dev \
        liblmdb-dev \
        libopencv-dev \
        libprotobuf-dev \
        libsnappy-dev \
        protobuf-compiler \
        python3-dev \
        python-numpy \
        python3-pip \
        python3-setuptools \
        python-scipy \
	openssh-server && \
    rm -rf /var/lib/apt/lists/*

ENV CAFFE_ROOT=/opt/caffe
WORKDIR $CAFFE_ROOT

# FIXME: use ARG instead of ENV once DockerHub supports this
# https://github.com/docker/hub-feedback/issues/460
ENV CLONE_TAG=1.0

# git nccl and caffe
RUN git clone -b ${CLONE_TAG} --depth 1 https://github.com/BVLC/caffe.git . && git clone https://github.com/NVIDIA/nccl.git  
# nccl compile
RUN cd nccl && \
    make -j24 src.build && make -j24 pkg.debian.build && \
    cd build/pkg/deb && dpkg -i libnccl2_2.3.7-1+cuda8.0_amd64.deb && dpkg -i libnccl-dev_2.3.7-1+cuda8.0_amd64.deb && \
    cd $CAFFE_ROOT && rm -rf nccl

# caffe requirements and build!
RUN pip3 install --upgrade pip && hash -r pip && \
    cd python && for req in $(cat requirements.txt) pydot 'python-dateutil>2'; do pip3 install $req; done && cd .. && \
    mkdir build && cd build && \
    cmake -DUSE_CUDNN=1 -DUSE_NCCL=1 -Dpython_version=3 .. && \
    make -j"$(nproc)"

ENV PYCAFFE_ROOT $CAFFE_ROOT/python
ENV PYTHONPATH $PYCAFFE_ROOT:$PYTHONPATH
ENV PATH $CAFFE_ROOT/build/tools:$PYCAFFE_ROOT:$PATH
RUN echo "$CAFFE_ROOT/build/lib" >> /etc/ld.so.conf.d/caffe.conf && ldconfig

# set ssh server config
RUN sed -i 's/UsePAM yes/UsePAM no/g' /etc/ssh/sshd_config &&\
	sed -ri 's/^PermitRootLogin\s+.*/PermitRootLogin yes/' /etc/ssh/sshd_config &&\
	mkdir /var/run/sshd && \
	echo "root:zhao"|chpasswd

EXPOSE 22
# set run images shell
CMD ["/usr/sbin/sshd","-D"]
