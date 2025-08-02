# n8n Ansible Deployment: Pre-Flight Checklist & Setup Guide

Before running this repository, review and update the following items to ensure a successful and secure deployment.

## Pre-Flight Checklist

1. **Domain & DNS Setup**
   - Change all example domains in `group_vars/vars.yml` to your real domains:
     - `n8n_domain`: Your main n8n domain (e.g., `n8n.yourdomain.com`)
     - `monitoring_prometheus_domain`: Prometheus subdomain (e.g., `prometheus.yourdomain.com`)
     - `monitoring_alertmanager_domain`: Alertmanager subdomain (e.g., `alerts.yourdomain.com`)
   - Set your server's public IP in `group_vars/vars.yml` (`n8n_public_ip`).
   - Create DNS A records for all subdomains pointing to your server IP.
   - Generate DKIM keys and update `dkim_public_key` in `group_vars/vars.yml`.

2. **Email Configuration**
   - Change all example emails in `group_vars/vars.yml` to your real admin/notification email:
     - `n8n_admin_email`: Main admin email
     - `dns_admin_email`: DNS/DMARC notifications
     - `dmarc_rua_email`: DMARC aggregate reports
     - `monitoring_alert_email_from/to`: Monitoring alerts
   - Configure Gmail App Password for SMTP relay in `group_vars/vault.yml`.

3. **Secrets & Vault Management**
   - **CRITICAL**: Update ALL passwords in `group_vars/vault.yml`:
     - `n8n_database_password`: PostgreSQL password
     - `user_access_n8n_password`: System user password
     - `n8n_basic_auth_password`: n8n web interface password
     - `postfix_relay_password`: Gmail App Password
     - `monitoring_basic_auth_password`: Monitoring interface password
   - Set up vault password: `echo "your-strong-password" > .vault_pass.txt && chmod 600 .vault_pass.txt`
   - Encrypt vault: `ansible-vault encrypt --vault-password-file .vault_pass.txt group_vars/vault.yml`

4. **Monitoring Subdomains Setup**
   - Configure monitoring domains in `group_vars/vars.yml`:
     - Set `monitoring_enable_nginx_proxy: true`
     - Set `monitoring_enable_basic_auth: true`
   - DNS records must be created for monitoring subdomains before deployment.
   - Monitoring services will be accessible via HTTPS with SSL certificates.

5. **Reverse Proxy Configuration**
   - This deployment uses **Nginx only** (Caddy has been removed).
   - SSL certificates are automatically managed via Certbot.
   - Ensure `reverse_proxy_type: "nginx"` in `group_vars/vars.yml`.

6. **System & Security Settings**
   - Set your preferred locale and timezone in `group_vars/vars.yml`:
     - `dns_locale_lang`: Language setting (e.g., `"en_US.UTF-8"`)
     - `dns_locale_timezone`: Timezone (e.g., `"Europe/Prague"`)
   - Review SSH settings: `system_update_ssh_port` (default: 22)
   - Configure firewall rules (UFW will block direct access to monitoring ports).

7. **n8n Application Settings**
   - Configure n8n settings in `group_vars/vars.yml`:
     - `n8n_image_tag`: n8n version to deploy
     - `n8n_log_level`: Logging verbosity
     - `n8n_webhook_url`: Webhook endpoint URL
   - Database settings are configured for PostgreSQL by default.

8. **Backup Configuration**
   - Set backup directories and retention in `group_vars/vars.yml`:
     - `backup_dir`: Main backup directory
     - `backup_keep_count`: Number of backups to retain
     - `backup_notify_email`: Email for backup notifications
   - Enable offsite backup if needed: `backup_offsite_enabled: true`

## How to Update

### Configuration Files:
- **Non-secrets**: Edit `group_vars/vars.yml` for all non-sensitive variables
- **Secrets**: Edit `group_vars/vault.yml` for passwords and sensitive data
- **DNS**: Use template in `roles/dns_locale/templates/dns_records.md.j2` for your DNS provider

### Advanced Customization:
- **Nginx configs**: Modify templates in `roles/reverse_proxy/templates/`
- **Monitoring**: Customize Prometheus/Alertmanager configs in `roles/monitoring/templates/`
- **Security**: Adjust hardening settings in `roles/hardening/`

### Documentation:
- Review role-specific docs in `docs/02_roles/` for detailed configuration options
- Check `docs/01_getting_started/USING_VARS_AND_VAULT.md` for vault management

## Deployment Workflow

### 1. **Pre-deployment Testing:**
```bash
# Install testing dependencies
python3 -m venv molecule-venv
source molecule-venv/bin/activate
pip install molecule[docker] ansible-lint

# Run quality checks
ansible-lint

# Run Molecule tests
cd molecule/default && molecule test

# Dry-run deployment
ansible-playbook -i inventory/hosts playbooks/main.yml \
  --check --diff --vault-password-file .vault_pass.txt
```

### 2. **Production Deployment:**
```bash
ansible-playbook -i inventory/hosts playbooks/main.yml --vault-password-file .vault_pass.txt
```

### 3. **Post-deployment Verification:**
- **n8n**: Visit `https://n8n.yourdomain.com`
- **Monitoring**: Visit `https://prometheus.yourdomain.com` and `https://alerts.yourdomain.com`
- **SSL**: Verify certificates are properly installed
- **Services**: Check all Docker containers are running
- **Firewall**: Confirm monitoring ports are blocked externally

## Security Checklist

- ✅ All passwords are strong and unique
- ✅ Vault file is encrypted and password is secure
- ✅ DNS records are properly configured
- ✅ SSL certificates are automatically managed
- ✅ Monitoring interfaces are password-protected
- ✅ Direct port access is blocked by firewall
- ✅ SSH keys are properly configured
- ✅ Backup notifications are configured

---

**⚠️ IMPORTANT SECURITY NOTES:**
- Never commit `.vault_pass.txt` or unencrypted `vault.yml` to version control
- Use strong, unique passwords for all services
- Regularly update the vault password and rotate secrets
- Monitor failed login attempts via Fail2ban logs
- Keep your n8n and system packages updated

## How to Update
- Edit `group_vars/vars.yml` and `group_vars/vault.yml` for all variables and secrets.
- Edit DNS records at your DNS provider using the template in `roles/dns_locale/templates/dns_records.md.j2`.
- Edit Nginx/Caddy templates in `roles/reverse_proxy/templates/` if you need custom proxy settings.
- Review all documentation in `docs/` and `docs/02_roles/` for role-specific requirements.

## Final Steps
- Run `ansible-lint` and `molecule test` to validate your changes.
- Run the playbook with your inventory and vault password:
  ```bash
  ansible-playbook -i inventory/hosts playbooks/main.yml --vault-password-file .vault_pass.txt
  ```
- Monitor the output for any warnings about missing or default values.
- Verify all services are accessible via their HTTPS domains.

---
**🛡️ Security First:** This deployment prioritizes security with encrypted secrets, firewall protection, SSL everywhere, and monitoring subdomain isolation. Always follow the security checklist above!
