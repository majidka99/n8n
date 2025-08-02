# Using vars.yml and vault.yml in This Ansible Project

This project separates non-secret variables (`vars.yml`) from secrets (`vault.yml`) for security and maintainability. Secrets are encrypted using Ansible Vault.

---

## 1. Vault Password File

Create a file named `.vault_pass.txt` in the project root:

```bash
echo "your-strong-vault-password" > /n8n/ansible-n8n-deployment/.vault_pass.txt
chmod 600 /n8n/ansible-n8n-deployment/.vault_pass.txt
```

- Replace `your-strong-vault-password` with a secure password.
- **Never commit this file to git** (it is already in `.gitignore`).

---

## 2. Encrypting and Editing Secrets

To edit or encrypt `vault.yml`:

```bash
ansible-vault edit --vault-password-file .vault_pass.txt group_vars/vault.yml
```

Or to encrypt a new file:

```bash
ansible-vault encrypt --vault-password-file .vault_pass.txt group_vars/vault.yml
```

---

## 3. How vars and vault are used

- `group_vars/vars.yml`: All non-secret variables (hostnames, ports, config, etc).
- `group_vars/vault.yml`: All secrets (passwords, API keys, tokens, etc). **This file should always be encrypted.**

Both files are loaded automatically by Ansible when running playbooks.

---

## 4. Running Playbooks with Vault

Run your playbook with the vault password file:

```bash
ansible-playbook -i inventory/hosts playbooks/main.yml --vault-password-file .vault_pass.txt
```

---

## 5. Best Practices

- Never commit `.vault_pass.txt` or unencrypted secrets to git.
- Share the vault password securely with trusted team members only.
- Use `ansible-vault rekey` to change the password if needed.
- Keep `vars.yml` for non-secrets and `vault.yml` for secrets only.

---

For more details, see the [Ansible Vault documentation](https://docs.ansible.com/ansible/latest/user_guide/vault.html).
