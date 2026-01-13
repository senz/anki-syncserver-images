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

### Deploying with Portainer

This repository includes `docker-compose.yml` and `stack.env` files for easy deployment via [Portainer](https://www.portainer.io/).

#### Option 1: Deploy from Git Repository (Recommended)

1. In Portainer, go to **Stacks** → **Add stack**
2. Give your stack a name (e.g., `anki-syncserver`)
3. Select **Git Repository**
4. Enter the repository URL: `https://github.com/YOUR_USERNAME/anki-syncserver-image`
5. Set **Compose path** to: `docker-compose.yml`
6. Configure environment variables:
   - Either use **Load variables from .env file** and upload `stack.env`
   - Or set variables individually in Portainer
7. **Required**: Set at least one user credential:
   ```
   SYNC_USER1=username:password
   ```
8. **Optional**: Configure additional settings in `stack.env`:
   - `IMAGE_NAME` - Docker image to use (default: `ghcr.io/senz/anki-syncserver:edge`)
   - `SYNC_PORT` - Host port mapping (default: `8383`)
   - `MAX_SYNC_PAYLOAD_MEGS` - Maximum sync payload size in megabytes (default: `1024`)
   - `PASSWORDS_HASHED` - Set to `1` if using hashed passwords
9. Ensure the data directory exists on the host:
   ```
   /portainer/Files/AppData/Config/anki-syncserver
   ```
   The compose file uses a bind mount to this directory for persistent data storage.
10. Click **Deploy the stack**

#### Option 2: Manual Docker Compose

You can also use the compose file directly:

```bash
# Edit stack.env with your settings (SYNC_USER1 is required)
nano stack.env

# Ensure data directory exists
mkdir -p /portainer/Files/AppData/Config/anki-syncserver

# Deploy
docker compose up -d
```

#### Configuration Notes

- **Port**: Default host port is `8383` (container port is `8080`)
- **Data Storage**: Uses bind mount to `/portainer/Files/AppData/Config/anki-syncserver` for persistent data
- **User Credentials**: Format is `username:password`. At least `SYNC_USER1` must be set
- **Image**: Default image is `ghcr.io/senz/anki-syncserver:edge`. Update `IMAGE_NAME` in `stack.env` to use a different image or version

For more information, see:
- [Portainer Stack Documentation](https://docs.portainer.io/user/docker/stacks/add#option-3-git-repository)
- [Anki Syncserver Documentation](https://github.com/ankitects/anki/tree/main/docs/syncserver)

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
