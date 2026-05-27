# Copilot instructions for `dev-container-template`

This repo is **not an application**. It produces two shipped artifacts that downstream repos consume:

1. A multi-arch container image — `ghcr.io/duongthaiha/python-azure-devcontainer:{latest,<sha>}` — built from [`.devcontainer/Dockerfile`](../.devcontainer/Dockerfile).
2. A Dev Container Template OCI artifact — `ghcr.io/duongthaiha/dev-container-template/python-azure:latest` — published from [`src/python-azure/`](../src/python-azure/).

Keep this distinction in mind: changes under `.devcontainer/` change the *image* consumers pull; changes under `src/` change the *template* the `devcontainer templates apply` CLI scaffolds. They are released by two separate workflows and must stay in sync (same image name, same `postCreateCommand` semantics).

## Architecture: two artifacts, two pipelines

| Path | Artifact | Workflow | Trigger |
|---|---|---|---|
| `.devcontainer/Dockerfile` + `.devcontainer/devcontainer.json` | GHCR image (multi-arch via parallel native amd64 + arm64 builds merged into one manifest) | [`.github/workflows/publish-devcontainer.yml`](workflows/publish-devcontainer.yml) | Push to `main` touching `.devcontainer/**`, weekly Monday cron, or manual dispatch |
| `src/python-azure/devcontainer-template.json` + `src/python-azure/.devcontainer/devcontainer.json` | Dev Container Template OCI artifact | [`.github/workflows/release-templates.yml`](workflows/release-templates.yml) | Push to `main` touching `src/**` |

The weekly cron exists so a fresh base image + latest `az` / `azd` / `bicep` / `@github/copilot` are picked up without code changes — do not add caching that would defeat this.

The image-building Dockerfile detects architecture via `dpkg --print-architecture` and downloads the matching `bicep-linux-{x64,arm64}` binary. Any new arch-specific tool must follow the same pattern.

## Tooling baked into the image

Anything in this list is preinstalled and on `PATH` for `vscode` (non-root, passwordless `sudo`):

`python` (3.12 bookworm base), `az`, `azd`, `bicep`, `gh`, `node`/`npm`/`npx` (Node.js **22** from NodeSource — required by `@github/copilot`), `copilot` (`@github/copilot` global npm), `oh-my-posh` (initialized globally in `/etc/bash.bashrc`).

If you add a tool, add it to **all three** of: `.devcontainer/Dockerfile`, the verification block in `README.md`, and the tooling table in `USAGE.md`. The `postCreateCommand` in `.devcontainer/devcontainer.json` runs `--version` on each tool as a smoke test — keep it in sync.

## Skills (baked into the image)

