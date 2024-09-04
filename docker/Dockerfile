# syntax=docker/dockerfile:1
FROM osrf/ros:melodic-desktop-full

ENV DOCKER_VERSION=26.0.0
ENV DIRPATH=/voxgraph_ws
SHELL ["/bin/bash", "-c"]

# Install debian packages
RUN apt update \
&& DEBIAN_FRONTEND=noninteractive apt install -y \
    ros-melodic-catkin ros-melodic-cmake-modules \
    python-catkin-tools python-wstool \
    autoconf libtool nano openssh-client protobuf-compiler rsync wget \
    && rm -rf /var/lib/apt/lists/*

RUN rosdep update

# Create known_hosts and add keys for cloning repositories
RUN mkdir -p -m 0600 ~/.ssh && ssh-keyscan github.com >> ~/.ssh/known_hosts

# Build voxblox
WORKDIR ${DIRPATH}
RUN mkdir -p ${DIRPATH}/src
RUN cd ${DIRPATH} && \
    catkin init && \
    catkin config --extend /opt/ros/melodic && \ 
    catkin config --cmake-args -DCMAKE_BUILD_TYPE=Release && \ 
    catkin config --merge-devel
RUN --mount=type=ssh cd ${DIRPATH}/src && \
    git clone git@github.com:ethz-asl/voxblox.git && \
    wstool init . ./voxblox/voxblox_https.rosinstall && \
    wstool update
RUN cd ${DIRPATH}/src && catkin build voxblox_ros

# Build voxgraph
RUN --mount=type=ssh cd ${DIRPATH}/src && \
    git clone git@github.com:luacghee/voxgraph.git && \
    wstool merge ./voxgraph/voxgraph_ssh.rosinstall && \
    wstool update
RUN cd ${DIRPATH}/src && catkin build voxgraph

# Build lidar undistortion for voxgraph demo
RUN --mount=type=ssh cd ${DIRPATH}/src && \
    wstool merge ./voxgraph/arche_ssh.rosinstall && \
    wstool update

# Add in corrected files for ouster_lidar compilation. See ethz-asl/voxgraph/issues #74 and #76
COPY ./ol_corrections/*.h ${DIRPATH}/src/ouster_lidar/ouster_ros/include/ouster_ros
RUN cd ${DIRPATH}/src && catkin build lidar_undistortion

COPY ./entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]