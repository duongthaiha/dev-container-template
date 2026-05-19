ARG VARIANT="3.12-bookworm"
FROM mcr.microsoft.com/devcontainers/python:${VARIANT}

RUN apt-get update \
    && export DEBIAN_FRONTEND=noninteractive \
    && apt-get install -y --no-install-recommends ca-certificates curl nodejs npm \
    && curl -sL https://aka.ms/InstallAzureCLIDeb | bash \
    && curl -fsSL https://aka.ms/install-azd.sh | bash \
    && curl -s https://ohmyposh.dev/install.sh | bash -s -- -d /usr/local/bin \
    && echo 'eval "$(oh-my-posh init bash)"' >> /etc/bash.bashrc \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*
