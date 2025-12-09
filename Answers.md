If I bind the config files exactly as shown in the docker-compose.yml you sent, the project will not find them at runtime. When Coolify deploys a Git repository it copies the repository into a temporary build/deploy directory (for example: `/tmp/coolify/<project-id>/…`). Because Coolify runs the compose from that temporary location, relative host paths such as .`/headscale/config.yaml` no longer point to the same location you expect. As a result, the container’s bind mount fails because the runtime path in the host filesystem does not match the original repository path.

Example:

```
# This will fail in Coolify because the host-relative path is different after Coolify copies the repo
volumes:
  - './headscale/config.yaml:/etc/headscale/config.yaml'
```
After Coolify copies the project, files may actually live at:
```
/tmp/coolify/<project-id>/headplane/headscale/config.yaml
```
## Applied fix:
### Build the config into the image (Best & simplest)

Embed the config into the container image at build time. This avoids host bind mounts entirely and is most robust for CI/CD-like deployments.
```
# Dockerfile
FROM headscale/headscale:stable
COPY config/config.yaml /etc/headscale/config.yaml
```
```
# docker-compose (no host bind required)
services:
  headscale:
    build: .
    # no host bind; config is baked into the image
    volumes: []
```

## About missing: - 'me.tale.headplane.target=headscale'
- This is optional. However added to docker-compose.yml

## About tailscale up --login-server=http://headscale:8080 issue

- Fixed! From now it will show `tailscale up --login-server=<Headscale_URL>`