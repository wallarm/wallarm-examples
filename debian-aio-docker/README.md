# Wallarm AIO Docker Image

This is an example of building a custom Wallarm AIO (All-in-One) Docker image. The image includes NGINX with Wallarm WAF module pre-installed.

## Prerequisites

- Docker installed on your system
- Wallarm API credentials (WALLARM_API_TOKEN)
- Optional: Custom NGINX configuration

## Important Notes

Before building the image, please ensure to:
1. Adjust the `entrypoint.sh` script with your specific commands
2. Modify `nginx.conf` and other NGINX configurations according to your settings
3. Set the correct Tarantool domain/IP in the `upstream wallarm_wstore` section of your NGINX configuration

## Building the Image

### Standard Build

```bash
docker build . \
  --build-arg WALLARM_AIO_VERSION=6.2.0 \
  --build-arg NGINX_VERSION="1.28.0" \
  --build-arg WALLARM_WCLI_MODE=filtering \
  -t wallarmcustom:latest
```

### Multi-Architecture Build

```bash
# Setup buildx
docker buildx rm multi-arch || true
docker buildx create \
  --name multi-arch \
  --platform linux/amd64,linux/arm64 \
  --driver docker-container \
  --use

# Run docker build
docker buildx build \
  --platform linux/amd64 \
  -f Dockerfile \
  --load \
  --build-arg WALLARM_AIO_VERSION="6.2.0" \
  --build-arg NGINX_VERSION="1.28.0" \
  --build-arg WALLARM_WCLI_MODE=filtering \
  -t wallarmcustom:latest .
```

## WALLARM_WCLI_MODE Options

The `WALLARM_WCLI_MODE` parameter can be set to one of the following values:

- `filtering`: The node will filter traffic and block malicious requests (default mode)
- `monitoring`: The node will only monitor traffic without blocking
- `off`: The node will be disabled
- `all`: The node will perform all operations including filtering, monitoring, and postanalytics

## Running the Container

```bash
docker run --rm -ti \
  -e WALLARM_API_TOKEN=your_api_token \
  -e WALLARM_API_HOST=api.wallarm.com \
  -e WALLARM_LABELS=your_labels \
  wallarmcustom:latest
```

### Environment Variables

- `WALLARM_API_TOKEN` (Required): Your Wallarm API token for node registration
- `WALLARM_API_HOST` (Optional): Wallarm API host (default: api.wallarm.com)
- `WALLARM_LABELS` (Optional): Labels for the node

## Security

The container runs as an unprivileged user `wallarm` for enhanced security. The NGINX binary has the necessary capabilities to bind to privileged ports.

## Logging

NGINX logs are redirected to stdout/stderr for better container integration:
- Access logs: stdout
- Error logs: stderr
