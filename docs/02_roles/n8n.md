# Role: n8n

## Purpose
Deploy n8n via Docker Compose with production best practices.
- Uses official n8nio/n8n image, pinned to a specific version
- Configures all environment variables from vars and vault (including HTTP basic auth, DB, and admin settings)
- Preconfigures admin user and disables tunnel in production
- Mounts /home/node/.n8n to a named Docker volume for persistence
- Healthcheck and restart policy for resilience
- Runs container as non-root user and inherits Docker security settings

## Variables (from group_vars/vars.yml)
- `n8n_image_tag`: n8n image version (e.g., 1.103.2)
- `n8n_port`: Port for n8n (default: 5678)
- `n8n_domain`, `n8n_webhook_url`, `n8n_database_type`, `n8n_database_host`, `n8n_database_port`, `n8n_database_user`, `n8n_database_name`

## Secrets (from group_vars/vault.yml)
- `n8n_basic_auth_user`: HTTP basic auth username
- `n8n_basic_auth_password`: HTTP basic auth password
- `n8n_database_password`: Database password

## Templates
- `docker-compose.yml.j2`: Secure Compose file for n8n

## Security Best Practices
- No tunnel enabled in production
- Healthcheck on /healthz endpoint
- restart: unless-stopped
- Non-root user, Docker security options

## Idempotence
All tasks are idempotent and safe to re-run.

## Usage
Include the role in your playbook:
```yaml
- hosts: all
  roles:
    - role: n8n
```
