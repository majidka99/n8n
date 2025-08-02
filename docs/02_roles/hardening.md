# Hardening Role: Security Controls for n8n Host

## Overview
This role applies advanced security controls to the n8n host, including fail2ban, host intrusion detection (AIDE), automatic security updates, web application firewall (ModSecurity for Nginx or Caddy hardening), Docker security auditing, and vulnerability scanning. All tasks are idempotent and only apply changes when needed.

## Features
- Installs and configures fail2ban for SSH, Postfix, and Nginx/Caddy with sensible jail defaults
- Deploys AIDE for host intrusion detection, initializes baseline, and schedules daily checks with email alerts
- Enables automatic security updates (unattended-upgrades)
- Integrates ModSecurity for Nginx or ensures Caddy is hardened
- Runs docker bench security to audit Docker configuration (outputs to /var/log/security/)
- Runs Lynis for vulnerability scanning and schedules daily scans with email alerts
- All features are toggleable via variables; role is idempotent

## Variables
Set in `group_vars/vars.yml`:
```yaml
hardening_enable_aide: true
hardening_enable_modsecurity: true
hardening_enable_docker_bench: true
hardening_enable_lynis: true
hardening_reverse_proxy: "nginx"  # or "caddy"
hardening_alert_email: "admin@n8n.majitask.fun"
```
Secrets in `group_vars/vault.yml` (none required by default):
```yaml
# No secrets required for default hardening role
```

## Usage
Add `hardening` to your playbook roles. The role will:
- Install and configure fail2ban for SSH, Postfix, and Nginx/Caddy
- Deploy and initialize AIDE, schedule daily integrity checks with email alerts
- Enable automatic security updates
- Integrate ModSecurity for Nginx or ensure Caddy is hardened
- Run docker bench security and Lynis, outputting reports to /var/log/security/
- Schedule daily Lynis scans with email alerts

## n8n Security Audit Integration
This role can run the official n8n security audit using the CLI (`n8n audit`). The audit report is saved to `/var/log/n8n/n8n-audit-report.json`. Enable or disable this feature with `hardening_enable_n8n_audit` in `group_vars/vars.yml`.

If the audit cannot be run, a warning is displayed. Review the report for risks related to credentials, database, file system, risky nodes, and instance configuration.

## Extending
- To use Wazuh or OpenVAS, add tasks and variables as needed
- To customize fail2ban jail policies, edit the `jail.local.j2` template
- To integrate scan results with monitoring, point alert emails to your monitoring pipeline

## References
- [Fail2ban](https://www.fail2ban.org/)
- [AIDE](https://aide.github.io/)
- [Unattended Upgrades](https://wiki.debian.org/UnattendedUpgrades)
- [ModSecurity](https://www.modsecurity.org/)
- [docker-bench-security](https://github.com/docker/docker-bench-security)
- [Lynis](https://cisofy.com/lynis/)
- [sliplane.io](https://sliplane.io/)

---
This role provides robust, production-grade security hardening for your n8n deployment, following best practices for Linux, Docker, and web application security.
