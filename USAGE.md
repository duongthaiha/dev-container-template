# Using this dev container in your repository

This guide shows how to add the prebuilt Python + Azure dev container to **any** repository in under a minute. No Dockerfile, no local build, no copy-paste of installation steps.

> **Image:** `ghcr.io/duongthaiha/python-azure-devcontainer:latest`
> **Architectures:** `linux/amd64`, `linux/arm64` (Apple Silicon native)
> **Source:** <https://github.com/duongthaiha/dev-container-template>

---

## What you get

Once your container starts, the following tooling is preinstalled and on `PATH`:

| Tool | Command | Notes |
|---|---|---|
| Python | `python --version` | 3.12 on Debian Bookworm |
| Azure CLI | `az --version` | Latest from `aka.ms/InstallAzureCLIDeb` |
| Azure Developer CLI | `azd version` | Latest from `aka.ms/install-azd.sh` |
| Bicep CLI | `bicep --version` | Installed from the latest GitHub release during image build |
| GitHub CLI | `gh --version` | Authenticate with `gh auth login`. Includes the built-in `gh copilot suggest` / `gh copilot explain` subcommands. |
| GitHub Copilot CLI | `copilot --help` | Standalone agent — `@github/copilot` on npm. Authenticate with `copilot` and follow the device flow on first run. Requires a Copilot subscription. |
| Node.js | `node -v` / `npm -v` | Node.js **22** from NodeSource (required by Copilot CLI) |
| Oh My Posh | `oh-my-posh version` | Enabled globally for Bash |
| Microsoft Skills pack | `ls ~/.copilot/skills` | **Baked into the image** at `/home/vscode/.copilot/skills/` (Copilot CLI's personal-skills path) — discovered automatically in any workspace, no network at container start, no consumer `postCreateCommand` changes required. Includes every top-level skill from `microsoft/skills` **plus the [`microsoft-foundry`](https://microsoft.github.io/skills/#plugin=microsoft-foundry) plugin** (Foundry router + 10 sub-skills). Refreshed every Monday by the publish workflow. |
| Product Backlog POC Planner | `ls ~/.copilot/skills/product-backlog-poc-planner` | Turns ideas into POC-ready engineering backlogs with Azure guardrails. **Baked into the image** and also installable standalone via `npx skills add duongthaiha/dev-container-template` (see [Installing skills without the container](#installing-skills-without-the-container)). |

User inside the container: **`vscode`** (non-root, passwordless `sudo`).

---

## Prerequisites

- **Docker Desktop** (or any Docker-compatible engine: Colima, Podman with the Docker socket, OrbStack, …).
- **VS Code** with the [**Dev Containers**](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) extension.
- *(Optional)* The [devcontainer CLI](https://github.com/devcontainers/cli) if you prefer the template-apply path: `npm install -g @devcontainers/cli`.

---

## Option A — Copy-paste (recommended, ~30 seconds)

In the **target repository**, create the file `.devcontainer/devcontainer.json` with this content:

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
  "postCreateCommand": "echo \"Microsoft Skills available: $(ls ~/.copilot/skills 2>/dev/null | wc -l)\"",
  "remoteUser": "vscode"
}
```

Then in VS Code:

1. Open the target repo.
2. Run the command palette (`Ctrl+Shift+P`) → **Dev Containers: Reopen in Container**.
3. Wait for the image to pull (first time only, ~1–2 minutes depending on connection).
4. Verify in the integrated terminal:

   ```bash
   python --version
   az --version
   azd version
   bicep --version
   gh --version
   copilot --version
   oh-my-posh version
   ```

That's it. Same workflow with **GitHub Codespaces** — no changes needed.

---

## Option B — Apply via the devcontainer CLI

Use this when scaffolding many repos or wiring into automation:

```bash
npm install -g @devcontainers/cli

devcontainer templates apply \
  --template-id ghcr.io/duongthaiha/dev-container-template/python-azure:latest \
  --workspace-folder .
```

This drops the same `.devcontainer/devcontainer.json` into the current folder.

---

## Pinning to a specific image version

`:latest` always tracks the most recent successful build on `main`. For reproducible environments (especially for shared teams or CI), pin to a commit SHA:

```jsonc
"image": "ghcr.io/duongthaiha/python-azure-devcontainer:<full-commit-sha>"
```

You can browse available tags at <https://github.com/duongthaiha/dev-container-template/pkgs/container/python-azure-devcontainer>.

To upgrade later, change the SHA and run **Dev Containers: Rebuild Container**.

---

## Customizing the container in your repo

The image is the *base*. Anything you add in your own `devcontainer.json` layers on top — **you don't need to fork or rebuild this image** for ordinary changes.

### Add more VS Code extensions

```jsonc
"customizations": {
  "vscode": {
    "extensions": [
      "ms-python.python",
      "github.copilot",
      "github.copilot-chat",
      "ms-toolsai.jupyter"
    ]
  }
}
```

### Install Python packages on container create

```jsonc
"postCreateCommand": "pip install -r requirements.txt"
```

Or combine with your own setup (skills are already available at `~/.copilot/skills/`, no sync needed):

```jsonc
"postCreateCommand": "pip install -r requirements.txt"
```

### Forward ports

```jsonc
"forwardPorts": [8000, 5173],
"portsAttributes": {
  "8000": { "label": "API", "onAutoForward": "notify" }
}
```

### Mount your local Azure / GitHub credentials

```jsonc
"mounts": [
  "source=${localEnv:HOME}/.azure,target=/home/vscode/.azure,type=bind,consistency=cached",
  "source=${localEnv:HOME}/.config/gh,target=/home/vscode/.config/gh,type=bind,consistency=cached"
]
```

> On Windows hosts replace `${localEnv:HOME}` with `${localEnv:USERPROFILE}`.

### Pass environment variables

```jsonc
"remoteEnv": {
  "AZURE_SUBSCRIPTION_ID": "${localEnv:AZURE_SUBSCRIPTION_ID}"
}
```

### Add dev container Features

The base image is fully compatible with the [Features ecosystem](https://containers.dev/features). Example — add Terraform and the GitHub CLI on top:

```jsonc
"image": "ghcr.io/duongthaiha/python-azure-devcontainer:latest",
"features": {
  "ghcr.io/devcontainers/features/terraform:1": {},
  "ghcr.io/devcontainers/features/github-cli:1": {}
}
```

---

## Using inside GitHub Codespaces

No extra configuration needed — the same `devcontainer.json` works as-is. Codespaces will pull the prebuilt image (fast) instead of building a Dockerfile.

If your repo is **private**, ensure the GHCR package is also reachable: this image is **public**, so no auth setup is required.

---

## Updating

- **Pinned to `:latest`** — run **Dev Containers: Rebuild Container** any time to pick up the newest image.
- **Pinned to a SHA** — bump the SHA in `devcontainer.json` and rebuild. Watch the [package page](https://github.com/duongthaiha/dev-container-template/pkgs/container/python-azure-devcontainer) for new versions.

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| `pull access denied` or `unauthorized` | The image is public; check Docker is logged out of stale credentials: `docker logout ghcr.io`. |
| Container build hangs on skills install layer | The Dockerfile runs `npx skills add microsoft/skills --all` at build time to bake skills into `~/.copilot/skills/`. If the build runner has no outbound HTTPS or a flaky npm registry, this layer will hang or fail. The weekly scheduled rebuild reruns this with `no-cache`, so transient failures self-heal on the next Monday run. |
| Container build hangs on `postCreateCommand` | Skills are now baked into the image and `postCreateCommand` only does a local `cp` — it should never hit the network. If it hangs, check the `python --version`/`az --version` smoke-test chain. |
| Wrong architecture / slow on Apple Silicon | Verify with `docker image inspect ghcr.io/duongthaiha/python-azure-devcontainer:latest --format '{{.Architecture}}'`. It should match `arm64` on M-series Macs. If not, `docker pull --platform linux/arm64 ghcr.io/duongthaiha/python-azure-devcontainer:latest` and rebuild. |
| Extensions not installing | Check the **Dev Containers** log: View → Output → "Dev Containers". Extensions install **after** container start. |
| Need a tool not in the base image | Add it via `features` or `postCreateCommand` in your own `devcontainer.json` — don't fork the base. |

---

## Reporting issues / contributing

File issues at <https://github.com/duongthaiha/dev-container-template/issues>. If you're adding tooling that should belong in the base image (not your repo), open a PR against the source repo's [.devcontainer/Dockerfile](.devcontainer/Dockerfile).

---

## Installing skills without the container

The **Product Backlog POC Planner** skill lives at [`.github/skills/`](.github/skills/) in this repo. You can install it into any workspace without using the dev container:

```bash
npx skills add duongthaiha/dev-container-template
```

This places the skill under `.agents/skills/product-backlog-poc-planner/` in your current directory. Copilot CLI, Claude Code, Cursor, Codex, and other supported agents discover it automatically.

### What the skill does

Turns a fuzzy product idea into a buildable POC backlog with:

- Intake questions to clarify scope
- A Mermaid architecture diagram (confirmed with the user before proceeding)
- Sequenced epics optimized for incremental dev delivery (infra → data → app)
- Azure guardrails (Bicep, `azd`, private endpoints, managed identity)
- Acceptance criteria, owner types, and a Definition of Done

Invoke it in Copilot CLI by mentioning keywords like *"product backlog"*, *"POC backlog"*, *"MVP planning"*, or *"engineering stories"*.

### Updating

If you installed via `npx skills add`, re-run the same command to pull the latest version. If you're using the dev container image, the skill is refreshed on every weekly rebuild.
