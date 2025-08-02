---
# n8n Ansible Deployment Blueprint

This blueprint provides a production-grade, security-hardened, and automation-focused roadmap for deploying n8n with Ansible. All recommendations are mapped to actionable roles/playbooks and cite authoritative sources.

## Quick Reference: Roles & Playbooks
| Role/Playbook         | Purpose                                      |
|----------------------|----------------------------------------------|
| system_update        | OS updates, hardening, time/locale           |
| user_access          | User creation, SSH, sudo, password mgmt      |
| docker               | Docker Engine & Compose, daemon security     |
| postgresql           | DB install, user, security, backups          |
| n8n                  | n8n Docker Compose, env, admin, health       |
| reverse_proxy        | NGINX/Caddy, SSL, hardening, renewal         |
| backup               | Data, DB, config backups, rotation           |
| ufw                  | Firewall config, logging                     |
| postfix              | SMTP relay, SPF/DKIM/DMARC, TLS              |
| monitoring           | System/app health, alerts, log aggregation   |
| hardening            | Fail2ban, intrusion detection, vulnerability |
| dns_locale           | DNS, locale, timezone                        |
| vault                | Secrets management, repo structure           |

---

## 0. System Update & Host Hardening
- Use a minimal OS (Ubuntu Server/Debian). Remove unnecessary packages/services ([virtualizationhowto.com](https://www.virtualizationhowto.com/)).
- Enable unattended-upgrades for security patches ([sliplane.io](https://sliplane.io/)).
- Set timezone to Europe/Prague; install ntp/chrony for accurate time.
- SSH: Disable password auth (prefer keys), restrict root login, use non-default port if policy allows ([virtualizationhowto.com](https://www.virtualizationhowto.com/)).
- Use Fail2ban for SSH brute-force protection. Consider MFA if supported.
- Regularly back up the entire server or use VM snapshots.

## 1. User & Access Management
- Create a dedicated `n8n` user (minimal sudo, only for Docker/maintenance).
- Provision SSH keys from Ansible Vault; support multiple admins.
- Store fallback password in Vault; auto-update on deploy.
- Sudoers: Use NOPASSWD only where needed, log all commands.

## 2. Docker & Docker Compose
- Install Docker Engine & Compose from official repos; pin versions.
- Docker daemon: Listen on Unix socket only; secure remote API with TLS if needed.
- Add `n8n` user to `docker` group; restrict others.
- Pull images from official sources; verify signatures; scan with Trivy/dockle.
- Drop unnecessary Linux capabilities; use read-only rootfs, seccomp, AppArmor.
- Run containers as non-root (user: "1000:1000").
- Add `restart: unless-stopped` to Compose for resilience.

## 3. PostgreSQL Database
- Deploy via Docker Compose or managed service; persist data with Docker volume.
- Set env vars (POSTGRES_USER, POSTGRES_PASSWORD, POSTGRES_DB) from Vault.
- Use `md5` or `scram-sha-256` auth; restrict listen_addresses to localhost/private net.
- For remote: enforce SSL, restrict IPs, use hostssl in pg_hba.conf ([postgresql.org](https://www.postgresql.org/docs/current/auth-pg-hba-conf.html)).
- Create dedicated DB user for n8n (least privilege).
- Logical backups via `pg_dump`; include in weekly backup rotation.

## 4. n8n Deployment via Docker Compose
- Use latest stable n8n image (pin version tag).
- Set env: N8N_BASIC_AUTH_USER/PASS (Vault), GENERIC_TIMEZONE=Europe/Prague, N8N_HOST, N8N_PORT, WEBHOOK_URL, DB vars.
- Preconfigure admin user (N8N_USER_MANAGEMENT_DISABLED=false, seed via env/init script).
- Mount `/home/node/.n8n` to Docker volume.
- Disable tunnel in production ([docs.n8n.io](https://docs.n8n.io/)).
- Add HEALTHCHECK in Compose or use n8n health endpoint.

## 5. Reverse Proxy & SSL Termination
- **Caddy (recommended):** Auto-SSL, simple config, HTTP→HTTPS, logs ([sliplane.io](https://sliplane.io/)).
- **Nginx:**
  - Disable server_tokens, limit buffer sizes, restrict HTTP methods, use WAF (ModSecurity), rotate logs, OCSP stapling, HSTS, strong TLS ciphers ([acunetix.com](https://www.acunetix.com/)).
- Cert renewal: Use Caddy or Certbot+cron/systemd; reload proxy after renewal.

## 6. Backups & Disaster Recovery
- Weekly backups: n8n data volume, PostgreSQL data, app configs, proxy configs, OS/user config.
- Use Ansible role/cron to create tar.gz archives; store under `/var/backups/n8n/YYYYMMDD/`.
- Use `docker cp` or `docker run --rm -v volume:/data ...` for volume backups.
- Rotate: keep last 2+ copies.
- (Optional) Off-site: rclone/restic to S3/B2 ([sliplane.io](https://sliplane.io/)).
- Nightly `pg_dump`; test restores regularly.

## 7. Firewall Configuration (UFW)
- Install/enable UFW; default deny incoming, allow outgoing.
- Open only required ports: SSH (22/custom), HTTP (80), HTTPS (443), n8n internal if needed ([sliplane.io](https://sliplane.io/)).
- Optionally restrict SSH to specific IPs/VPN.
- Enable UFW logging.

## 8. Email Configuration (Postfix & Relay)
- Install Postfix (satellite mode) to send via external SMTP relay.
- Configure relay (Gmail, SendGrid, Mailgun); store creds in Vault.
- Publish SPF, DKIM, DMARC records (provide DNS entries for admin) ([easydmarc.com](https://easydmarc.com/)).
- Enforce TLS for SMTP; verify CA certs.
- Monitor mail queues (Logwatch or similar).

## 9. Monitoring & Alerts
- Use Prometheus/node_exporter/cAdvisor or Monit for system/container health.
- Monitor n8n health endpoint.
- Email alerts for failed backups/cron jobs.
- Centralize logs (Loki, syslog, Filebeat, Vector).

## 10. Security Hardening
- Install Fail2ban for SSH and other services ([sliplane.io](https://sliplane.io/)).
- Intrusion detection (AIDE, Wazuh).
- Automatic security updates (see Section 0).
- Combine UFW with WAF (ModSecurity) for proxy.
- Audit Docker host (docker bench security, OpenSCAP).
- Regular vulnerability scans (Lynis, OpenVAS).

## 11. DNS & Locale Configuration
- Set A/CNAME for `n8n.majitask.fun` to server IP; MX for mail relay; SPF/DKIM/DMARC.
- System locale: `en_US.UTF-8` or `cs_CZ.UTF-8`; timezone: Europe/Prague.

## 12. Ansible Project & Vault
- Structure:
  - `inventory/` for hosts
  - `group_vars/`, `host_vars/` for config
  - `roles/` for each component
  - `playbooks/` for orchestration
  - `files/`, `templates/` for static/Jinja2
- Idempotence: Test with Molecule, ansible-lint, CI/CD.
- Vault: Store all secrets (DB, SMTP, SSH, basic auth) encrypted; keep `vault_password_file` outside VCS or inject via CI.
- Use Git with clear commit messages and branching.
- Document setup, variables, and usage in README.md.

## 13. Scalability & High Availability (Future)
- Monitor resource usage; scale up server as needed ([sliplane.io](https://sliplane.io/)).
- For high traffic: multiple n8n behind load balancer, shared Postgres/Redis.
- Use central secrets store (e.g., HashiCorp Vault) for multi-instance.
- Disaster recovery: mirror stack in another region, replicate DB, off-site backups.

---

**Implementing this blueprint will position your n8n Ansible deployment among the top tier of self-hosted automation platforms. Every improvement is grounded in proven, cited best practices.**
Objective	Implementation & Rationale
Dedicated system user	Keep the n8n service (and associated Docker containers) running under a dedicated system user with limited privileges. Create this user via Ansible and assign it only the rights required to manage Docker Compose.
sudo policy	Use minimal privilege; only allow the n8n user to run necessary administrative tasks (e.g., Docker commands) via sudo. Make sure sudoers uses NOPASSWD only where appropriate and logs all commands.
SSH keys	Provision SSH public keys to the n8n user from Ansible Vault; avoid embedding private keys in the repository. Support multiple keys for each administrator to allow revocation.
Vaulted passwords	If password login is needed (e.g., as fallback), store the credentials securely in Ansible Vault and automatically update the user’s password during deployment.
2  Docker & Docker Compose
Objective	Implementation & Rationale	Sources
Install Docker Engine & Compose	Use the official Docker repositories to install Docker Engine and the docker compose plugin. Pin the versions to avoid surprises.	
Docker daemon hardening	- Configure the Docker daemon to listen on a local Unix socket only; if remote API is required, secure it with TLS and client authentication
virtualizationhowto.com
. - Use a dedicated group (e.g., docker), add the n8n user to this group, and restrict other users.	Docker‑security article advises securing Docker daemon and using TLS
virtualizationhowto.com
.
Container security	- Pull images from official sources and verify signatures. - Scan images for vulnerabilities using tools like Trivy or dockle during CI. - Drop unnecessary Linux capabilities, use read‑only root filesystems where possible, and apply seccomp and AppArmor profiles to restrict system calls
virtualizationhowto.com
. - Use user: "1000:1000" or user: ${UID}:${GID} in Docker Compose to avoid running containers as root.	
Service restart policy	Add a restart: unless-stopped (or always) directive in Compose to ensure n8n containers automatically restart after crashes or system reboots.	
3  PostgreSQL Database
Objective	Implementation & Rationale	Sources
Deploy via Docker Compose	Run PostgreSQL as a separate container or managed service. Persist data via a dedicated Docker volume and set environment variables (e.g., POSTGRES_USER, POSTGRES_PASSWORD, POSTGRES_DB) from Ansible Vault.	
Secure connection settings	- Set POSTGRES_HOST_AUTH_METHOD=trust only during initial bootstrap; use md5 authentication with strong passwords or scram-sha-256 for final configuration. - For remote connections, configure pg_hba.conf to only allow SSL (hostssl) connections
postgresql.org
and restrict allowed IP ranges. - Set listen_addresses = '127.0.0.1' (or a private network range) if n8n and PostgreSQL run on the same host; avoid exposing port 5432 publicly.	
TLS encryption	If using remote connections (e.g., DB on separate host), create self‑signed or CA‑signed certificates and configure Postgres to enforce TLS with ssl=on and ssl_ciphers.	
Database user	Create a dedicated DB user for n8n with minimal privileges (e.g., only on the n8n database). Store credentials in Ansible Vault.	
Automated backups	Implement regular logical backups (via pg_dump) and include the DB volume in the weekly backup rotation (see Section 6).	
4  n8n Deployment via Docker Compose
Objective	Implementation & Rationale	Sources
Run the latest stable n8n image	Pull from the official n8nio/n8n repository and specify an explicit version tag to avoid unexpected updates.	
Environment variables	- Set N8N_BASIC_AUTH_USER and N8N_BASIC_AUTH_PASSWORD to protect the editor UI with HTTP Basic Auth (use Ansible Vault for credentials). - Define GENERIC_TIMEZONE=Europe/Prague to ensure workflows execute in the correct time zone. - Provide N8N_HOST, N8N_PORT (e.g., 5678), WEBHOOK_URL, and DB connection details.	
Admin user creation	Pre‑configure the n8n admin account using N8N_USER_MANAGEMENT_DISABLED=false and seed a user via environment variables or initialization script. After the first login, enforce password change.	
Data persistence	Mount /home/node/.n8n to a Docker volume to preserve workflows and credentials.	
Disable tunnel in production	Avoid using --tunnel for production; the official docs note that this exposes your instance publicly and is only meant for local testing
docs.n8n.io
.	
Health checks	Include a HEALTHCHECK directive in Docker Compose or rely on n8n’s built‑in health endpoint to monitor container state; configure restart policies accordingly.	
5  Reverse Proxy & SSL Termination

Instead of manually configuring Nginx for certificates, consider a modern proxy that automates certificate acquisition (e.g., Caddy) or ensure Nginx is hardened.
Objective	Implementation & Rationale	Sources
Caddy (recommended)	Caddy automatically obtains and renews Let’s Encrypt certificates, simplifies configuration, and enforces HTTPS by default. Example Caddyfile:	

n8n.majitask.fun {
    reverse_proxy localhost:5678
    log {
        output file /var/log/caddy/n8n_access.log
        format single_field common_log
    }
}

Deploy via an official caddy Docker container or install on host. Caddy uses acme-server to manage certificates and handles HTTP→HTTPS redirection
sliplane.io
.

| Nginx hardening | If you prefer Nginx, adopt the following security measures:

    Disable server_tokens to avoid disclosing version information
    acunetix.com
    .

    Limit client buffer sizes (e.g., client_body_buffer_size, client_header_buffer_size) to mitigate DoS attacks
    acunetix.com
    .

    Restrict HTTP methods with limit_except GET POST to disallow unwanted methods like DELETE or PUT
    acunetix.com
    .

    Use a WAF such as ModSecurity or open‑source alternatives; the article suggests integrating ModSecurity with Nginx for additional protection
    acunetix.com
    .

    Manage logs appropriately, configure log rotation, and ensure log files are owned by an unprivileged user
    acunetix.com
    .

    Employ OCSP stapling, HSTS, and strong TLS ciphersuites; test configuration with SSL Labs.

| Automated certificate renewal | Whether using Caddy or Nginx + Certbot, ensure certificates renew automatically and restart or reload the proxy service after renewal. Use a cron job or systemd timer for Certbot. |
6  Backups & Disaster Recovery
Objective	Implementation & Rationale	Sources
Backup scope	Weekly backups should include:	

    Docker volumes for n8n data (/home/node/.n8n) and PostgreSQL data.

    Application configuration files (Ansible inventory, templates).

    Nginx/Caddy configuration.

    OS and user configuration as needed.
    | Automation | Use an Ansible role or cron job to create compressed archives (tar.gz) of volumes and configuration directories; store them under /var/backups/n8n/YYYYMMDD/. Use docker cp or docker run --rm -v volume:/data tar -czf /backup/n8n_data.tar.gz /data to back up volumes. Rotate backups to keep only the last two or more copies. |
    | Off‑site storage | For higher resilience, integrate with a cloud storage provider (e.g., AWS S3, Backblaze B2) using rclone or restic so backups are automatically uploaded off‑site
    sliplane.io
    . Store credentials in Ansible Vault. |
    | PostgreSQL dumps | Use pg_dump to create logical backups; schedule nightly dumps and include them in the weekly backup rotation. Verify dumps by periodically restoring them to a test database. |
    | Test recovery | Regularly verify that backups can be restored; perform disaster recovery drills. |

7  Firewall Configuration (UFW)
Objective	Implementation & Rationale	Sources
Enable UFW	Install and enable ufw; set default policy to deny for incoming traffic and allow for outgoing.	
Allow necessary ports	Open only required ports: SSH (22 or custom), HTTP (80), HTTPS (443), and the n8n internal port (if accessed directly). The Sliplane guide lists typical commands for this configuration
sliplane.io
.	
Restrict source addresses	Optionally restrict SSH access to a specific IP range or VPN. For remote database connections or other services, restrict to internal networks.	
Logging	Enable UFW logging at a suitable level to detect blocked attempts.	
8  Email Configuration (Postfix and Relay)
Objective	Implementation & Rationale	Sources
Install Postfix	Use the satellite configuration during installation to send mail via an external SMTP relay rather than accepting inbound mail.	
External relay	Configure an external relay service (e.g., Gmail, SendGrid, Mailgun) for reliable delivery; set credentials using Ansible Vault.	
SPF/DKIM/DMARC records	- Publish SPF records for your domain to authorize your relay. - Set up DKIM signing; create RSA keys, configure Postfix to sign outgoing mail, and add DKIM public key to DNS. - Publish a DMARC record to specify your policy. These measures improve deliverability and trust
easydmarc.com
.	
TLS for SMTP	Enforce TLS when communicating with the relay; set smtp_tls_security_level=may or encrypt, and verify CA certificates.	
Monitoring	Configure Logwatch or similar to monitor mail queues and send alerts on failures.	
9  Monitoring & Alerts
Objective	Implementation & Rationale
System & service monitoring	Use a monitoring stack (e.g., Prometheus + node exporter + cAdvisor) or simpler tools like Monit. Monitor CPU, memory, disk usage, and Docker container health. Expose metrics via Docker Compose and configure an alerting service (Prometheus Alertmanager or a simple email alert).
Application health checks	n8n exposes a health endpoint; configure checks in your monitoring tool to detect downtime.
Backup & cron job failures	Set up email alerts for backup scripts or cron jobs that fail; integrate with Postfix and monitoring.
Log aggregation	Centralize logs using tools like Loki or a remote syslog server. Use Filebeat or Vector to ship logs from Nginx/Caddy, PostgreSQL, and n8n.
10  Security Hardening
Objective	Implementation & Rationale	Sources
Fail2ban	Install fail2ban to monitor SSH and other service logs and ban IP addresses after repeated failures. This mitigates brute‑force attacks
sliplane.io
.	
Intrusion detection	Deploy a host intrusion detection tool (e.g., AIDE or Wazuh). Monitor critical files and alert on unexpected changes.	
Automatic security updates	Already covered in Section 0
sliplane.io
.	
Firewall & WAF	Combine UFW with Nginx/Caddy WAF rules or ModSecurity
acunetix.com
to block malicious traffic.	
Container runtime security	Use docker bench security or OpenSCAP to audit your Docker host; apply recommended fixes.	
Regular vulnerability scanning	Use Lynis or OpenVAS to scan the server periodically; integrate results into your patch management process.	
11  DNS & Locale Configuration
Objective	Implementation & Rationale
DNS	Use your registrar or DNS provider to create an A or CNAME record for n8n.majitask.fun pointing to your server’s public IP. Configure a separate record for mail (e.g., MX record pointing to your relay) and add SPF/DKIM/DMARC records. Automate DNS updates if possible (e.g., using Cloudflare API) but treat this as out‑of‑scope for the initial playbook.
Locale	Ensure system locale is set to en_US.UTF-8 or cs_CZ.UTF-8 and timezone to Europe/Prague. Configure this via /etc/default/locale and timedatectl.
12  Ansible Project & Vault
Objective	Implementation & Rationale
Repository structure	Use a clearly structured Ansible repo:

    inventory/ for host group definitions.

    group_vars/ and host_vars/ for configuration variables.

    roles/ directory with one role per component (system, docker, postgres, n8n, reverse_proxy, backup, firewall, postfix, monitoring, hardening).

    playbooks/ directory containing separate playbooks or an umbrella site playbook.

    files/ and templates/ for static files and Jinja2 templates.
    | Idempotence & testing | Write tasks to be idempotent; test via Molecule and ansible-lint. Use CI/CD to run tests on pull requests. |
    | Vault password management | Store secrets (DB credentials, SMTP credentials, basic auth, SSH keys) encrypted with Ansible Vault. Place a vault_password_file outside version control or inject the password via CI. |
    | Version control & branching | Use Git with descriptive commit messages and branching strategies (e.g., main for production, develop for features). |
    | Documentation | Maintain a README.md explaining project setup, variables, and how to run playbooks. Provide usage instructions for new administrators. |

13  Scalability & High Availability (optional future stage)
Objective	Implementation & Rationale	Sources
Monitoring resource utilization	Monitor CPU/memory/disk usage of n8n; if the workload grows, allocate more resources or upgrade to a larger server
sliplane.io
.	
Horizontal scaling	For high traffic, deploy multiple n8n instances behind a load balancer. Use a shared PostgreSQL database and shared Redis for queues (n8n’s queue mode).	
Secrets management	In multi‑instance setups, use a central secrets store (e.g., HashiCorp Vault) instead of environment variables.	
Disaster recovery	Mirror the entire stack in another region; replicate the database and store backups off‑site.	
Conclusion

Implementing the above enhancements will position your Ansible deployment of n8n among the top tier of self‑hosted automation platforms. By focusing on security hardening, reliability, maintenance automation, and operational readiness, the final solution will exceed typical guidelines and incorporate best practices from authoritative sources. Each improvement is backed by cited documentation, ensuring the plan is grounded in proven guidance.