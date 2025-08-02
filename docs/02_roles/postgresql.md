# Role: postgresql

## Purpose
Deploy and secure a PostgreSQL database for n8n using Docker Compose, with best practices for authentication, access, and backup.
- Runs the official postgres image with a persistent named volume
- Reads all secrets (user, password, db) from Ansible Vault
- Hardened authentication (md5 or scram-sha-256), secure listen_addresses
- Supports SSL/TLS for remote scenarios (with documented config)
- Creates least-privilege n8n user and database
- Nightly logical backups via pg_dump, included in weekly backup rotation

## Variables (from group_vars/vars.yml)
- `postgresql_image_tag`: Postgres image version (default: 15)
- `postgresql_backup_dir`: Directory for logical backups
- `postgresql_listen_addresses`: Listen addresses (default: 127.0.0.1)
- `postgresql_ssl`: Enable SSL (default: off)

## Secrets (from group_vars/vault.yml)
- `n8n_database_user`: Database user
- `n8n_database_password`: Database password
- `n8n_database_name`: Database name

## Templates
- `docker-compose.postgres.yml.j2`: Compose file for local/remote/managed DB
- `postgresql.conf.j2`: Secure config (listen_addresses, SSL)
- `pg_hba.conf.j2`: Authentication and access control

## Security Best Practices
- Uses md5 or scram-sha-256 auth (never trust)
- Restricts listen_addresses to localhost or private subnet
- Optionally enforces SSL for remote access (with certs)
- Only n8n user has DB access/ownership
- Backups are automated and rotated

## Idempotence
All tasks are idempotent and safe to re-run.

## Usage
Include the role in your playbook:
```yaml
- hosts: all
  roles:
    - role: postgresql
```
