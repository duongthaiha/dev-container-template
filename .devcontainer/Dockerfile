ARG VARIANT="3.12-bookworm"
FROM mcr.microsoft.com/devcontainers/python:${VARIANT}

RUN apt-get update \
    && export DEBIAN_FRONTEND=noninteractive \
    && apt-get install -y --no-install-recommends ca-certificates curl gnupg \
    # Azure CLI
    && curl -sL https://aka.ms/InstallAzureCLIDeb | bash \
    # Bicep CLI
    && arch="$(dpkg --print-architecture)" \
    && case "$arch" in amd64) bicep_arch=x64 ;; arm64) bicep_arch=arm64 ;; *) echo "Unsupported architecture: $arch" && exit 1 ;; esac \
    && curl -fsSL "https://github.com/Azure/bicep/releases/latest/download/bicep-linux-${bicep_arch}" \
        -o /usr/local/bin/bicep \
    && chmod +x /usr/local/bin/bicep \
    # Azure Developer CLI
    && curl -fsSL https://aka.ms/install-azd.sh | bash \
    # Oh My Posh
    && curl -s https://ohmyposh.dev/install.sh | bash -s -- -d /usr/local/bin \
    && echo 'eval "$(oh-my-posh init bash)"' >> /etc/bash.bashrc \
    # Node.js 22 (required by @github/copilot; also provides npm + npx)
    && curl -fsSL https://deb.nodesource.com/setup_22.x | bash - \
    && apt-get install -y --no-install-recommends nodejs \
    # GitHub CLI
    && mkdir -p -m 755 /etc/apt/keyrings \
    && curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg \
        -o /etc/apt/keyrings/githubcli-archive-keyring.gpg \
    && chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" \
        > /etc/apt/sources.list.d/github-cli.list \
    && apt-get update \
    && apt-get install -y --no-install-recommends gh \
    # GitHub Copilot CLI (provides the `copilot` command)
    && npm install -g @github/copilot \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*
