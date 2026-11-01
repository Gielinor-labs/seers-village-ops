# Getting started

Prepare your local machine to work on Seers' Ops using Windows, macOS or Linux.

## Tool versions

The proposed minimum tool versions are defined in [versions.env](../versions.env).

These minimum versions are provisional until the guide has been reviewed and tested on at least two operating systems.

## Required tools

| Tool | Required on | Purpose |
| --- | --- | --- |
| Docker Desktop | Windows and macOS | Provides Docker Engine and manages its Linux virtual machine. |
| Docker Engine | All systems; included in Docker Desktop | Runs the Linux containers used by kind. |
| kind | All systems | Creates a local Kubernetes cluster using containers as nodes. |
| kubectl | All systems | Communicates with the Kubernetes API to manage and inspect cluster resources. |
| Helm | All systems | Installs and manages Kubernetes applications packaged as charts. |
| Go Task | All systems | Runs named tasks defined in a Taskfile. |
| WSL with the WSL2 backend | Windows | Provides the Linux environment required by the Docker Desktop setup in this guide. |

## Load tool versions

Run the following commands from the repository root to load the proposed minimum versions into your current shell session.

### Windows (PowerShell)

```powershell
$toolVersions = Get-Content -LiteralPath .\versions.env -Raw | ConvertFrom-StringData
$toolVersions.KIND_MIN_VERSION
```

### macOS and Linux (sh, bash or zsh)

```sh
. ./versions.env
printf '%s\n' "$KIND_MIN_VERSION"
```

## Windows setup

Run the Windows commands in PowerShell.

### 1. Check system requirements

