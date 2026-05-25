# dev-container-template

Skeleton repository for a Python development container with Azure tooling preinstalled.

## Included tooling

- Python (via `mcr.microsoft.com/devcontainers/python`)
- Azure CLI (`az`)
- Azure Developer CLI (`azd`)
- Bicep CLI (`bicep`)
- GitHub CLI (`gh`, with built-in `gh copilot suggest` / `gh copilot explain`)
- GitHub Copilot CLI (`copilot`, via `@github/copilot`)
- Node.js 22 (`node`, `npm`, `npx`) from NodeSource
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
   bicep --version
   gh --version
   copilot --version
   oh-my-posh version
   ls -la .github/skills
   ```

## Reusing this dev container in other repositories

**See [USAGE.md](USAGE.md) for the full consumer guide** — covers copy-paste setup, the `devcontainer templates apply` CLI flow, version pinning, customization, Codespaces, and troubleshooting.

TL;DR — drop this into `.devcontainer/devcontainer.json` in any other repo and run *Dev Containers: Reopen in Container*:

```jsonc
{
  "name": "Python Azure",
  "image": "ghcr.io/duongthaiha/python-azure-devcontainer:latest",
  "remoteUser": "vscode"
}
```

The image is public (multi-arch: `linux/amd64` + `linux/arm64`), so no `docker login` is needed.

## Publishing pipeline (maintainers)

This repository hosts two GitHub Actions workflows that ship the artifacts consumers depend on:

| Workflow | Trigger | What it publishes |
|---|---|---|
| [`.github/workflows/publish-devcontainer.yml`](.github/workflows/publish-devcontainer.yml) | Push to `main` touching `.devcontainer/**`, plus a weekly Monday schedule | Multi-arch image `ghcr.io/duongthaiha/python-azure-devcontainer:{latest, <sha>}`. Native builds on `ubuntu-latest` + `ubuntu-24.04-arm` in parallel, merged into one manifest; the weekly run pulls a fresh base image and bypasses Docker layer cache so the latest Azure / azd / Bicep / Copilot tooling is republished automatically. |
| [`.github/workflows/release-templates.yml`](.github/workflows/release-templates.yml) | Push to `main` touching `src/**` | Dev Container Template OCI artifact `ghcr.io/duongthaiha/dev-container-template/python-azure:latest`. |

Every successful image build is tagged with both `:latest` and the commit SHA, so consumers can pin to a specific version. Browse all published versions at the [package page](https://github.com/duongthaiha/dev-container-template/pkgs/container/python-azure-devcontainer).

### Getting the template into VS Code's built-in picker

VS Code's **Add Dev Container Configuration Files…** picker only lists templates from the curated public index at <https://containers.dev/templates>. To get this template into that dropdown for everyone, open a PR adding it to the [`devcontainers/templates`](https://github.com/devcontainers/templates) collection index — once merged it appears automatically in every user's VS Code with no local configuration. Until then, consumers use the `devcontainer templates apply` CLI command (see [USAGE.md](USAGE.md#option-b--apply-via-the-devcontainer-cli)).
