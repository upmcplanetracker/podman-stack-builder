# Daily Podman Stack Builder

A GitHub Actions workflow that automatically compiles the latest upstream releases of the [Podman](https://podman.io) container stack into native `.deb` packages for **Ubuntu 26.04**.

## What It Builds

| Package | Upstream Repo | Description |
|---------|---------------|-------------|
| `podman` | [containers/podman](https://github.com/containers/podman) | Container engine |
| `crun` | [containers/crun](https://github.com/containers/crun) | OCI runtime (default for Podman) |
| `conmon` | [containers/conmon](https://github.com/containers/conmon) | Container monitor |
| `netavark` | [containers/netavark](https://github.com/containers/netavark) | Network stack |
| `aardvark-dns` | [containers/aardvark-dns](https://github.com/containers/aardvark-dns) | DNS server for rootless containers |
| `skopeo` | [containers/skopeo](https://github.com/containers/skopeo) | Container image utility |
| `fuse-overlayfs` | [containers/fuse-overlayfs](https://github.com/containers/fuse-overlayfs) | FUSE overlay driver |

## How It Works

1. **Scheduled builds** — Runs daily at 08:00 UTC (4 AM EST) via cron.
2. **Manual trigger** — You can also click **Run workflow** from the Actions tab.
3. **Smart skipping** — Checks if a release already exists for the current upstream version. If yes, that package is skipped. Only new versions trigger a build.
4. **Native compilation** — Each package compiles inside an `ubuntu:26.04` container to match your target environment exactly.
5. **GitHub Releases** — Successful builds are published as GitHub Releases with downloadable `.deb` assets.

## Installing

Download the `.deb` for the package you want from the [Releases](../../releases) page, then install:

```bash
sudo dpkg -i podman_5.x.x-0local1_amd64.deb

# If apt complains about missing dependencies:
sudo apt-get install -f
```

Or install all packages at once:

```bash
sudo dpkg -i *.deb
sudo apt-get install -f
```

## APT Pinning (Recommended)

To prevent Ubuntu's older packages from overwriting these builds on `apt upgrade`, create a pinning file:

```bash
sudo tee /etc/apt/preferences.d/podman-stack <<'EOF'
Package: podman crun conmon netavark aardvark-dns skopeo fuse-overlayfs
Pin: origin ""
Pin-Priority: 1001
EOF
```

> **Note:** Since these packages use epoch `1:` (e.g., `1:5.4.0-0local1`), they will naturally outrank Ubuntu's un-epoch'd packages even without pinning. Pinning is just extra insurance.

## Architecture

- **Matrix strategy** — Each package builds independently. If one fails, the others continue.
- **Epoch pinning** — Packages are built with `--epoch 1` so APT always prefers them over distro versions.
- **Dependency mapping** — The `podman` package declares dependencies on `crun`, `conmon`, `netavark`, `aardvark-dns`, and `fuse-overlayfs` so `apt` resolves them automatically.

## Requirements

- Ubuntu 26.04 (or compatible) amd64 system
- `dpkg` / `apt` for package management
