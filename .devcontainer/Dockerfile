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

# Pre-bake the Microsoft Skills pack into the image at the GitHub Copilot CLI
# personal-skills path (`$HOME/.copilot/skills/`) so every consumer of this
# image gets the full skill set automatically — regardless of what
# postCreateCommand or workspace .agents/skills/ layout their repo uses.
# This path is auto-discovered by Copilot CLI in any workspace, with no
# need for consumers to update their own devcontainer.json.
# Refreshed on every weekly scheduled rebuild (workflow runs with no-cache).
#
# Two install passes (run as the vscode user so HOME resolves correctly):
#   1. `--all` pulls every top-level skill from `microsoft/skills`.
#   2. `--full-depth --skill microsoft-foundry` pulls the Microsoft Foundry
#      plugin's orchestrator skill (it lives under
#      `.github/plugins/microsoft-foundry/`, which the default scan does NOT
#      traverse — `--full-depth` is required).
#      See: https://microsoft.github.io/skills/#plugin=microsoft-foundry
#
# The skills CLI writes to `<cwd>/.agents/skills/` (universal format). We run
# the install in a scratch dir, then move the tree under
# `/home/vscode/.copilot/skills/` (Copilot CLI personal skills location).
RUN su vscode -c '\
        set -eux \
        && mkdir -p /tmp/skills-install \
        && cd /tmp/skills-install \
        && npx --yes skills add microsoft/skills --all \
        && npx --yes skills add microsoft/skills --full-depth --skill microsoft-foundry --agent "*" -y \
        && mkdir -p /home/vscode/.copilot/skills \
        && cp -r /tmp/skills-install/.agents/skills/. /home/vscode/.copilot/skills/ \
        && rm -rf /tmp/skills-install /home/vscode/.npm \
    '
