# hugo-upgrade

**Hugo install / upgrade tool**

Install or upgrade [Hugo](https://gohugo.io/) static site generator from [GitHub releases](https://github.com/gohugoio/hugo/releases).

## Features

- Installs or upgrades Hugo (standard or extended edition)
- Detects edition mismatch (standard ↔ extended) and warns before overwriting
- Verifies SHA256 checksum before installing
- Atomic installation (no broken binary on interruption)
- Supports dry-run and update check without installing
- No external dependencies — uses Python 3 stdlib only

## Requirements

| | |
|---|---|
| OS | RHEL / Rocky / AlmaLinux 8.x\*, 9.x, 10.x · Debian 11+ · Ubuntu 20.04+ |
| Python | 3.8 or later |
| Privilege | `sudo` required for installation |

\* RHEL/Rocky/AlmaLinux 8.x ships Python 3.6 by default. Install Python 3.8 first:
```bash
sudo dnf install python3.8
```

## Installation

```bash
sudo curl -fsSL https://raw.githubusercontent.com/yamk/hugo-upgrade/main/hugo-upgrade \
    -o /usr/local/bin/hugo-upgrade
sudo chmod 755 /usr/local/bin/hugo-upgrade
```

### Allow `sudo hugo-upgrade` without full path

By default, `sudo` does not search `/usr/local/bin`. Add it to the secure path:

```bash
sudo visudo -f /etc/sudoers.d/hugo-upgrade
```

Add this line:

```
Defaults secure_path = /sbin:/bin:/usr/sbin:/usr/bin:/usr/local/bin
```

## Usage

```
hugo-upgrade 0.1  Hugo install / upgrade tool

usage: hugo-upgrade [-h] [-e] [-d DIR] [-V VERSION] [-a ARCH] [-u] [-f] [-n] [-c] [-q] [-v]

options:
  -e, --extended        use Hugo extended edition (includes SCSS/Sass support)
  -d DIR, --dir DIR     installation directory (default: /usr/local/bin)
  -V VERSION, --hugo-version VERSION
                        install a specific Hugo version, e.g. 0.145.0
  -a ARCH, --arch ARCH  target architecture: amd64 or arm64 (default: auto-detect)
  -u, --upgrade         perform install / upgrade (required to make changes)
  -f, --force           reinstall even if already at the target version
  -n, --dry-run         show what would be done without making any changes
  -c, --check           check for updates only (exit 1 if update is available)
  -q, --quiet           suppress informational output (errors are still shown)
  -v, --version         show script version and exit
```

### Examples

```bash
# Install / upgrade standard edition
sudo hugo-upgrade -u

# Install / upgrade extended edition
sudo hugo-upgrade -u -e

# Check for updates (no install)
hugo-upgrade -c -e

# Preview what would happen (dry run)
hugo-upgrade -n -e

# Install a specific version
sudo hugo-upgrade -u -e -V 0.145.0

# Force reinstall (same version)
sudo hugo-upgrade -u -e -f

# Install to a custom directory
sudo hugo-upgrade -u -e -d /opt/bin
```

### Sample output

**Install (first time):**

```
$ sudo hugo-upgrade -u -e
INFO     Installed : not found at /usr/local/bin/hugo
INFO     Checking  : GitHub for latest Hugo release...
INFO     Available : Hugo 0.159.1  (extended, amd64, published 2026-03-26)
INFO     Action    : installing Hugo 0.159.1 (extended)...
INFO     Download  : hugo_extended_0.159.1_linux-amd64.tar.gz
  hugo_extended_0.159.1_linux-amd64.tar.gz  [============================] 100%   18.3 / 18.3 MB
INFO     Verify    : SHA256 checksum...
INFO     Verify    : OK
INFO     Install   : /usr/local/bin/hugo
INFO     Done      : Hugo 0.159.1 (extended) installed successfully.

$ hugo version
hugo v0.159.1+extended linux/amd64 BuildDate=2026-03-26T09:54:15Z VendorInfo=gohugoio
```

**Already up to date:**

```
$ hugo-upgrade -c -e
INFO     Installed : Hugo 0.159.1 (extended)  (/usr/local/bin/hugo)
INFO     Checking  : GitHub for latest Hugo release...
INFO     Available : Hugo 0.159.1  (extended, amd64, published 2026-03-26)
INFO     Status    : already up to date.
```

**Edition mismatch (forgot `-e`):**

```
$ sudo hugo-upgrade -u
INFO     Installed : Hugo 0.159.1 (extended)  (/usr/local/bin/hugo)
INFO     Checking  : GitHub for latest Hugo release...
INFO     Available : Hugo 0.159.1  (standard, amd64, published 2026-03-26)
WARNING  Warning   : installed edition is extended, but standard was requested.
WARNING             Use -f to force the edition change.
```

**Dry run:**

```
$ hugo-upgrade -n -e
INFO     Installed : Hugo 0.159.1 (extended)  (/usr/local/bin/hugo)
INFO     Checking  : GitHub for latest Hugo release...
INFO     Available : Hugo 0.159.1  (extended, amd64, published 2026-03-26)
INFO     Status    : already up to date.
```

### Exit codes

| Code | Meaning |
|------|---------|
| 0 | Success (up to date, or installed successfully) |
| 1 | Update available / not installed (`--check`), or general error |
| 2 | GitHub API rate limit exceeded |
| 3 | Checksum mismatch (security warning) |
| 130 | Interrupted by user |

## License

MIT
