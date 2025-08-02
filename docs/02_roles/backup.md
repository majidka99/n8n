# Role: backup

## Purpose
Implement robust backup and disaster recovery for the n8n stack, including data, configuration, and off-site options.

## Features
- Schedules a weekly backup job (cron) that archives:
  - n8n data volume (/home/node/.n8n)
  - PostgreSQL data volume
  - Application configs (Ansible inventory, templates)
  - Proxy configs (Caddy/Nginx, SSL certs)
- Uses tar -czf to create compressed archives named with the current date (YYYYMMDD), stored in /var/backups/n8n/
- Rotates backups, keeping only the last two (configurable)
- Nightly logical PostgreSQL backups via pg_dump, stored in a separate directory and included in weekly rotation
- Optionally uploads archives to off-site storage (rclone/restic, credentials from Vault)
- All cron jobs are idempotent and log to /var/log/n8n_backup.log
- Includes restore procedure documentation

## Variables (from group_vars/vars.yml)
- `backup_dir`: Main backup directory
- `backup_pg_dump_dir`: Directory for nightly pg_dump backups
- `backup_script_path`, `backup_pg_dump_script_path`, `backup_upload_script_path`: Script locations
- `backup_log_path`: Log file for backup jobs
- `backup_keep_count`: Number of backups to retain (default: 2)
- `backup_offsite_enabled`: Enable off-site upload (default: false)

## Secrets (from group_vars/vault.yml)
- rclone/restic credentials (if off-site enabled)

## Templates
- `backup.sh.j2`: Main backup script
- `pg_dump_backup.sh.j2`: Nightly pg_dump script
- `upload_offsite.sh.j2`: Off-site upload script (optional)

## Idempotence
All tasks and cron jobs are idempotent and safe to re-run.

## Usage
Include the role in your playbook:
```yaml
- hosts: all
  roles:
    - role: backup
```

## Restore Procedure
- Extract the desired backup archive from /var/backups/n8n/
- Restore Docker volumes using docker run or docker cp
- Restore configs and pg_dump files as needed
- Test restores in a staging environment before production recovery
