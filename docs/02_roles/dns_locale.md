# DNS & Locale Role: DNS Records and Locale Settings for n8n

## Overview
This role provides DNS record templates/instructions and configures system locale and timezone for your n8n deployment. It verifies DNS resolution and ensures locale packages are present and configured.

## Features
- Generates a Markdown file with A/CNAME, MX, SPF, DKIM, and DMARC record templates for `n8n.majitask.fun`
- Sets system locale (en_US.UTF-8 or cs_CZ.UTF-8) and timezone (Europe/Prague)
- Installs required locale and timezone packages
- Verifies DNS resolution for A/CNAME and MX records, warns if not correct
- Handlers to restart services if locale/timezone changes

## Variables
Set in `group_vars/vars.yml`:
```yaml
dns_locale_lang: "en_US.UTF-8"  # or "cs_CZ.UTF-8"
dns_locale_timezone: "Europe/Prague"
n8n_public_ip: "1.2.3.4"  # Set to your server's public IP
dns_locale_mx_target: "smtp.gmail.com."
```

## Usage
Add `dns_locale` to your playbook roles. The role will:
- Output `/etc/n8n-dns-records.md` with DNS record templates
- Set locale and timezone as configured
- Warn if DNS records do not resolve as expected

## Manual Steps
- Update your DNS provider with the records in `/etc/n8n-dns-records.md`
- Generate and publish DKIM keys as described in the Postfix role documentation

## References
- [Debian Locales](https://wiki.debian.org/Locale)
- [timedatectl](https://man7.org/linux/man-pages/man1/timedatectl.1.html)
- [SPF, DKIM, DMARC Setup](https://easydmarc.com/)

---
This role ensures your n8n deployment is correctly localized and DNS is ready for secure, reliable operation.
