# Testing, Linting, and Dry-Run for This Ansible Project

This project uses Molecule for role testing, ansible-lint for code quality, and Ansible's check mode for dry-run validation.

---

## 1. Prerequisites

### Install Dependencies
```bash
# Install Docker (required for Molecule testing)
sudo apt update && sudo apt install -y docker.io

# Add your user to docker group (to run without sudo)
sudo usermod -aG docker $USER

# Create virtual environment for testing tools
python3 -m venv molecule-venv
source molecule-venv/bin/activate

# Install testing dependencies
pip install molecule[docker] ansible-lint
pip install molecule-plugins[docker]
```

### Restart Terminal
After adding yourself to the docker group, restart your terminal or run:
```bash
newgrp docker
```

---

## 2. ansible-lint: Code Quality & Best Practices

Run ansible-lint from the project root to check for best practices and errors:

```bash
# Basic linting
ansible-lint

# Lint specific files
ansible-lint playbooks/main.yml
ansible-lint roles/*/tasks/main.yml

# Show all rules
ansible-lint --list-rules

# Generate report
ansible-lint --format=sarif > ansible-lint-results.sarif
```

### Common Issues to Fix:
- **YAML formatting**: Proper indentation and syntax
- **Ansible best practices**: Use of handlers, tags, documentation
- **Security**: Avoid shell commands, use proper modules
- **Idempotency**: Tasks should be safe to run multiple times

---

## 3. Molecule: Role Testing

Molecule tests your roles in isolated Docker containers.

### Basic Molecule Commands:
```bash
# Activate virtual environment first
source molecule-venv/bin/activate

# Run full test suite
cd molecule/default
molecule test

# Individual test steps
molecule create      # Create test instances
molecule converge    # Apply the playbook
molecule verify      # Run verification tests
molecule destroy     # Clean up instances

# List available scenarios
molecule list
```

### Testing Specific Components:
```bash
# Test monitoring setup
molecule test --scenario-name monitoring

# Test with different variables
MOLECULE_DEBUG=1 molecule test
```

### Molecule Configuration:
Your `molecule/default/molecule.yml` defines:
- **Test platforms**: Ubuntu containers
- **Test scenarios**: Different variable combinations
- **Verification**: Custom tests for services

---

## 4. Ansible Dry-Run (Check Mode)

Perform dry-runs to see what would change without making actual changes:

```bash
# Full dry-run with encrypted vault
ansible-playbook -i inventory/hosts playbooks/main.yml \
  --check --diff --vault-password-file .vault_pass.txt

# Test specific roles or tags
ansible-playbook -i inventory/hosts playbooks/main.yml \
  --check --diff --tags "monitoring,reverse_proxy" \
  --vault-password-file .vault_pass.txt

# Dry-run without vault (for non-secret changes)
ansible-playbook -i inventory/hosts playbooks/main.yml \
  --check --diff --skip-tags "vault"
```

### Check Mode Options:
- `--check`: Enables dry-run mode (no changes made)
- `--diff`: Shows differences for files/templates
- `--limit`: Restrict to specific hosts
- `--start-at-task`: Start from specific task

---

## 5. Integration Testing

### Test Monitoring Stack:
```bash
# After deployment, verify monitoring endpoints
curl -k https://prometheus.yourdomain.com/-/healthy
curl -k https://alerts.yourdomain.com/-/healthy

# Check Docker containers
docker ps | grep -E "(prometheus|alertmanager|node-exporter|cadvisor)"

# Verify Nginx configurations
nginx -t
systemctl status nginx
```

### Test SSL Certificates:
```bash
# Check certificate status
certbot certificates

# Test SSL configuration
openssl s_client -connect yourdomain.com:443 -servername yourdomain.com
```

### Test Firewall Rules:
```bash
# Verify UFW status
sudo ufw status verbose

# Test blocked ports (should fail from external)
nmap -p 9090,9093,8080,9100 your-server-ip
```

---

## 6. Continuous Integration

### GitHub Actions Workflow:
```yaml
name: Ansible CI
on: [push, pull_request]
jobs:
  lint-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      - name: Install dependencies
        run: |
          pip install ansible ansible-lint molecule[docker]
      - name: Run ansible-lint
        run: ansible-lint
      - name: Run molecule tests
        run: |
          cd molecule/default
          molecule test
```

---

## 7. Testing Workflow

### Recommended Testing Sequence:
```bash
# 1. Lint first
ansible-lint

# 2. Run molecule tests
source molecule-venv/bin/activate
cd molecule/default && molecule test

# 3. Dry-run on target
ansible-playbook -i inventory/hosts playbooks/main.yml \
  --check --diff --vault-password-file .vault_pass.txt

# 4. Deploy to staging/test environment
ansible-playbook -i inventory/staging playbooks/main.yml \
  --vault-password-file .vault_pass.txt

# 5. Verify deployment
./scripts/verify-deployment.sh

# 6. Deploy to production
ansible-playbook -i inventory/hosts playbooks/main.yml \
  --vault-password-file .vault_pass.txt
```

---

## 8. Troubleshooting Tests

### Common Issues:

**Docker Permission Denied:**
```bash
sudo usermod -aG docker $USER
newgrp docker
```

**Molecule Not Found:**
```bash
source molecule-venv/bin/activate
which molecule
```

**Lint Failures:**
```bash
# Fix common issues
ansible-lint --fix
```

**SSH Connection Issues:**
```bash
# Test connectivity
ansible all -i inventory/hosts -m ping --vault-password-file .vault_pass.txt
```

---

## 9. Advanced Testing

### Custom Verification Scripts:
Create `molecule/default/tests/test_default.py`:
```python
def test_nginx_running(host):
    nginx = host.service("nginx")
    assert nginx.is_running
    assert nginx.is_enabled

def test_monitoring_containers(host):
    assert host.docker("prometheus").is_running
    assert host.docker("alertmanager").is_running
```

### Performance Testing:
```bash
# Test n8n response time
curl -w "@curl-format.txt" -o /dev/null -s https://n8n.yourdomain.com

# Load testing
ab -n 100 -c 10 https://n8n.yourdomain.com/
```

---

Always run the complete testing suite before deploying to production! 🚀
