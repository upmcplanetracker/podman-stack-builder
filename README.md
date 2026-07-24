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
| `passt` | [passt.top/passt](https://passt.top/passt/) | User-mode networking for rootless containers |
| `containers-common` | [containers/common](https://github.com/containers/common) | Shared config files (`/etc/containers/*`) |
| `containers-storage` | [containers/storage](https://github.com/containers/storage) | Container storage library and CLI |

## How It Works

1. **Scheduled builds**: Runs daily at 08:00 UTC via cron.
2. **Manual trigger**: You can also click **Run workflow** from the Actions tab.
3. **Smart skipping**: Checks if a release already exists for the current upstream version. If yes, that package is skipped. Only new versions trigger a build.
4. **Native compilation**: Each package compiles inside an `ubuntu:26.04` container to match your target environment exactly.
5. **GitHub Releases**: Successful builds are published as GitHub Releases with downloadable `.deb` assets.

## Installing

Download the `.deb` for the package you want from the [Releases](../../releases) page, then install:

```bash
# example
sudo dpkg -i podman_5.x.x-0local1_amd64.deb

# If apt complains about missing dependencies:
sudo apt-get install -f
```

### Install order for multiple packages

If you're installing everything fresh, use this order so dependencies resolve correctly:

```bash
sudo dpkg -i   containers-common_*.deb   containers-storage_*.deb   conmon_*.deb   crun_*.deb   netavark_*.deb   aardvark-dns_*.deb   fuse-overlayfs_*.deb   passt_*.deb   skopeo_*.deb   podman_*.deb

sudo apt-get install -f
```

Or install all packages at once:

```bash
sudo dpkg -i *.deb
sudo apt-get install -f
```

### Package dependencies

| Package | Depends on |
|---------|-----------|
| `podman` | `conmon`, `crun`, `netavark`, `aardvark-dns`, `fuse-overlayfs`, `containers-common`, `containers-storage` |
| `skopeo` | `containers-common` |
| Everything else | No custom dependencies declared |

> **Note about `skopeo`:** The upstream `skopeo` package ships `/etc/containers/policy.json`, which conflicts with Ubuntu's `golang-github-containers-common` package. This build strips that file from `skopeo` and instead declares `containers-common` as a dependency, so the config is provided cleanly by one package.

## APT Pinning (Recommended)

To prevent Ubuntu's older packages from overwriting these builds on `apt upgrade`, create a pinning file:

```bash
sudo tee /etc/apt/preferences.d/podman-stack <<'EOF'
Package: podman crun conmon netavark aardvark-dns skopeo fuse-overlayfs passt containers-common containers-storage
Pin: origin ""
Pin-Priority: 1001
EOF
```

> **Note:** Since these packages use epoch `1:` (e.g., `1:5.4.0-0local1`), they will naturally outrank Ubuntu's un-epoch'd packages even without pinning. Pinning is just extra insurance.

## Architecture

- **Matrix strategy** — Each package builds independently. If one fails, the others continue.
- **Epoch pinning** — Packages are built with `--epoch 1` so APT always prefers them over distro versions.
- **Dependency mapping** — The `podman` package declares dependencies on all companion tools so `apt` resolves them automatically.

## Requirements

- Ubuntu 26.04 (or compatible) amd64 system
- `dpkg` / `apt` for package management
