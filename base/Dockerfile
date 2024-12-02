#
# https://github.com/kubeflow/kubeflow/blob/v1.9.2/components/example-notebook-servers/base/Dockerfile
#

FROM ubuntu:22.04

ARG TARGETARCH

# common environemnt variables
ENV LLMWEBUI_USER=llmwebui
ARG LLMWEBUI_UID=1000
# WARNING: the primary GID of 'llmwebui' MUST be 0!
#          this allows any UID to be used as all important folders are owned by GID 0.
#          some Kubernetes environments run containers as a random UID (e.g. OpenShift).
#          note, having GID 0 (root) does NOT give you root permissions, so this is not a security issue.
ENV LLMWEBUI_GID=0
ENV HOME=/home/$LLMWEBUI_USER

# the GID of the 'users' group
ARG USERS_GID=100

# we copy the contents of $HOME_TMP to $HOME on startup
# this is to work around the fact that a PVC will be mounted to $HOME
# but we still want to have some default files in $HOME
# see `s6/cont-init.d/01-copy-tmp-home`
ENV HOME_TMP=/tmp_home/$LLMWEBUI_USER

# s6-overlay only gives 5 seconds by default, which is too small for slow PVC storage backends
# when running `/etc/cont-inid.d/01-copy-tmp-home` (note, this is in milliseconds)
ENV S6_CMD_WAIT_FOR_SERVICES_MAXTIME=300000

# s6-overlay does not fail by default if the `/etc/cont-init.d/` or `/etc/services.d/` scripts fail
# this is not the desired behavior, so we set it to fail
ENV S6_BEHAVIOUR_IF_STAGE2_FAILS=2

# args - software versions
# https://kubernetes.io/releases/
# https://github.com/just-containers/s6-overlay/releases
ARG KUBECTL_VERSION=v1.29.9
ARG S6_VERSION=v3.2.0.0

# install - usefull linux packages
RUN DEBIAN_FRONTEND=noninteractive apt-get -yq update \
 && DEBIAN_FRONTEND=noninteractive apt-get -yq install --no-install-recommends \
    apt-transport-https \
    bash \
    bzip2 \
    ca-certificates \
    curl \
    git \
    gnupg \
    gnupg2 \
    locales \
    lsb-release \
    nano \
    software-properties-common \
    tzdata \
    unzip \
    vim \
    wget \
    xz-utils \
    zip \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

# install - s6 overlay
RUN case "${TARGETARCH}" in \
      amd64) S6_ARCH="x86_64" ;; \
      arm64) S6_ARCH="aarch64" ;; \
      ppc64le) S6_ARCH="ppc64le" ;; \
      *) echo "Unsupported architecture: ${TARGETARCH}"; exit 1 ;; \
    esac \
 && curl -fsSL "https://github.com/just-containers/s6-overlay/releases/download/${S6_VERSION}/s6-overlay-noarch.tar.xz" -o /tmp/s6-overlay-noarch.tar.xz \
 && curl -fsSL "https://github.com/just-containers/s6-overlay/releases/download/${S6_VERSION}/s6-overlay-noarch.tar.xz.sha256" -o /tmp/s6-overlay-noarch.tar.xz.sha256 \
 && echo "$(cat /tmp/s6-overlay-noarch.tar.xz.sha256 | awk '{ print $1; }')  /tmp/s6-overlay-noarch.tar.xz" | sha256sum -c - \
 && curl -fsSL "https://github.com/just-containers/s6-overlay/releases/download/${S6_VERSION}/s6-overlay-${S6_ARCH}.tar.xz" -o /tmp/s6-overlay-${S6_ARCH}.tar.xz \
 && curl -fsSL "https://github.com/just-containers/s6-overlay/releases/download/${S6_VERSION}/s6-overlay-${S6_ARCH}.tar.xz.sha256" -o /tmp/s6-overlay-${S6_ARCH}.tar.xz.sha256 \
 && echo "$(cat /tmp/s6-overlay-${S6_ARCH}.tar.xz.sha256 | awk '{ print $1; }')  /tmp/s6-overlay-${S6_ARCH}.tar.xz" | sha256sum -c - \
 && tar -C / -Jxpf /tmp/s6-overlay-noarch.tar.xz \
 && tar -C / -Jxpf /tmp/s6-overlay-${S6_ARCH}.tar.xz \
 && rm /tmp/s6-overlay-noarch.tar.xz \
       /tmp/s6-overlay-noarch.tar.xz.sha256 \
       /tmp/s6-overlay-${S6_ARCH}.tar.xz \
       /tmp/s6-overlay-${S6_ARCH}.tar.xz.sha256

# fix permissions of '/run' folder for s6
# https://github.com/just-containers/s6-overlay/blob/v3.2.0.0/layout/rootfs-overlay/package/admin/s6-overlay-%40VERSION%40/libexec/preinit#L86
RUN chmod 0775 /run

# install - kubectl
RUN curl -fsSL "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/${TARGETARCH}/kubectl" -o /usr/local/bin/kubectl \
 && curl -fsSL "https://dl.k8s.io/${KUBECTL_VERSION}/bin/linux/${TARGETARCH}/kubectl.sha256" -o /tmp/kubectl.sha256 \
 && echo "$(cat /tmp/kubectl.sha256 | awk '{ print $1; }')  /usr/local/bin/kubectl" | sha256sum -c - \
 && rm /tmp/kubectl.sha256 \
 && chmod +x /usr/local/bin/kubectl

