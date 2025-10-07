FROM osrf/ros:noetic-desktop-full

##################
# parameters
##################

# Set IP of Franka control PC
ENV FCI_IP=172.16.0.2

##################
# libfranka build
##################
# Download and build the required franka libraries
RUN apt-get update && apt-get install -y \
    build-essential \
    cmake \
    git \
    libpoco-dev \
    libeigen3-dev \
    qt5-default \
    firefox \
    iputils-ping \
    net-tools \
    python3-pip \
    python3-catkin-tools \
    ros-noetic-ros-control \
    ros-noetic-ros-controllers \
    ros-noetic-controller-manager \
    ros-noetic-joint-state-controller \
    ros-noetic-effort-controllers \
    ros-noetic-position-controllers \
    ros-noetic-rviz \
    ros-noetic-boost-sml \
    bc \
    curl \
    debhelper \
    dpkg-dev \
    devscripts \
    fakeroot \
    libssl-dev \
    libelf-dev \
    bison \
    flex \
    cpio \
    kmod \
    rsync \
    libncurses-dev\
    && rm -rf /var/lib/apt/lists/*

RUN git clone --recursive https://github.com/frankaemika/libfranka --branch 0.9.2 
#only for FR3
WORKDIR /libfranka
RUN mkdir build
WORKDIR /libfranka/build
RUN cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_TESTS=OFF ..
RUN cmake --build .

# Make a Debian package and install it
RUN cpack -G DEB
RUN dpkg -i libfranka*.deb

##################
# franka_ros build
##################
SHELL ["/bin/bash", "-c"]
WORKDIR /catkin_ws
RUN mkdir -p src
RUN /bin/bash -c "source /opt/ros/noetic/setup.sh && catkin_init_workspace src"
RUN git clone --recursive https://github.com/frankaemika/franka_ros src/franka_ros
RUN /bin/bash -c "source /opt/ros/noetic/setup.sh && rosdep update && rosdep install --from-paths src --ignore-src --rosdistro noetic -y --skip-keys libfranka" || true
RUN /bin/bash -c "source /opt/ros/noetic/setup.sh && catkin_make -DCMAKE_BUILD_TYPE=Release -DFranka_DIR:PATH=/libfranka/build"
RUN /bin/bash -c "source /opt/ros/noetic/setup.sh && catkin_make -DCMAKE_BUILD_TYPE=Release -DFranka_DIR:PATH=/libfranka/build" || cat /catkin_ws/build/CMakeFiles/CMakeError.log || true
COPY cartesian_impedance_example_controller_no_marker.launch src/franka_ros/franka_example_controllers/launch/

# Add lines to the bashrc file that source ROS
RUN echo "source /ros_entrypoint.sh" >> /root/.bashrc
RUN echo "source /catkin_ws/devel/setup.bash" >> /root/.bashrc