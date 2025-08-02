# Monitoring Role: Enterprise-Grade Observability for n8n

## Overview
This role deploys a comprehensive monitoring stack with secure web access, including Prometheus, Alertmanager, Node Exporter, and cAdvisor. All monitoring services are accessible via HTTPS subdomains with basic authentication protection.

## Features
- **🔒 Secure Access**: HTTPS subdomains with SSL certificates
- **🔐 Authentication**: Basic auth protection for all interfaces
- **🔥 Firewall Protection**: Direct port access blocked
- **📊 Comprehensive Metrics**: System, container, and application monitoring
- **🚨 Smart Alerting**: Email notifications for issues
- **📧 n8n Integration**: Enhanced logging and monitoring

## Architecture

```
Internet → Nginx (HTTPS) → Monitoring Services (localhost only)
```

- **prometheus.example.com** → Prometheus (localhost:9090)
- **alerts.example.com** → Alertmanager (localhost:9093)
- Node Exporter (localhost:9100) - metrics only
- cAdvisor (localhost:8080) - metrics only

## Configuration

### Variables in `group_vars/vars.yml`:
```yaml
# Monitoring subdomain configuration
monitoring_prometheus_domain: "prometheus.example.com"
monitoring_alertmanager_domain: "alerts.example.com"
monitoring_enable_nginx_proxy: true
monitoring_enable_basic_auth: true

# Component toggles
monitoring_enable_prometheus: true
monitoring_enable_alertmanager: true
monitoring_enable_node_exporter: true
monitoring_enable_cadvisor: true
n8n_port: 5678
```
Secrets in `group_vars/vault.yml` (reuses Postfix relay):
```yaml
postfix_relay_username: "vaulted-smtp-user@n8n.example.com"
postfix_relay_password: "vaulted-smtp-password"
```

## Usage
Add `monitoring` to your playbook roles. The role will:
- Deploy node exporter, cAdvisor, Prometheus, and Alertmanager (as enabled)
- Configure Prometheus to scrape metrics and Alertmanager to send email alerts
- Add alert for n8n /healthz endpoint
- Optionally install Monit and configure for n8n/backup monitoring
- Optionally install and configure Filebeat/Vector for log shipping

## Log Aggregation
- Enable log shipping by setting `monitoring_enable_log_shipper: true` and choosing `filebeat` or `vector`.
- Set `monitoring_log_aggregation_host` and `monitoring_log_aggregation_port` for your log aggregation stack (e.g., Loki).

## Email Alerts
- Alerts are sent via the SMTP relay configured in the `postfix` role.
- Set `monitoring_alert_email_from` and `monitoring_alert_email_to` as needed.

## n8n Logging Integration
This role configures n8n logging via environment variables for robust debugging and log streaming:
- `N8N_LOG_LEVEL`: Log output level (`error`, `warn`, `info`, `debug`).
- `N8N_LOG_OUTPUT`: Where to output logs (`console`, `file`, or both).
- `N8N_LOG_FILE_LOCATION`: Log file location (default: `/var/log/n8n/n8n.log`).
- `N8N_LOG_FILE_MAXSIZE`: Maximum log file size in MB (default: 50).
- `N8N_LOG_FILE_MAXCOUNT`: Maximum number of log files to keep (default: 60).

Set these in `group_vars/vars.yml` as needed. The role ensures the log directory exists and injects these into `/etc/environment` for n8n containers or services to consume.

For more details, see the [n8n logging documentation](https://docs.n8n.io/hosting/logging/).

## Extending
- Integrate with external Prometheus/Alertmanager by disabling the built-in containers and pointing scrape configs to your endpoints.
- Add more alert rules in `/etc/prometheus/alert.rules.d/` as needed.

## References
- [Prometheus Node Exporter](https://github.com/prometheus/node_exporter)
- [cAdvisor](https://github.com/google/cadvisor)
- [Prometheus](https://prometheus.io/)
- [Alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager/)
- [Monit](https://mmonit.com/monit/)
- [Filebeat](https://www.elastic.co/beats/filebeat)
- [Vector](https://vector.dev/)
- [Loki](https://grafana.com/oss/loki/)

---
This role provides robust, modular monitoring and alerting for your n8n deployment, following best practices for observability and operational readiness.