# create user and set required ownership
RUN useradd -M -N \
    --shell /bin/bash \
    --home ${HOME} \
    --uid ${LLMWEBUI_UID} \
    --gid ${LLMWEBUI_GID} \
    --groups ${USERS_GID} \
    ${LLMWEBUI_USER} \
 && mkdir -pv ${HOME} \
 && mkdir -pv ${HOME_TMP} \
    # in the interest of backwards compatibility we have the 'users' group owns the home directory
    # we also set the SGID bit so that new files and directories are created with the 'users' group
 && chmod 2775 ${HOME} \
 && chmod 2775 ${HOME_TMP} \
 && chown -R ${LLMWEBUI_USER}:${USERS_GID} ${HOME} \
 && chown -R ${LLMWEBUI_USER}:${USERS_GID} ${HOME_TMP} \
 && chown -R ${LLMWEBUI_USER}:${LLMWEBUI_GID} /usr/local/bin

# set locale configs
RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen \
 && locale-gen
ENV LANG=en_US.UTF-8
ENV LANGUAGE=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8

# s6 - copy scripts
COPY --chown=${LLMWEBUI_USER}:${LLMWEBUI_GID} --chmod=755 s6/ /etc

ENTRYPOINT ["/init"]


#
# https://github.com/kubeflow/kubeflow/blob/v1.9.2/components/example-notebook-servers/jupyter/Dockerfile
#

# args - software versions
ARG MINIFORGE_VERSION=24.7.1-2
ARG PYTHON_VERSION=3.11.10
ARG CUDA_TOOLKIT_VERSION=12.6.3

# setup environment for conda
ENV CONDA_DIR=/opt/conda
ENV PATH="${CONDA_DIR}/bin:${PATH}"
RUN mkdir -pv ${CONDA_DIR} \
 && chmod 2775 ${CONDA_DIR} \
 && echo ". /opt/conda/etc/profile.d/conda.sh" >> ${HOME}/.bashrc \
 && echo ". /opt/conda/etc/profile.d/conda.sh" >> /etc/profile \
 && echo "conda activate base" >> ${HOME}/.bashrc \
 && echo "conda activate base" >> /etc/profile \
 && chown -R ${LLMWEBUI_USER}:${LLMWEBUI_GID} ${CONDA_DIR} \
 && chown -R ${LLMWEBUI_USER}:${USERS_GID} ${HOME}

# switch to LLMWEBUI_UID for installs
USER $LLMWEBUI_UID
WORKDIR $HOME

# install - conda, python, cuda-toolkit
RUN case "${TARGETARCH}" in \
      amd64) MINIFORGE_ARCH="x86_64" ;; \
      arm64) MINIFORGE_ARCH="aarch64" ;; \
      ppc64le) MINIFORGE_ARCH="ppc64le" ;; \
      *) echo "Unsupported architecture: ${TARGETARCH}"; exit 1 ;; \
    esac \
 && curl -fsSL "https://github.com/conda-forge/miniforge/releases/download/${MINIFORGE_VERSION}/Miniforge3-${MINIFORGE_VERSION}-Linux-${MINIFORGE_ARCH}.sh" -o /tmp/Miniforge3.sh \
 && curl -fsSL "https://github.com/conda-forge/miniforge/releases/download/${MINIFORGE_VERSION}/Miniforge3-${MINIFORGE_VERSION}-Linux-${MINIFORGE_ARCH}.sh.sha256" -o /tmp/Miniforge3.sh.sha256 \
 && echo "$(cat /tmp/Miniforge3.sh.sha256 | awk '{ print $1; }')  /tmp/Miniforge3.sh" | sha256sum -c - \
 && rm /tmp/Miniforge3.sh.sha256 \
 && /bin/bash /tmp/Miniforge3.sh -b -f -p ${CONDA_DIR} \
 && rm /tmp/Miniforge3.sh \
 && conda config --system --set auto_update_conda false \
 && conda config --system --set show_channel_urls true \
 && echo "python ==${PYTHON_VERSION}" >> ${CONDA_DIR}/conda-meta/pinned \
 && conda install -y -q python=${PYTHON_VERSION} \
 && conda update -y -q --all \
 && conda install -y -q nvidia::cuda-toolkit=${CUDA_TOOLKIT_VERSION} \
 && conda clean -a -f -y

# s6 - 01-copy-tmp-home
# NOTE: the contents of $HOME_TMP are copied to $HOME at runtime
#       this is a workaround because a PVC will be mounted at $HOME
#       and the contents of $HOME will be hidden
RUN cp -p -r -T "${HOME}" "${HOME_TMP}" \
    # give group same access as user (needed for OpenShift)
 && chmod -R g=u "${HOME_TMP}"


#
# https://github.com/kubeflow/kubeflow/blob/v1.9.2/components/example-notebook-servers/jupyter-pytorch-cuda/Dockerfile
#

# nvidia container toolkit
# https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/docker-specialized.html
ENV NVIDIA_VISIBLE_DEVICES=all
ENV NVIDIA_DRIVER_CAPABILITIES=compute,utility
ENV NVIDIA_REQUIRE_CUDA="cuda>=12.1"


#
# https://github.com/kubeflow/kubeflow/blob/v1.9.2/components/example-notebook-servers/jupyter-pytorch-cuda-full/Dockerfile
#

# install - conda packages
# NOTE: we use mamba to speed things up
RUN mamba install -y -q \
    bokeh==3.3.4 \
    cloudpickle==2.2.1 \
    dill==0.3.8 \
    matplotlib==3.8.4 \
    numpy==1.24.4 \
    pandas==2.1.4 \
    scikit-image==0.22.0 \
    scikit-learn==1.3.2 \
    scipy==1.11.3 \
    seaborn==0.13.2 \
    xgboost==1.7.6 \
 && mamba clean -a -f -y
