# MiniQR on Portainer — hoangthang0106 fork

## 1. Check the host port before deployment

The stack uses **host TCP port 18285** by default, mapped to container port 8080.
This port was chosen to avoid the previously discussed 80, 443, 3000,
3001, 8080, 8088, 8181, 8285, 8293, 9090 and 9443 ports.
**It has NOT been verified live**: a GitHub repository cannot read the live
Docker host's port bindings.

On the **Ubuntu Docker host** (not inside a container), run:

```bash
docker ps --format 'table {{.Names}}\t{{.Ports}}'
sudo ss -ltnp '( sport = :18285 )'
```

If the `ss` command shows a listener, or `docker ps` shows `:18285->`,
**do not use 18285**. Pick a different verified free TCP port and set
`MINIQR_PORT` in the Portainer stack environment.

## 2. Deploy through Portainer from this GitHub fork

1. Open **Portainer → Stacks → Add stack**.
2. **Name**: `miniqr`
3. Choose **Repository** (Git).
4. **Repository URL**: `https://github.com/hoangthang0106/mini-qr`
5. **Repository reference**: `refs/heads/main`
6. **Compose path**: `portainer-stack.yml` (NOT the repo-root `docker-compose.yml`).
7. Deploy the stack.

By default this uses the upstream **public, prebuilt image**
`ghcr.io/lyqht/mini-qr:latest` so it can start immediately even if your
fork has not produced a container image.

Open `http://<YOUR_DOCKER_HOST_IP>:18285` (or the port you selected).

The stack exposes **one container only**, no reverse proxy, PostgreSQL, Redis,
or shared volumes. It does **not** take over host ports 80 or 443.

## 3. Use the image built from your personal fork (optional)

This repository already has `.github/workflows/docker.yml`, which is
configured to build and publish a GHCR image on pushes to `main`.
However, GitHub Actions for a fork may require manual enablement, the
workflow may fail, and a new GHCR package may be private.

Only after the build succeeds and the image is publicly pullable (or Docker
has registry credentials), add this **environment variable** in the Portainer
stack and redeploy:

```text
MINIQR_IMAGE=ghcr.io/hoangthang0106/mini-qr:latest
```

The image name does **not** automatically change when you fork the code.
Until you switch this variable, the fork holds the deployment configuration
but the application image still comes from the original author.

## 4. How to use MiniQR without a redirect

In MiniQR, enter the **final destination URL** directly, e.g.
`https://example.com`. Upload the center logo and customize colors and
styling, then export a PNG or SVG file. The generated **static** QR code
encodes your final URL; scanning it does not call a redirect/short-link service.
Do not encode a dynamic short link if you do not want a redirect.

## 5. Upgrades and rollback

For regular deployment, pin a tested image tag or digest rather than relying
on `:latest`. Store and back up the Stack config in Git.

If the Stack fails with a port allocation error, the selected host port is
occupied. Change only `MINIQR_PORT` to a verified unused port and redeploy.
Do not stop another app to make room.

MiniQR is licensed under GPL-3.0. Retain applicable notices and comply with
the license when distributing modifications.