- The Microsoft Skills pack is **pre-installed into the image at build time** by a dedicated `RUN` step in `.devcontainer/Dockerfile` that runs (as the `vscode` user) two passes in a scratch dir, then relocates the tree to `/home/vscode/.copilot/skills/`:
  1. `npx --yes skills add microsoft/skills --all` — every top-level skill in `microsoft/skills` (`.github/skills/*`).
  2. `npx --yes skills add microsoft/skills --full-depth --skill microsoft-foundry --agent '*' -y` — the [Microsoft Foundry plugin](https://microsoft.github.io/skills/#plugin=microsoft-foundry) (orchestrator + 10 sub-skills). The default scan does **not** traverse `.github/plugins/<plugin>/skills/`, so `--full-depth` is required to surface plugin skills.
- The install location is **deliberately `~/.copilot/skills/` (Copilot CLI's personal-skills path)**, not a workspace-relative path. Copilot CLI auto-discovers personal skills in any workspace, so every consumer of this image gets all skills with **zero changes** to their own `devcontainer.json` / `postCreateCommand`. An earlier design that staged skills under `/opt/microsoft-skills/` and relied on `cp -rn` in `postCreateCommand` was abandoned because consumer repos with their own `devcontainer.json` would never run that copy.
- The `skills` CLI installs in **universal format** under `.agents/skills/<skill>/` in the install cwd; the Dockerfile moves that tree into `~/.copilot/skills/<skill>/`. The universal format is recognized by Copilot CLI, Claude Code, Cursor, Codex, and 12 other agents.
- The weekly Monday cron in `publish-devcontainer.yml` runs with `no-cache: true` and `pull: true`, so the skills `RUN` is re-executed and picks up the latest `microsoft/skills` content automatically — do not add caching that would defeat this.
- `skills-lock.json` at the repo root records `.github/skills/...` paths from an older skills-CLI layout — it is stale relative to the current `.agents/skills/` checkout used for *this repo's own* development. It is managed by the `skills` CLI; do not hand-edit.
- The skills source-of-truth checkout for *this* repo (for editing/testing) lives at `.agents/skills/<skill-name>/` (each contains a `SKILL.md`). That is unrelated to the in-image bake — consumers do not need it.
- To add another plugin skill (e.g. `foundry-models`, `foundry-hosted-agents`), append a pass inside the same `RUN` (still under `su vscode -c '...'`): `&& npx --yes skills add microsoft/skills --full-depth --skill <skill-name> --agent "*" -y`. To add a different skill source, use a separate `npx skills add <org>/<repo> --all`. Do **not** move these into `postCreateCommand` — that would re-introduce the network-at-startup dependency we removed AND fail for consumers who keep their own `postCreateCommand`.
- Note: the `microsoft-foundry` plugin's full experience also expects three MCP servers (Azure MCP, Foundry MCP, Microsoft Docs MCP). The skill *files* are baked into the image; the MCP servers themselves are configured per-agent at runtime (e.g. via `copilot`'s `/mcp` or `/plugin install microsoft-foundry@skills`) and are **not** auto-wired by this image.

## Consumer-facing contract (do not break)

Downstream repos pin one of:

- The image: `ghcr.io/duongthaiha/python-azure-devcontainer:latest` or `:<full-commit-sha>` (every successful build tags both).
- The template: `ghcr.io/duongthaiha/dev-container-template/python-azure:latest` via `devcontainer templates apply --template-id ...`.

The template definition in `src/python-azure/devcontainer-template.json` declares an `imageTag` option (default `latest`) that is substituted into the template's `devcontainer.json` as `${templateOption:imageTag}`. Any new template option must be declared here AND referenced in the template's `devcontainer.json` — both files are required for the OCI artifact to be valid.

When changing the published image name or template id, update **every** occurrence: `README.md`, `USAGE.md`, `src/python-azure/devcontainer-template.json` (`id`, `documentationURL`, `licenseURL`), and the workflow env `IMAGE` variable.

## No build / test / lint locally

There are no unit tests, linters, or build scripts in this repo. Validation is done by:

- Running `docker build .devcontainer` locally (or letting the workflow do it).
- Opening the repo in VS Code → **Dev Containers: Reopen in Container** and running the verification block from `README.md`:
  ```bash
  python --version && az --version && azd version && bicep --version \
    && gh --version && copilot --version && oh-my-posh version && ls -la ~/.copilot/skills
  ```
- The publish workflow's matrix build is the de facto CI — if `docker build` fails on either `linux/amd64` or `linux/arm64`, the merged manifest is not pushed.

Do not add a Node/Python project, a `package.json`, or a test framework unless the user explicitly asks — this repo intentionally ships only a Dockerfile + template manifest.

## Editing conventions specific to this repo

- The Dockerfile is a single `RUN` chain joined with `&&` and `\` line continuations so it forms one layer; preserve that structure when inserting steps and end with `apt-get clean && rm -rf /var/lib/apt/lists/*`.
- Markdown tables in `README.md` and `USAGE.md` are how features are advertised — when adding tooling, update the relevant table row, don't append prose.
- VS Code's built-in **Add Dev Container Configuration Files…** picker only lists templates from <https://containers.dev/templates>; getting this template there requires a PR to [`devcontainers/templates`](https://github.com/devcontainers/templates), not changes here.
