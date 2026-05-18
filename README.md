# dev-container-template

Skeleton repository for a Python development container with Azure tooling preinstalled.

## Included tooling

- Python (via `mcr.microsoft.com/devcontainers/python`)
- Azure CLI (`az`)
- Azure Developer CLI (`azd`)
- Oh My Posh (`oh-my-posh`) with global Bash initialization

## Usage

1. Open this repository in VS Code.
2. Run **Dev Containers: Reopen in Container**.
3. After the container starts, verify tools:

   ```bash
   python --version
   az --version
   azd version
   oh-my-posh version
   ```
