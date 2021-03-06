FROM bitnami/minideb:buster

# Make all shells run in a safer way. Ref: https://vaneyckt.io/posts/safer_bash_scripts_with_set_euxo_pipefail/
SHELL [ "/bin/bash", "-euxo", "pipefail", "-c" ]
WORKDIR /

# Need root to do rooty things
USER root

# Install debian apt-get packages
# Some of this is lifted from bitnami/minideb-extras
RUN install_packages \
  apt-transport-https \
  build-essential \
  ca-certificates \
  curl \
  dirmngr \
  git \
  gnupg \
  gosu \
  libaio1 \
  locales \
  openssh-client \
  pkg-config \
  procps \
  python3 \
  python3-pip \
  python3-setuptools \
  python3-venv \
  unzip

# Update the locale for container-optimized settings, and add a user so we aren't stuck with using root.
RUN update-locale LANG=C.UTF-8 LC_MESSAGES=POSIX \
  && locale-gen en_US.UTF-8 \
  && DEBIAN_FRONTEND=noninteractive dpkg-reconfigure locales \
  && useradd -ms /bin/bash dadsgarage

# The following security actions are recommended by some security scans.
# https://console.bluemix.net/docs/services/va/va_index.html
# This is lifted straight out of bitnami/minideb-extras
RUN  sed -i 's/^PASS_MAX_DAYS.*/PASS_MAX_DAYS    90/' /etc/login.defs \
  && sed -i 's/^PASS_MIN_DAYS.*/PASS_MIN_DAYS    1/' /etc/login.defs \
  && sed -i 's/sha512/sha512 minlen=8/' /etc/pam.d/common-password

# Add Tini. Ref: https://github.com/krallin/tini
ARG TINI_VERSION=v0.18.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /tini
RUN chmod +x /tini

# Install kubectl
ARG KUBECTL_VERSION=1.16.3-00
RUN curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - && \
  echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | tee -a /etc/apt/sources.list.d/kubernetes.list && \
  install_packages kubectl=${KUBECTL_VERSION}

# Quit being root for safety.
USER dadsgarage

# Install pipx for safer pip installations of tools. Ref: https://pipxproject.github.io/pipx/
ENV PATH="${PATH}:/home/dadsgarage/.local/bin"
RUN python3 -m pip install --user pipx \
  && pipx ensurepath

# Install pre-commit
ARG PRE_COMMIT_VERSION=1.20.0
RUN pipx install pre-commit==${PRE_COMMIT_VERSION}

# Install AWS CLI
ARG AWS_CLI_VERSION=1.16.287
RUN pipx install awscli==${AWS_CLI_VERSION}

USER root

ENTRYPOINT ["/tini", "--"]
