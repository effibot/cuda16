# Pull nvidia cuda image for drivers
FROM nvidia/cudagl:11.3.1-base-ubuntu16.04

# Define environment variables
ARG username=dock
ARG uid=1000
ARG gid=1000
ENV USER=$username
ENV UID=$uid
ENV GID=$gid
ENV HOME=/home/$USER
ARG DEBIAN_FRONTEND=noninteractive

ENV CONDA_DIR=${HOME}/conda
ENV PATH=${CONDA_DIR}/bin:${PATH}

# Create non-root user with UID and GID and workspace directories
RUN groupadd -g ${gid} ${username} && \
    useradd -u ${uid} -g ${gid} -s /bin/bash -m ${username} && \
    mkdir -p /etc/sudoers.d && \
    echo ${username} ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/${username} &&\
    chmod 0440 /etc/sudoers.d/${username} && \
    mkdir -p ${HOME}/zsh_history && \
    chown ${USER}:${USER} ${HOME}/zsh_history && \
    mkdir ${HOME}/workspace && \
    chown ${USER}:${USER} ${HOME}/workspace

# Copy user configuration files
COPY --chown=${USER}:${USER} ./config/.aliases.sh ${HOME}/
COPY --chown=${USER}:${USER} ./config/.nanorc ${HOME}/
COPY --chown=${USER}:${USER} ./config/.bashrc ${HOME}/


# Install packages and clean up
RUN apt-key adv --fetch-keys http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/3bf863cc.pub && \
    apt-key adv --fetch-keys http://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu1604/x86_64/7fa2af80.pub && \
    apt update && apt install -q -y --no-install-recommends --fix-missing --allow-change-held-packages \
    apt-utils \
    ca-certificates curl cmake \
    gcc g++ git \
    make \
    nano \
    openssl openssh-client \
    qtcreator \
    sudo \
    unzip \
    wget \
    zip \
    && \
    apt full-upgrade -y && apt autoremove -y && apt clean -y && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*/apt/lists/*

# Install LSD for ls substitute
RUN wget https://github.com/Peltoche/lsd/releases/download/0.23.1/lsd_0.23.1_amd64.deb -P /tmp && \
    dpkg -i /tmp/lsd_0.23.1_amd64.deb && \
    rm /tmp/lsd_0.23.1_amd64.deb

# Switch to non-root user
USER ${USER}

# Install ZSH and Oh My ZSH with Powerlevel10k theme through script
RUN sh -c "$(wget -O- https://github.com/deluan/zsh-in-docker/releases/download/v1.1.5/zsh-in-docker.sh)" -- \
    -p git \
    -p git-extras \
    -p git-flow \
    -p command-not-found \
    -p colorize \
    -p common-aliases \
    -p sudo \
    -p zsh-interactive-cd \
    -p https://github.com/chrissicool/zsh-256color \
    -p https://github.com/zsh-users/zsh-autosuggestions \
    -p https://github.com/zsh-users/zsh-completions \
    -p https://github.com/zshzoo/cd-ls \
    -p https://github.com/zdharma-continuum/fast-syntax-highlighting \
    -a "zstyle ':omz:update' mode auto" \
    -a "HIST_STAMPS='dd.mm.yyyy'" \
    -a "typeset -g POWERLEVEL9K_MODE=nerdfont-complete" && \
    echo "source ~/.aliases.sh" >> ${HOME}/.zshrc

# Install Mambaforge and conda packages, then clean up
RUN wget "https://github.com/conda-forge/miniforge/releases/latest/download/Mambaforge-$(uname)-$(uname -m).sh" -O ~/miniforge.sh && \
    chmod +x ~/miniforge.sh && \
    /bin/bash ~/miniforge.sh -b -p ${CONDA_DIR} &&  \
    rm ~/miniforge.sh && \
    mamba update --all --yes && \
    mamba install fzf -y && \
    conda clean --tarballs --index-cache --packages --yes && \
    find ${CONDA_DIR} -follow -type f -name '*.a' -delete && \
    find ${CONDA_DIR} -follow -type f -name '*.pyc' -delete && \
    conda clean --force-pkgs-dirs --all --yes && \
    mamba init zsh && mamba init bash && \
    sudo chsh -s /bin/zsh ${USER}

# Start zsh
WORKDIR ${HOME}
CMD ["/usr/bin/zsh", "-l"]