Use a Windows release supported by [Docker Desktop](https://docs.docker.com/desktop/setup/install/windows-install/), with hardware virtualization enabled and at least 8 GB of RAM.

Check WinGet availability and inspect your WSL installation:

```powershell
winget --version
wsl --version
wsl --status
```

If WinGet is unavailable, install or update [App Installer](https://learn.microsoft.com/en-us/windows/package-manager/winget/) and reopen PowerShell.

### 2. Prepare WSL2

If WSL is missing, open PowerShell as Administrator, run `wsl --install --no-distribution`, and restart Windows if prompted.

If the installed WSL version is below the minimum in `versions.env`, run `wsl --update` and follow any restart instructions.

Run `wsl --set-default-version 2` to use WSL2 for newly installed distributions.

Docker Desktop manages its own Linux distribution. This PowerShell workflow does not require installing Ubuntu separately. See the [Docker WSL2 backend documentation](https://docs.docker.com/desktop/features/wsl/).

### 3. Install Docker Desktop

From the repository root, load `$toolVersions` as shown in [Load tool versions](#load-tool-versions), then install Docker Desktop:

```powershell
winget install --id Docker.DockerDesktop --exact --source winget --version $toolVersions.DOCKER_DESKTOP_MIN_VERSION --interactive
```

If Docker Desktop is already installed below the proposed minimum, update it instead:

```powershell
winget upgrade --id Docker.DockerDesktop --exact --source winget --version $toolVersions.DOCKER_DESKTOP_MIN_VERSION --interactive
```

Start Docker Desktop, select the WSL2 backend, use Linux containers, and wait for the engine to start.

Check the Desktop and Engine versions against `versions.env`. The second command must print `linux`:

```powershell
docker version
docker info --format '{{.OSType}}'
```

### 4. Install the command-line tools

From the repository root, load the version settings and install each tool:

```powershell
$toolVersions = Get-Content -LiteralPath .\versions.env -Raw | ConvertFrom-StringData
winget install --id Kubernetes.kind --exact --source winget --version $toolVersions.KIND_MIN_VERSION
winget install --id Kubernetes.kubectl --exact --source winget --version $toolVersions.KUBECTL_MIN_VERSION
winget install --id Helm.Helm --exact --source winget --version $toolVersions.HELM_MIN_VERSION
winget install --id Task.Task --exact --source winget --version $toolVersions.TASK_MIN_VERSION
```

For a tool already installed below the proposed minimum, use `winget upgrade` with the same arguments instead of `winget install`.

Close and reopen PowerShell after installation. If using the VS Code integrated terminal, restart VS Code so its terminals inherit the updated PATH.

### 5. Configure command lookup

For this x64 Windows setup using the default WinGet user installation paths, run this block once in each new PowerShell session before checking versions or running project commands:

```powershell
$wingetPackagesDirectory = Join-Path $env:LOCALAPPDATA 'Microsoft\WinGet\Packages'
$kindBinDirectory = Join-Path $wingetPackagesDirectory 'Kubernetes.kind_Microsoft.Winget.Source_8wekyb3d8bbwe'
$helmBinDirectory = Join-Path $wingetPackagesDirectory 'Helm.Helm_Microsoft.Winget.Source_8wekyb3d8bbwe\windows-amd64'
$taskBinDirectory = Join-Path $wingetPackagesDirectory 'Task.Task_Microsoft.Winget.Source_8wekyb3d8bbwe'
$kubectlBinDirectory = Join-Path $wingetPackagesDirectory 'Kubernetes.kubectl_Microsoft.Winget.Source_8wekyb3d8bbwe'
$env:Path = "$kindBinDirectory;$helmBinDirectory;$taskBinDirectory;$kubectlBinDirectory;$env:Path"
```

This gives the installed tools priority over other copies, including the kubectl bundled with Docker Desktop. These PATH changes apply to the current session and programs started from it.

Verify the selected tool versions:

```powershell
kind version
kubectl version --client --output=yaml
helm version --short
task --version
```

Compare the reported versions with `versions.env`.

## macOS setup

Validation status: pending execution on macOS.

Use a macOS release that meets both the [Homebrew requirements](https://docs.brew.sh/Installation) and the [Docker Desktop requirements](https://docs.docker.com/desktop/setup/install/mac-install/).

Use a native terminal for your Mac architecture: `arm64` for Apple Silicon or `x86_64` for Intel. Homebrew currently classifies Intel Macs as Tier 3; review its requirements before proceeding.

### 1. Install Homebrew

Check whether Homebrew is already available:

```sh
brew --version
```

If it is unavailable, run the official installer:

```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Follow the installer's Next steps instructions, including the shell configuration needed to make `brew` available in new terminals.

### 2. Install Docker Desktop and the command-line tools

Homebrew installs the stable versions currently available in its catalog. Compare the installed versions with the proposed minimums in `versions.env`.

```sh
brew update
brew install --cask docker-desktop
brew install kind kubernetes-cli helm go-task
eval "$(brew shellenv)"
open -a Docker
```

Complete the Docker Desktop first-launch setup and wait for its engine to start.

To update existing installations below the proposed minimums, use:

```sh
brew upgrade kind kubernetes-cli helm go-task
brew upgrade --cask --greedy docker-desktop
```

### 3. Check command lookup

In each new terminal, ensure Homebrew is configured using the installer's shell setup instructions. Check which executables are selected:

```sh
eval "$(brew shellenv)"
command -v docker kind kubectl helm task
```

Continue with [Verification](#verification) and record the actual versions tested.

## Linux setup

Validation status: pending execution on Linux.

The Docker installation example below targets Ubuntu 24.04 LTS with systemd. The command-line tool downloads support AMD64 and ARM64.

For other distributions, follow the [Docker Engine installation instructions](https://docs.docker.com/engine/install/) for your distribution, then continue with the command-line tools below. Ensure `curl`, CA certificates, `tar`, `gzip` and GNU coreutils are available.

Run these commands from your normal user account, using `sudo` where shown.

### 1. Install Docker Engine on Ubuntu

If an existing Docker Engine already meets the minimum in `versions.env` and is accessible to your user, continue to step 2.

For an existing machine, review the [official Ubuntu installation prerequisites](https://docs.docker.com/engine/install/ubuntu/) and resolve conflicting Docker packages before proceeding.

The following block configures the official stable repository and installs the available stable packages:

```sh
(
set -eu
sudo apt-get update
sudo apt-get install -y ca-certificates curl tar gzip
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
. /etc/os-release
sudo tee /etc/apt/sources.list.d/docker.sources >/dev/null <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $VERSION_CODENAME
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl enable --now docker
sudo usermod -aG docker "$(id -un)"
)
```

Membership in the `docker` group grants root-level access through Docker. See the [Docker post-installation documentation](https://docs.docker.com/engine/install/linux-postinstall/).

Log out and log back in to activate the group membership. Then open a terminal and return to the cloned repository root.

### 2. Install the command-line tools

Run the entire block from the repository root. It reads the declared versions from `versions.env` and installs the executables into your user's `~/.local/bin` directory.

```sh
(
set -eu
. ./versions.env
case "$(uname -m)" in
    x86_64) tools_arch=amd64 ;;
    aarch64|arm64) tools_arch=arm64 ;;
    *) printf '%s\n' 'Unsupported CPU architecture.' >&2; exit 1 ;;
esac
tools_download_dir=$(mktemp -d)
mkdir -p "$HOME/.local/bin" "$tools_download_dir/helm" "$tools_download_dir/task"
curl -fL "https://kind.sigs.k8s.io/dl/v$KIND_MIN_VERSION/kind-linux-$tools_arch" -o "$tools_download_dir/kind"
install -m 0755 "$tools_download_dir/kind" "$HOME/.local/bin/kind"
curl -fL "https://dl.k8s.io/release/v$KUBECTL_MIN_VERSION/bin/linux/$tools_arch/kubectl" -o "$tools_download_dir/kubectl"
install -m 0755 "$tools_download_dir/kubectl" "$HOME/.local/bin/kubectl"
curl -fL "https://get.helm.sh/helm-v$HELM_MIN_VERSION-linux-$tools_arch.tar.gz" -o "$tools_download_dir/helm.tar.gz"
tar -xzf "$tools_download_dir/helm.tar.gz" -C "$tools_download_dir/helm"
install -m 0755 "$tools_download_dir/helm/linux-$tools_arch/helm" "$HOME/.local/bin/helm"
curl -fL "https://github.com/go-task/task/releases/download/v$TASK_MIN_VERSION/task_linux_$tools_arch.tar.gz" -o "$tools_download_dir/task.tar.gz"
tar -xzf "$tools_download_dir/task.tar.gz" -C "$tools_download_dir/task"
install -m 0755 "$tools_download_dir/task/task" "$HOME/.local/bin/task"
)
export PATH="$HOME/.local/bin:$PATH"
hash -r
```

The downloads remain in the temporary directory created by `mktemp`.

### 3. Configure command lookup

For new interactive terminals, add the following line once to your shell configuration file: `~/.bashrc` for Bash or `~/.zshrc` for Zsh.

```sh
export PATH="$HOME/.local/bin:$PATH"
```

Reopen your terminal and check which executables are selected:

```sh
command -v docker kind kubectl helm task
```

Continue with [Verification](#verification) and record the actual versions tested.

## Verification

After completing your OS-specific setup and configuring command lookup, run:

```sh
docker version
docker info --format '{{.OSType}}'
kind version
kubectl version --client --output=yaml
helm version --short
task --version
```

These commands also work in Windows PowerShell.

Compare the reported versions with the minimums in `versions.env`. For Docker, check the server's Engine version; on Windows and macOS, also check the Docker Desktop version.

The Docker client must successfully reach the server, and the second command must print `linux`.

On Windows, also verify WSL:

```powershell
wsl --version
wsl --list --verbose
```

Compare the WSL application version with `WSL_MIN_VERSION`. Confirm that the `docker-desktop` distribution uses WSL version `2`.

Once a Kubernetes cluster is created, kubectl must remain within one minor release of the API server, following the [Kubernetes compatibility policy](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/).

Record the operating system, architecture, tool versions, tester, date and result when validating this guide. An unexecuted procedure must remain marked as pending.

## Troubleshooting

- **Command not found or unexpected version:** repeat the command lookup setup for your OS. On Windows, use `Get-Command kubectl -All` to identify competing kubectl installations.
- **Docker server unavailable:** start Docker Desktop and wait for its engine. On Ubuntu, check the service with `sudo systemctl status docker`.
- **Docker permission denied on Linux:** confirm Docker group membership and log out and back in after changing it.

## Validation record

| Platform | Architecture | Tester | Date | Result |
| --- | --- | --- | --- | --- |
| Windows | amd64 | Bittencourt | 2026-11-01 | Tool version checks and Docker API access passed on an existing installation. |
| macOS | Pending | Pending | Pending | Runtime validation pending. |
| Linux | Pending | Pending | Pending | Runtime validation pending. |

Observed Windows versions: Docker Desktop 4.97.0, Docker Engine 29.9.2, kind 0.33.0, kubectl 1.37.1, Helm 4.3.1, Go Task 3.54.0 and WSL 2.7.13.0.

All 11 shell blocks passed a Bash syntax check on Windows. All 9 PowerShell blocks parsed without errors. These checks do not replace execution on macOS or Linux.

A teammate review and runtime validation on a second operating system are still required.