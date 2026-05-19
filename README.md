# dev-container-template

Skeleton repository for a Python development container with Azure tooling preinstalled.

## Included tooling

- Python (via `mcr.microsoft.com/devcontainers/python`)
- Azure CLI (`az`)
- Azure Developer CLI (`azd`)
- Oh My Posh (`oh-my-posh`) with global Bash initialization
- Microsoft Skills pack (`microsoft/skills`) installed via `npx skills add microsoft/skills`

## Usage

1. Open this repository in VS Code.
2. Run **Dev Containers: Reopen in Container**.
3. After the container starts, verify tools:

   ```bash
   python --version
   az --version
   azd version
   oh-my-posh version
   ls -la .github/skills
   ```

## Reusing this dev container in other repositories

Every push to `main` that touches `.devcontainer/**` builds and publishes the image to GitHub Container Registry via [`.github/workflows/publish-devcontainer.yml`](.github/workflows/publish-devcontainer.yml):

```
ghcr.io/duongthaiha/python-azure-devcontainer:latest
```

In any other repository, drop in a minimal `.devcontainer/devcontainer.json` — no Dockerfile required:

```jsonc
{
  "name": "Python Azure",
  "image": "ghcr.io/duongthaiha/python-azure-devcontainer:latest",
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-python.python",
        "ms-python.vscode-pylance",
        "ms-azuretools.vscode-azurecli",
        "ms-azuretools.azd",
        "eamodio.gitlens"
      ]
    }
  },
  "postCreateCommand": "command -v npx >/dev/null && npx --yes skills add microsoft/skills --all || echo 'Skipping skills install: npx not available'",
  "remoteUser": "vscode"
}
```

### First-time setup

1. The image is published as **private** by default. After the first successful workflow run, open <https://github.com/users/duongthaiha/packages/container/python-azure-devcontainer/settings> and either:
   - set the package visibility to **Public** (anyone can pull), or
   - keep it private and grant read access to the consuming repos / users.
2. Consuming repos that pull a private image must `docker login ghcr.io` locally (or in CI use a PAT / `GITHUB_TOKEN` with `read:packages`).

### Notes

- The image is built for **linux/amd64** and **linux/arm64**, so it runs natively on Intel/AMD Linux & Windows hosts and on Apple Silicon Macs (M1/M2/M3/M4).
- Every build is also tagged with the commit SHA, so consumers can pin a specific version:

  ```jsonc
  "image": "ghcr.io/duongthaiha/python-azure-devcontainer:<commit-sha>"
  ```

## Using this as a Dev Container Template (VS Code picker)

In addition to the raw prebuilt image, this repo publishes a [Dev Container Template](https://containers.dev/implementors/templates/) to GHCR via [`.github/workflows/release-templates.yml`](.github/workflows/release-templates.yml). The template source lives under [`src/python-azure/`](src/python-azure/) and is published as:

```
ghcr.io/duongthaiha/dev-container-template/python-azure:latest
```

### Apply it to any folder

Easiest, no UI required — install the [devcontainer CLI](https://github.com/devcontainers/cli) once and run:

```bash
npm install -g @devcontainers/cli

devcontainer templates apply \
  --template-id ghcr.io/duongthaiha/dev-container-template/python-azure:latest \
  --workspace-folder .
```

This drops a minimal `.devcontainer/devcontainer.json` (pointing at the prebuilt image) into the target repo. Then run **Dev Containers: Reopen in Container**.

### Showing up in VS Code's "Add Dev Container Configuration Files…" picker

VS Code's built-in picker only lists templates from the **curated public index** at <https://containers.dev/templates>. To get this template into that dropdown for everyone, open a PR adding it to the [`devcontainers/templates`](https://github.com/devcontainers/templates) collection index — once merged it appears automatically in every user's VS Code without any local configuration.

Until then, the template is fully usable via the CLI command above.
