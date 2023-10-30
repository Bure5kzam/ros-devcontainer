ARG FROM_IMAGE=ros:humble
FROM ${FROM_IMAGE} AS vscode-setter

ENV PATH_DEVCON_WS=/root
COPY ./dotfiles ${PATH_DEVCON_WS}/dotfiles
RUN . ${PATH_DEVCON_WS}/dotfiles/dotfile_symlink_setting.sh

ENTRYPOINT /bin/bash
