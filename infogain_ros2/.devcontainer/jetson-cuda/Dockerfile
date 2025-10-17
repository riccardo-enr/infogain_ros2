# =============================================================================
# === Base Image Setup
# =============================================================================
ARG DEVICE_TYPE=cuda

# ARM64 Ubuntu 24.04 base for Jetson simulation with ROS 2 Jazzy
FROM arm64v8/ubuntu:24.04 AS base-jetson

ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    locales \
    curl \
    gnupg2 \
    lsb-release \
    ca-certificates && \
    locale-gen en_US en_US.UTF-8 && \
    update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8 && \
    curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/ros2.list > /dev/null && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
    ros-jazzy-ros-base \
    ros-dev-tools && \
    rm -rf /var/lib/apt/lists/*

# CUDA base image for x86 workstation development
FROM althack/ros2:jazzy-cuda-full AS base-cuda

# Select base based on DEVICE_TYPE argument
FROM base-${DEVICE_TYPE} AS base

ARG TARGET_PLATFORM="desktop"

# Create ros user, grant sudo, create /workspaces
RUN getent group ros || groupadd -g 1000 ros && \
    getent passwd ros || useradd -m -u 1000 -g ros -s /bin/bash ros && \
    echo "ros ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers && \
    mkdir -p /workspaces /home/ros && \
    chown -R 1000:1000 /workspaces /home/ros && \
    touch /home/ros/.bash_history && \
    chown 1000:1000 /home/ros/.bash_history

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    build-essential \
    cmake \
    ffmpeg \
    git \
    libboost-program-options-dev \
    libeigen3-dev \
    liboctomap-dev \
    libusb-1.0-0-dev \
    libyaml-cpp-dev \
    python3-colcon-common-extensions \
    python3-setuptools \
    python3-tk \
    python3-pip \
    python3-venv \
    ros-jazzy-octomap-msgs \
    ros-jazzy-octomap-rviz-plugins \
    ros-jazzy-octomap-server \
    ros-jazzy-plotjuggler-ros \
    ros-jazzy-teleop-twist-keyboard \
    ros-jazzy-tf-transformations \
    sudo \
    tree \
    wget \
    ccache \
    x11-xserver-utils \
    xclip \
    wl-clipboard \
    ca-certificates \
    curl && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /workspaces

ENV HISTFILE=/home/ros/.bash_history \
    HISTSIZE=1000 \
    HISTFILESIZE=2000 \
    PROMPT_COMMAND='history -a' \
    LANG=en_US.UTF-8 \
    LC_ALL=en_US.UTF-8

# =============================================================================
# === Final Development Image
# =============================================================================
FROM base AS final
WORKDIR /workspaces

# Install dev tools, Node.js, and LSPs
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash - && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
    clangd \
    nodejs \
    htop \
    tmux && \
    rm -rf /var/lib/apt/lists/* && \
    npm install -g pyright typescript-language-server tree-sitter-cli

# Install Python stack (including JAX for MPPI)
RUN pip3 install --break-system-packages \
    numpy==1.26.4 \
    scipy==1.14.1 \
    pandas \
    seaborn \
    matplotlib

# Switch to non-root user
USER ros
RUN git config --global user.name "Riccardo Enrico" && \
    git config --global user.email "riccardo.enrico97@gmail.com" && \
    echo "\n# Sourcing ROS 2" >> /home/ros/.bashrc && \
    echo "source /opt/ros/jazzy/setup.bash" >> /home/ros/.bashrc && \
    echo "\n# Sourcing Local Workspace" >> /home/ros/.bashrc && \
    echo "if [ -f /workspaces/infogain_ros2/ros2_ws/install/setup.bash ]; then source /workspaces/infogain_ros2/ros2_ws/install/setup.bash; fi" >> /home/ros/.bashrc && \
    echo "\n# Environment Variables" >> /home/ros/.bashrc && \
    echo "export PATH=/usr/local/bin:\${PATH}" >> /home/ros/.bashrc && \
    echo "\n# Ccache" >> /home/ros/.bashrc && \
    echo "export CC=\"/usr/lib/ccache/gcc\"" >> /home/ros/.bashrc && \
    echo "export CXX=\"/usr/lib/ccache/g++\"" >> /home/ros/.bashrc

CMD ["/bin/bash"]
