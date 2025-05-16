FROM ubuntu:24.04

ARG TARGETARCH=amd64
ARG S6_VERSION=v3.2.0.2

# install - base packages
RUN export DEBIAN_FRONTEND=noninteractive \
  && apt-get update && apt-get install -y \
     build-essential cmake git lsb-release \
     python3-dev python3-pip python3-venv \
     curl wget ca-certificates sudo locales tzdata \
     iputils-arping iputils-ping iputils-tracepath net-tools iproute2 \
     language-pack-zh-hans language-pack-zh-hant \
     unzip zip bzip2 xz-utils \
     openssh-server tmux vim \
  && systemctl disable ssh.service \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*

# args - software versions
# https://github.com/coder/code-server/releases
ARG CODESERVER_VERSION=4.100.2

# install - code-server
RUN curl -fsSL "https://github.com/coder/code-server/releases/download/v${CODESERVER_VERSION}/code-server_${CODESERVER_VERSION}_${TARGETARCH}.deb" -o /tmp/code-server.deb \
 && dpkg -i /tmp/code-server.deb \
 && rm -f /tmp/code-server.deb

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
 && rm /tmp/s6-overlay-noarch.tar.xz  \
       /tmp/s6-overlay-noarch.tar.xz.sha256 \
       /tmp/s6-overlay-${S6_ARCH}.tar.xz \
       /tmp/s6-overlay-${S6_ARCH}.tar.xz.sha256

# fix permissions of '/run' folder for s6
# https://github.com/just-containers/s6-overlay/blob/v3.2.0.0/layout/rootfs-overlay/package/admin/s6-overlay-%40VERSION%40/libexec/preinit#L86
RUN chmod 0775 /run

ARG NODE_MAJOR_VERSION=22

# install -- node.js
RUN export DEBIAN_FRONTEND=noninteractive \
 && curl -sL "https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key" | apt-key add - \
 && echo "deb https://deb.nodesource.com/node_${NODE_MAJOR_VERSION}.x nodistro main" > /etc/apt/sources.list.d/nodesource.list \
 && apt-get -yq update \
 && apt-get -yq install --no-install-recommends \
    nodejs \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

# setup environment for conda
ENV CONDA_DIR /opt/conda
ENV PATH "${CONDA_DIR}/bin:${PATH}"
RUN mkdir -pv ${CONDA_DIR} \
 && chmod 2775 ${CONDA_DIR} \
 && echo ". ${CONDA_DIR}/etc/profile.d/conda.sh" >> ${HOME}/.bashrc \
 && echo ". ${CONDA_DIR}/etc/profile.d/conda.sh" >> /etc/profile \
 && echo "conda activate base" >> ${HOME}/.bashrc \
 && echo "conda activate base" >> /etc/profile

# install - conda, pip, python, jupyter
ARG MINIFORGE_VERSION=25.3.0-3
ARG PYTHON_VERSION=3.12
ARG PIP_VERSION=25.1.1

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
&& conda install -y -q \
python=${PYTHON_VERSION} \
pip=${PIP_VERSION} \
&& conda update -y -q --all \
&& conda clean -a -f -y

# install - jupyter
ARG JUPYTERLAB_VERSION=4.4.2
ARG JUPYTER_VERSION=7.4.2

RUN echo "jupyterlab ==${JUPYTERLAB_VERSION}" >> ${CONDA_DIR}/conda-meta/pinned \
&& echo "notebook ==${JUPYTER_VERSION}" >> ${CONDA_DIR}/conda-meta/pinned \
&& conda install -y -q \
jupyterlab==${JUPYTERLAB_VERSION} \
notebook==${JUPYTER_VERSION} \
&& conda clean -a -f -y

# configure - jupyter
# NOTE: we use `--level=system` to write these configs at `/usr/local/etc/jupyter` because it defaults
##      to `/opt/conda/etc/jupyter` and conda cleans that directory after every install
RUN mkdir -pv /usr/local/etc/jupyter \
 && jupyter labextension disable --level=system "@jupyterlab/apputils-extension:announcements" \
 && jupyter labextension lock --level=system "@jupyterlab/apputils-extension"

# s6 - copy scripts
COPY --chmod=755 s6/ /etc

# ssh
EXPOSE 22
# code-server
EXPOSE 8866
# jupyterlab
EXPOSE 8888

ENTRYPOINT ["/init"]
