# Role: docker

## Purpose
Set up a secure, production-ready Docker environment for running n8n and its dependencies.
- Installs Docker Engine and Compose plugin from official repositories, with version pinning
- Configures Docker daemon for Unix socket only (optionally supports TLS-secured TCP socket)
- Ensures only the n8n user is in the docker group
- Pulls official n8nio/n8n and postgres images
- Provides Compose and daemon templates for secure, non-root, least-privilege container operation
- Documents image signature verification and vulnerability scanning in CI

## Variables (from group_vars/vars.yml)
- `docker_engine_version`: Docker Engine version to install (e.g., 5:24.0.7-1~ubuntu.24.04~jammy)
- `docker_compose_plugin_version`: Compose plugin version (e.g., 2.27.1-1~ubuntu.24.04~jammy)
- `containerd_version`: Containerd version (e.g., 1.6.28-1)

## Templates
- `daemon.json.j2`: Hardened Docker daemon config (Unix socket only by default, TLS optional)
- `docker-compose.snippet.yml.j2`: Example Compose snippet for secure, non-root containers

## Security Best Practices
- Only n8n user in docker group
- Daemon listens on Unix socket only (TCP/TLS optional, disabled by default)
- Service containers run as non-root (user: "1000:1000"), drop all Linux capabilities, use read-only rootfs, seccomp/AppArmor
- Compose services use `restart: unless-stopped`
- Image signature verification and vulnerability scanning (cosign, trivy, dockle) should be performed in CI

## Handlers
- Restart Docker if daemon config changes

## Idempotence
All tasks are idempotent and safe to re-run.

## Usage
Include the role in your playbook:
```yaml
- hosts: all
  roles:
    - role: docker
```
