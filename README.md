# DCP-o-matic APT Repository for Debian 13

Unofficial APT repository for [DCP-o-matic](https://dcpomatic.com) on Debian 13 (Trixie), automatically maintained via GitHub Actions.

The workflow checks the DCP-o-matic download pages daily and publishes new versions automatically. Two channels are available:

| Channel | Source | Description |
| --- | --- | --- |
| `stable` | `Stable release:` on [dcpomatic.com/download](https://dcpomatic.com/download) | Recommended stable version |
| `testing` | `Test release:` on [dcpomatic.com/test-download](https://dcpomatic.com/test-download) | Development version |

> **Note:** This project is not affiliated with the DCP-o-matic author.

## Installation

### 1. Install the public key

```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://tiagocasalribeiro.github.io/dcpomatic-apt/dcpomatic-apt.asc \
  | sudo gpg --dearmor -o /etc/apt/keyrings/dcpomatic-apt.gpg
```

### 2. Add the desired channel

**Stable** (recommended):

```bash
sudo tee /etc/apt/sources.list.d/dcpomatic-stable.sources << 'EOF'
Types: deb
URIs: https://tiagocasalribeiro.github.io/dcpomatic-apt/
Suites: stable
Components: main
Signed-By: /etc/apt/keyrings/dcpomatic-apt.gpg
EOF
```

**Testing** (development):

```bash
sudo tee /etc/apt/sources.list.d/dcpomatic-testing.sources << 'EOF'
Types: deb
URIs: https://tiagocasalribeiro.github.io/dcpomatic-apt/
Suites: testing
Components: main
Signed-By: /etc/apt/keyrings/dcpomatic-apt.gpg
EOF
```

### 3. Update and install

```bash
sudo apt update
sudo apt install dcpomatic
```

## Using both channels simultaneously

Since both channels contain a package with the same name (`dcpomatic`), if you enable both, `apt` will pick the most recent version by default (testing).

To prioritise a specific channel, create a pinning file:

```bash
sudo tee /etc/apt/preferences.d/dcpomatic << 'EOF'
Package: dcpomatic
Pin: release a=stable
Pin-Priority: 900

Package: dcpomatic
Pin: release a=testing
Pin-Priority: 500
EOF
```

## Updating

```bash
sudo apt update && sudo apt upgrade
```

To install a specific version:

```bash
sudo apt install dcpomatic/stable     # stable version
sudo apt install dcpomatic/testing    # testing version
```

## Troubleshooting

| Symptom | Solution |
| --- | --- |
| `404 Not Found` during `apt update` | GitHub Pages may not be ready yet; wait a few minutes |
| `NO_PUBKEY` | Re-run step 1 of the installation |
| Package not found | Run `sudo apt update` first |
| `404` when installing | Clear the cache: `sudo rm -rf /var/lib/apt/lists/* && sudo apt update` |

## Repository structure

```
dcpomatic-apt/
├── dcpomatic-apt.asc
├── stable-version.txt
├── testing-version.txt
├── pool/
│   ├── stable/main/d/dcpomatic/
│   │   └── dcpomatic_<version>_amd64.deb
│   └── testing/main/d/dcpomatic/
│       └── dcpomatic_<version>_amd64.deb
└── dists/
    ├── stable/
    │   ├── InRelease, Release, Release.gpg
    │   └── main/binary-amd64/Packages(.gz)
    └── testing/
        ├── InRelease, Release, Release.gpg
        └── main/binary-amd64/Packages(.gz)
```

## Credits

DCP-o-matic © Carl Hetherington, licensed under GNU GPL.
<https://dcpomatic.com>
