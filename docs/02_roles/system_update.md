# Role: system_update

## Purpose
Harden and maintain a minimal, secure Ubuntu/Debian host for n8n by:
- Removing unnecessary packages and disabling unneeded services
- Enabling automatic security updates
- Setting timezone and ensuring time synchronization
- Hardening SSH access
- Installing and configuring fail2ban for SSH brute-force protection
- (Optionally) Invoking VM snapshot or backup hooks

## Variables (from group_vars/vars.yml)
- `system_update_purge_packages`: List of packages to purge (default: empty)
- `system_update_disable_services`: List of services to disable (default: empty)
- `system_update_ssh_port`: SSH port (default: 22)
- `system_update_snapshot_enabled`: Enable snapshot/backup task (default: false)

## Secrets (from group_vars/vault.yml)
- None by default

## Handlers
- Restart SSH, fail2ban, unattended-upgrades

## Idempotence
All tasks are idempotent and safe to re-run.

## Usage
Include the role in your playbook:
```yaml
- hosts: all
  roles:
    - role: system_update
```
