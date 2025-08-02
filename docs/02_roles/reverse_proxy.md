# Role: reverse_proxy

## Purpose
Install and configure a secure reverse proxy for n8n, supporting both Caddy (default) and Nginx (optional).

## Features
- **Caddy (default):**
  - Installs from official repo, runs as a system service
  - Proxies {{ n8n_domain }} to http://localhost:5678
  - Automatic Let's Encrypt certificates and HTTP→HTTPS redirection
  - Access logs in /var/log/caddy/n8n_access.log (common log format)
  - Log rotation for Caddy logs
- **Nginx (optional):**
  - Installs from distro repo
  - Hardened server block with strong TLS, OCSP, HSTS, DoS mitigation, and WAF (ModSecurity) integration
  - Proxies HTTPS to http://localhost:5678
  - Certbot for automatic certificate management and renewal
  - Log rotation for Nginx logs

## Variables (from group_vars/vars.yml)
- `reverse_proxy_type`: 'caddy' (default) or 'nginx'
- `n8n_domain`: Domain for n8n (e.g., n8n.majitask.fun)
- `n8n_admin_email`: Email for Let's Encrypt notifications (e.g., example@example.com)

## Templates
- `Caddyfile.j2`: Caddy configuration
- `n8n.conf.j2`: Hardened Nginx config
- `logrotate-caddy`, `logrotate-nginx`: Log rotation configs

## Handlers
- Reload Caddy or Nginx after config/cert changes

## Idempotence
All tasks are idempotent and safe to re-run. Switching proxy type is supported by changing `reverse_proxy_type`.

## Usage
Include the role in your playbook:
```yaml
- hosts: all
  roles:
    - role: reverse_proxy
```

## Certbot Automation (Nginx only)
- Installs Certbot and the Nginx plugin
- Obtains and renews Let's Encrypt certificates for your n8n domain
- Sets up a cron job to renew certificates and reload Nginx

## References
- [n8n Docker + Nginx + SSL](https://docs.n8n.io/hosting/installation/server-setups/docker-compose/)
- [Certbot for Nginx](https://certbot.eff.org/instructions?ws=nginx&os=ubuntufocal)

---
This role ensures your n8n instance is securely exposed with automated SSL and best-practice reverse proxy configuration.
