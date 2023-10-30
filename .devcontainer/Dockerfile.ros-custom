ARG FROM_IMAGE="ros:galactic"
ARG OVERLAY_WS="/opt/ros/overlay_ws"
###########################################################################################
#                                                                                         #
#  vs-code                                                                                       #
#                                                                                         #
###########################################################################################

FROM ${FROM_IMAGE} AS vscode-setter

ENV PATH_DEVCON_WS=/root
COPY ./dotfiles ${PATH_DEVCON_WS}/dotfiles
RUN . ${PATH_DEVCON_WS}/dotfiles/dotfile_symlink_setting.sh

#ENTRYPOINT /bin/bash

##########################################################################################
#                                                                                        #
#  cache                                                                                 #
#                                                                                        #
##########################################################################################


FROM ${FROM_IMAGE} AS cacher
ARG OVERLAY_WS=/opt/ros/overlay_ws
COPY ${localWorkspaceFolder}/${OriginalWorkspacePath} ${OVERLAY_WS}/src
WORKDIR $OVERLAY_WS

WORKDIR /opt
RUN mkdir -p /tmp/opt && \
  find ./ -name "package.xml" | \
  xargs cp --parents -t /tmp/opt && \
  find ./ -name "COLCON_IGNORE" | \
  xargs cp --parents -t /tmp/opt || true

##########################################################################################
#                                                                                        #
#                                                                                        #
#  dependencies                                                                               #
#                                                                                        #
#                                                                                        #
##########################################################################################

FROM vscode-setter AS dependencies
RUN apt-get update
#RUN apt-get install -y ros-galactic-rmw-fastrtps-cpp
ARG OVERLAY_WS
WORKDIR ${OVERLAY_WS}

# COPY --from=cacher /tmp/${OVERLAY_WS}/src ./src

RUN   . /opt/ros/${ROS_DISTRO}/setup.sh && \
  apt-get update && \
  rosdep install -y \
  --from-paths \
    ./src \
  --ignore-src
RUN rm -rf /var/lib/apt/lists/*

##########################################################################################
#                                                                                        #
#                                                                                        #
#  builder                                                                               #
#                                                                                        #
#                                                                                        #
##########################################################################################

FROM dependencies AS builder
ARG OVERLAY_WS
WORKDIR ${OVERLAY_WS}


# build overlay source
COPY --from=cacher ${OVERLAY_WS}/src ./src
ARG OVERLAY_MIXINS="release"
RUN . /opt/ros/${ROS_DISTRO}/setup.sh && \
  colcon build \
    --packages-select \
      demo_nodes_cpp \
      demo_nodes_py \
  --mixin $OVERLAY_MIXINS

# source entrypoint setup
ENV OVERLAY_WS $OVERLAY_WS

RUN sed -i 's/# setup ros2 environment/export ROS_DOMAIN_ID=57\nexport RMW_IMPLEMENTATION=rmw_fastrtps_cpp\n/' /ros_entrypoint.sh
RUN sed --in-place --expression \
  '$isource "$OVERLAY_WS/install/setup.bash"' \
  /ros_entrypoint.sh

ENTRYPOINT ["/ros_entrypoint.sh"]

# run launch file
CMD ["ros2", "launch", "demo_nodes_cpp", "talker_listener.launch.py"]