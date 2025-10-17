# ==============================================================================
# === Base Image Setup
# ==============================================================================
ARG DEVICE_TYPE=cpu
FROM althack/ros2:jazzy-full AS base-cpu

FROM base-cpu AS base

ARG TARGET_PLATFORM="desktop"

# Create ros user (UID/GID 1000), grant sudo, and create /workspaces
RUN getent group ros || groupadd -g 1000 ros && \
    getent passwd ros || useradd -m -u 1000 -g ros -s /bin/bash ros && \
    echo "ros ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers && \
    mkdir -p /workspaces && chown 1000:1000 /workspaces

WORKDIR /workspaces

# Set up bash history
COPY --chown=1000:1000 ./.devcontainer/.bash_history /home/ros/.bash_history
ENV HISTFILE=/home/ros/.bash_history \
    HISTSIZE=1000 \
    HISTFILESIZE=2000 \
    PROMPT_COMMAND='history -a'

# Refresh ROS 2 apt sources using the modern ros2-apt-source package
RUN apt-get update && apt-get install -y curl && \
    export ROS_APT_SOURCE_VERSION=$(curl -s https://api.github.com/repos/ros-infrastructure/ros-apt-source/releases/latest | grep -F "tag_name" | awk -F\" '{print $4}') && \
    curl -L -o /tmp/ros2-apt-source.deb "https://github.com/ros-infrastructure/ros-apt-source/releases/download/${ROS_APT_SOURCE_VERSION}/ros2-apt-source_${ROS_APT_SOURCE_VERSION}.$(. /etc/os-release && echo ${UBUNTU_CODENAME:-${VERSION_CODENAME}})_all.deb" && \
    dpkg -i /tmp/ros2-apt-source.deb && \
    rm /tmp/ros2-apt-source.deb && \
    apt-get update

# Install common OS dependencies required by all stages
RUN apt-get update && apt-get install -y --no-install-recommends \
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
    ros-jazzy-motion-capture-tracking \
    ros-jazzy-octomap-msgs \
    ros-jazzy-octomap-rviz-plugins \
    ros-jazzy-octomap-server \
    ros-jazzy-plotjuggler-ros \
    ros-jazzy-ros-gz \
    ros-jazzy-ros-gz-bridge \
    ros-jazzy-teleop-twist-keyboard \
    ros-jazzy-tf-transformations \
    sudo \
    tree \
    wget \
    ccache \
    x11-xserver-utils \
    xclip \
    wl-clipboard && \
    rm -rf /var/lib/apt/lists/*

# Allow pip to install packages globally
RUN python3 -m pip config set global.break-system-packages true


# ==============================================================================
# === Builder Stages (for compiling external tools)
# ==============================================================================

# --- PX4 builder ---
FROM base AS px4-builder
WORKDIR /workspaces
RUN git clone https://github.com/PX4/PX4-Autopilot.git --branch=v1.16.0 --recursive && \
    mkdir -p PX4-Autopilot/src/modules/uxrce_dds_client/ && \
    mkdir -p PX4-Autopilot/Tools/simulation/gz/models/ && \
    mkdir -p PX4-Autopilot/Tools/simulation/gz/worlds/ && \
    mkdir -p PX4-Autopilot/Tools/simulation/gz/data/ && \
    mkdir -p PX4-Autopilot/ROMFS/px4fmu_common/init.d-posix/airframes/

COPY ./px4-gazebo-models/dds_topics.yaml PX4-Autopilot/src/modules/uxrce_dds_client/dds_topics.yaml
COPY ./px4-gazebo-models/CMakeLists.txt PX4-Autopilot/ROMFS/px4fmu_common/init.d-posix/airframes/
COPY ./px4-gazebo-models/models/x500_custom/ PX4-Autopilot/Tools/simulation/gz/models/x500_custom/
COPY ./px4-gazebo-models/models/x500_depth_2/ PX4-Autopilot/Tools/simulation/gz/models/x500_depth_2/
COPY ./px4-gazebo-models/models/x500_custom/4150_gz_x500_custom PX4-Autopilot/ROMFS/px4fmu_common/init.d-posix/airframes/
COPY ./px4-gazebo-models/models/x500_depth/4002_gz_x500_depth PX4-Autopilot/ROMFS/px4fmu_common/init.d-posix/airframes/4002_gz_x500_depth

RUN bash ./PX4-Autopilot/Tools/setup/ubuntu.sh --no-nuttx

# --- Micro XRCE-DDS builder ---
FROM base AS microxrce-builder
WORKDIR /workspaces
RUN git clone https://github.com/eProsima/Micro-XRCE-DDS-Agent.git && \
    cd Micro-XRCE-DDS-Agent && mkdir build && cd build && \
    cmake .. && make -j"$(nproc)" && make install && ldconfig

# --- ACADOS builder ---
FROM base AS acados-builder
WORKDIR /workspaces
RUN git clone https://github.com/acados/acados.git --branch=v0.5.1 && \
    cd acados && git submodule update --init --recursive && \
    mkdir -p build && cd build && \
    cmake -DCMAKE_INSTALL_PREFIX=/opt/acados -DACADOS_WITH_QPOASES=ON .. && \
    make -j"$(nproc)" && make install


# ==============================================================================
# === Final Development Image
# ==============================================================================
FROM base AS final
WORKDIR /workspaces

# --- Bring in artifacts from builder stages ---
COPY --from=px4-builder          --chown=1000:1000 /workspaces/PX4-Autopilot /workspaces/PX4-Autopilot
COPY --from=microxrce-builder    /usr/local/                       /usr/local/
COPY --from=acados-builder       /opt/acados                       /opt/acados
COPY --from=acados-builder       --chown=1000:1000 /workspaces/acados      /workspaces/acados

# --- Install all development tools, editors, and LSPs ---
RUN mkdir -p /etc/apt/keyrings && \
    curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg && \
    echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_22.x nodistro main" | tee /etc/apt/sources.list.d/nodesource.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
    # For LSPs and code tooling
    clangd \
    nodejs \
    # General terminal utilities
    htop \
    tmux && \
    rm -rf /var/lib/apt/lists/*

# --- Install Node.js-based Language Servers ---
RUN npm install -g pyright typescript-language-server tree-sitter-cli

# Fix acados paths by linking the installed artifacts into the source tree
RUN mv /workspaces/acados/lib /workspaces/acados/lib_source && \
    ln -sf /opt/acados/lib /workspaces/acados/lib && \
    ln -sf /opt/acados/include /workspaces/acados/include && \
    chown -h 1000:1000 /workspaces/acados/lib /workspaces/acados/include


# --- Install Python stack ---
RUN pip3 install --break-system-packages --ignore-installed numpy==1.26.4 scipy==1.14.1 pandas seaborn matplotlib && \
    pip3 install --break-system-packages --timeout 900 --retries 5 -U "jax[cpu]==0.7.2" && \
    pip3 install --break-system-packages -e /workspaces/acados/interfaces/acados_template && \
    pip3 install --break-system-packages cflib transform3D bresenham python-dotenv

# --- Set environment variables for runtime ---
ENV ACADOS_SOURCE_DIR=/workspaces/acados
ENV LD_LIBRARY_PATH=/opt/acados/lib:/workspaces/acados/lib:${LD_LIBRARY_PATH}

# --- Switch to the non-root user ---
USER ros

# ---- Final setup as 'ros' user ----
RUN cd /workspaces/PX4-Autopilot && \
    pip3 install --break-system-packages --user -r Tools/setup/requirements.txt

RUN cd /workspaces/PX4-Autopilot && \
    python3 ./Tools/simulation/gz/simulation-gazebo --dryrun

RUN cd /home/ros/.simulation-gazebo && \
    wget https://raw.githubusercontent.com/PX4/PX4-gazebo-models/main/simulation-gazebo

RUN git config --global user.name "Riccardo Enrico" && \
    git config --global user.email "riccardo.enrico97@gmail.com"

# --- Add environment setup to user's .bashrc ---
RUN echo "\n# Sourcing ROS 2" >> /home/ros/.bashrc && \
    echo "source /opt/ros/jazzy/setup.bash" >> /home/ros/.bashrc && \
    echo "\n# Sourcing Local Workspace" >> /home/ros/.bashrc && \
    echo "if [ -f /workspaces/infogain_ros2/ros2_ws/install/setup.bash ]; then source /workspaces/infogain_ros2/ros2_ws/install/setup.bash; fi" >> /home/ros/.bashrc && \
    echo "\n# Environment Variables" >> /home/ros/.bashrc && \
    echo "export ACADOS_SOURCE_DIR=/workspaces/acados" >> /home/ros/.bashrc && \
    echo "export LD_LIBRARY_PATH=/opt/acados/lib:/workspaces/acados/lib:\${LD_LIBRARY_PATH}" >> /home/ros/.bashrc && \
    echo "export GZ_SIM_RESOURCE_PATH=/home/ros/.simulation-gazebo:/home/ros/.simulation-gazebo/models:/home/ros/.simulation-gazebo/worlds" >> /home/ros/.bashrc && \
    echo "export GZ_SIM_RESOURCE_PATH=\${GZ_SIM_RESOURCE_PATH}:/workspaces/PX4-Autopilot/Tools/simulation/gz:/workspaces/PX4-Autopilot/Tools/simulation/gz/models:/workspaces/PX4-Autopilot/Tools/simulation/gz/worlds" >> /home/ros/.bashrc && \
    echo "export GZ_SIM_SERVER_CONFIG_PATH=/home/ros/.simulation-gazebo/server.config" >> /home/ros/.bashrc && \
    echo "export ACADOS_INCLUDE_PATH=/opt/acados/include" >> /home/ros/.bashrc && \
    echo "export ACADOS_LIB_PATH=/opt/acados/lib" >> /home/ros/.bashrc && \
    echo "export PATH=/usr/local/bin:\${PATH}" >> /home/ros/.bashrc && \
    echo "\n# Aliases and Editor" >> /home/ros/.bashrc && \
    echo "# Neovim removed from this image; no editor aliases set" >> /home/ros/.bashrc && \
    echo "\n# Ccache" >> /home/ros/.bashrc && \
    echo "export CC=\"/usr/lib/ccache/gcc\"" >> /home/ros/.bashrc && \
    echo "export CXX=\"/usr/lib/ccache/g++\"" >> /home/ros/.bashrc

CMD ["/bin/bash"]
