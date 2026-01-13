# Anki Syncserver Docker Images

This repository contains GitHub Actions workflow for building and publishing [Anki Syncserver](https://github.com/ankitects/anki) Docker images to GitHub Container Registry (GHCR).

## Description

The workflow automatically builds Docker images from the official Anki repository for different variants and architectures:

- **Image variants:**
  - `standard` - standard image based on full distribution
  - `distroless` - minimal image without unnecessary components

- **Supported architectures:**
  - `linux/amd64` (x86_64)
  - `linux/arm64` (ARM64)

All images are built with multi-arch manifest support, allowing automatic architecture selection at runtime.

## Usage

### Building Images

Go to the [workflow page](.github/workflows/build.yml) and run it manually with the following input:

- **version** (required): Version tag from the official Anki repository (e.g., `25.09.2`)

### Cleaning Up Images

A [cleanup workflow](.github/workflows/cleanup.yml) is available to remove old and untagged images from the container registry. This helps reduce storage usage by cleaning up:

- Untagged (dangling) images
- Old run_id-tagged images (older than a configurable threshold, default: 7 days)

The workflow can be run manually via `workflow_dispatch` with the following options:

- **keep_run_id_days** (optional): Keep run_id-tagged images newer than this many days (default: `7`)
- **dry_run** (optional): Dry run mode - list images but do not delete (default: `true`)

The cleanup workflow preserves all version tags, edge tags, and major version tags, only removing temporary build artifacts.

### Using Images

After successful build, images are available in GHCR:

```bash
# Standard image
docker pull ghcr.io/YOUR_USERNAME/anki-syncserver:25.09.2-standard

# Distroless image
docker pull ghcr.io/YOUR_USERNAME/anki-syncserver:25.09.2-distroless
```

### Tag Format

Multi-arch tags (automatically select correct architecture):
- `VERSION-variant-RUN_ID` - specific build
  ```bash
  docker pull ghcr.io/YOUR_USERNAME/anki-syncserver:25.09.2-standard-1234567890
  ```
- `VERSION-variant` - latest for version
  ```bash
  docker pull ghcr.io/YOUR_USERNAME/anki-syncserver:25.09.2-standard
  docker pull ghcr.io/YOUR_USERNAME/anki-syncserver:25.09.2-distroless
  ```
- `edge` / `edge-distroless` - latest builds
  ```bash
  docker pull ghcr.io/YOUR_USERNAME/anki-syncserver:edge
  docker pull ghcr.io/YOUR_USERNAME/anki-syncserver:edge-distroless
  ```
- `MAJOR` / `MAJOR-distroless` - major version tags
  ```bash
  docker pull ghcr.io/YOUR_USERNAME/anki-syncserver:25
  docker pull ghcr.io/YOUR_USERNAME/anki-syncserver:25-distroless
  ```

Architecture-specific tags:
- `VERSION-variant-amd64-RUN_ID` - amd64 build
  ```bash
  docker pull ghcr.io/YOUR_USERNAME/anki-syncserver:25.09.2-standard-amd64-1234567890
  ```
- `VERSION-variant-arm64-RUN_ID` - arm64 build
  ```bash
  docker pull ghcr.io/YOUR_USERNAME/anki-syncserver:25.09.2-standard-arm64-1234567890
  ```

## Development

### Pre-commit Hooks

```bash
pip install pre-commit
pre-commit install
pre-commit run --all-files
```

## Links

- [Official Anki Repository](https://github.com/ankitects/anki)
- [Anki Syncserver Documentation](https://github.com/ankitects/anki/tree/main/docs/syncserver)
- [GitHub Container Registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
- [Build Workflow](.github/workflows/build.yml)
- [Cleanup Workflow](.github/workflows/cleanup.yml)

## License

This repository contains only CI/CD configuration. The license for Anki source code is determined by the official repository.

Container images built by this workflow are licensed under [GNU Affero General Public License v3.0 (AGPL-3.0-only)](LICENSE).
