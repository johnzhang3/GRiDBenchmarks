# FROM julia
FROM ros:noetic-perception-focal
# fixed a key issue using the method mentioned in https://github.com/PRBonn/bonnetal/commit/0ab883b8356954c3e57a07a73c30bbac8f035a05
RUN sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list' && \
  apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654 && \
  apt update 
# install necessary dependencies 
RUN   if [ "x$(nproc)" = "x1" ] ; then export USE_PROC=1 ; \
      else export USE_PROC=4 ; fi && \
      apt-get update && apt-get install -y \
      vim \
      libatlas-base-dev \
      libeigen3-dev \
      libgoogle-glog-dev \
      libsuitesparse-dev \
      python3-catkin-tools python3-osrf-pycommon \
      python3-matplotlib \
      gfortran \
      autoconf \
      git \
      coinor-libipopt-dev \
      curl \
      wget \
      libopenmpi-dev \
      apt-utils \
      software-properties-common \
      build-essential \
      libssl-dev

RUN apt-get update && \
      apt-get -y install python3-pip\
      git \
      gcc \
      g++ \
      gdb \
      make \
      ninja-build \
      cmake \
      autoconf \
      automake \
      locales-all \
      dos2unix \
      rsync \
      tar \
      python3-rospkg \
      sudo\
  && apt-get clean

RUN sudo apt-get -y install clang-12 --install-suggests &&\
    sudo ln -s /usr/bin/clang-12 /usr/bin/clang &&\
    sudo ln -s /usr/bin/clang++-12 /usr/bin/clang++

# create a non-root user
ARG USERNAME=GRiDBenchmarks
ARG USER_UID=1000
ARG USER_GID=$USER_UID
# Create the user
RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME \
    #
    # [Optional] Add sudo support. Omit if you don't need to install software after connecting.
    && apt-get update \
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME
ENV workspaces = /home/${USERNAME}/GRiDBenchmarks


# create user
# RUN useradd -ms /bin/bash GRiDBenchmarks
USER GRiDBenchmarks
CMD /bin/bash
WORKDIR /home/GRiDBenchmarks

ENV downloads=/home/${USERNAME}/downloads
RUN mkdir -p $downloads
ENV main=/workspaces/${USERNAME}
# python dependencies
# WORKDIR $main
# RUN cd ${main} &&\
#     pip3 install -r GRiD/requirements.txt
# other dependencies
RUN sudo apt-get update &&\
    sudo apt-get -y install git curl build-essential libglib2.0-dev dkms xorg xorg-dev cpufrequtils net-tools meld apt-transport-https cmake libboost-all-dev liburdfdom-dev doxygen libgtest-dev

# TODO: cuda

# Clang/LLVM
RUN sudo bash -c "$(wget -O - https://apt.llvm.org/llvm.sh)"

# eigen cpu
WORKDIR $downloads
RUN cd ${downloads} &&\
    wget -q https://gitlab.com/libeigen/eigen/-/archive/3.3.9/eigen-3.3.9.tar.bz2 &&\
    tar -xf eigen-3.3.9.tar.bz2 &&\
    cd eigen-3.3.9 &&\
    mkdir build && cd build &&\
    cmake .. &&\
    sudo make install &&\
    sudo ln -s /usr/local/include/eigen3/Eigen /usr/local/include/Eigen &&\
    sudo ln -s /usr/local/include/eigen3/unsupported /usr/local/include/unsupported

# eigenpy
RUN sudo sh -c "echo 'deb [arch=amd64] http://robotpkg.openrobots.org/packages/debian/pub $(lsb_release -cs) robotpkg' >> /etc/apt/sources.list.d/robotpkg.list" &&\
    curl http://robotpkg.openrobots.org/packages/debian/robotpkg.key | sudo apt-key add - &&\
    sudo apt-get update &&\
    sudo apt-get install -y robotpkg-py38-eigenpy &&\
    export PATH=/opt/openrobots/bin:$PATH &&\
    export PKG_CONFIG_PATH=/opt/openrobots/lib/pkgconfig:$PKG_CONFIG_PATH &&\
    export LD_LIBRARY_PATH=/opt/openrobots/lib:$LD_LIBRARY_PATH &&\
    export PYTHONPATH=/opt/openrobots/lib/python3.8/site-packages:$PYTHONPATH &&\
    export CMAKE_PREFIX_PATH=/opt/openrobots:$CMAKE_PREFIX_PATH &&\
    export C_INCLUDE_PATH=/opt/openrobots/include:$C_INCLUDE_PATH &&\
    export CPLUS_INCLUDE_PATH=/opt/openrobots/include:$CPLUS_INCLUDE_PATH
    
# cpp ad
WORKDIR $downloads
RUN git clone https://github.com/coin-or/CppAD.git &&\
    cd ${downloads}/CppAD &&\
    git fetch && git checkout -q 83e249ec7819224138f35aaba564e2b977fb0078 &&\
    mkdir build && cd build &&\
    cmake .. &&\
    sudo make install 
# cpp ad-codegen 
WORKDIR $downloads
RUN git clone https://github.com/joaoleal/CppADCodeGen.git CppADCodeGen &&\
    cd ${downloads}/CppADCodeGen && mkdir build && cd build &&\
    cmake .. -DLLVM_VERSION=12 &&\
    sudo make install 

# pinocchio (this part is always tricky)
WORKDIR $downloads
RUN git clone --recursive https://github.com/stack-of-tasks/pinocchio &&\
    cd ${downloads}/pinocchio &&\
    # git checkout -b pinocchio3-preview origin/pinocchio3-preview &&\
    mkdir build && cd build &&\
    cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local -DBUILD_WITH_AUTODIFF_SUPPORT=ON -DBUILD_WITH_CODEGEN_SUPPORT=ON -DCMAKE_CXX_COMPILER=clang++-12 -DBUILD_PYTHON_INTERFACE=OFF &&\
    # cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local -DBUILD_WITH_AUTODIFF_SUPPORT=ON -DBUILD_WITH_CODEGEN_SUPPORT=ON -DCMAKE_CXX_COMPILER=clang++-12 -DPYTHON_EXECUTABLE=/usr/bin/python3 &&\
    make -j4 &&\
    sudo make install &&\
    export PATH=/usr/local/bin:$PATH &&\
    export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH &&\
    export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH &&\
    export PYTHONPATH=/usr/local/lib/python3.8/site-packages:$PYTHONPATH &&\
    export CMAKE_PREFIX_PATH=/usr/local:$CMAKE_PREFIX_PATH

ENV SHELL /bin/bash