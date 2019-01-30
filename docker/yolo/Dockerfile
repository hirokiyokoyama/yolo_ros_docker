FROM tensorflow/tensorflow:latest-gpu

# install packages
RUN apt-get update && apt-get install -q -y \
    dirmngr \
    gnupg2 \
    lsb-release \
    && rm -rf /var/lib/apt/lists/*

# setup keys
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 421C365BD9FF1F717815A3895523BAEEB01FA116

# setup sources.list
RUN echo "deb http://packages.ros.org/ros/ubuntu `lsb_release -sc` main" > /etc/apt/sources.list.d/ros-latest.list

# install bootstrap tools
RUN apt-get update && apt-get install --no-install-recommends -y \
    python-rosdep \
    python-rosinstall \
    python-vcstools \
    && rm -rf /var/lib/apt/lists/*

# setup environment
ENV LANG C.UTF-8
ENV LC_ALL C.UTF-8

# bootstrap rosdep
RUN rosdep init 

# install ros packages
ENV ROS_DISTRO kinetic
RUN apt-get update && apt-get install -y \
    ros-kinetic-ros-core=1.3.2-0* \
    && rm -rf /var/lib/apt/lists/*

# create catkin workspace
RUN mkdir -p /catkin_ws/src
WORKDIR /catkin_ws/src
RUN . /opt/ros/$ROS_DISTRO/setup.sh && catkin_init_workspace \
    && cd .. && catkin_make

# setup yolo_tf
RUN git clone https://github.com/hirokiyokoyama/yolo_ros.git
WORKDIR /catkin_ws
RUN . devel/setup.sh && apt-get update && rosdep update \
    && rosdep install -y yolo_ros \
    && rm -rf /var/lib/apt/lists/*
RUN cd src/yolo_ros && git   pull
RUN . devel/setup.sh && catkin_make

# download data
RUN mkdir /root/.gdrive
COPY ./token_v2.json /root/.gdrive
COPY ./gdrive /usr/local/bin
WORKDIR /catkin_ws/src/yolo_ros
RUN gdrive download --recursive --force 1CHHccYks0Mgf2NGUDDFKIG_g6V8Il_QN

# setup entrypoint
COPY ./ros_entrypoint.sh /

ENTRYPOINT ["/ros_entrypoint.sh"]
CMD ["bash"]

