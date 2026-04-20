# RHEL 10 Upgrade Role

This role automates the end-to-end upgrade path from **RHEL 9** to **RHEL 10** using the `leapp` utility. It is designed for modularity and secure credential handling.

The role doesn't just run the upgrade blindly; it uses that [JSON](leapp_report_details.md) file as a gatekeeper. By parsing the report, the role can decide whether to proceed with the actual transformation or stop to prevent a broken system.

## Dependencies
To function correctly, this role requires external Ansible roles for system registration and analysis.

### Ansible Role Dependencies
Before running the playbook, you must install the required roles from GitLab. Create a requirements.yml file in your project root:

```yaml
# requirements.yml
---
roles:
  - name: role-rhsm-registration
    src: https://gitlab.com/ansible-ssa/role-rhsm-registration.git
    scm: git

  - name: role-insights-registration
    src: https://gitlab.com/ansible-ssa/role-insights-registration.git
    scm: git
```
Install them using the following command:
`ansible-galaxy install -r requirements.yml`

* **RHSM Account:** A valid Red Hat Subscription is required.
* **Insights:** Registration is mandatory for the `leapp` pre-upgrade analysis.
* **Automatic Dependencies:** The following packages are managed and installed by the role:
    * `leapp-upgrade` (The core upgrade engine)
    * `python3-dnf-plugin-versionlock` (To manage package constraints)

## Variable & Secret Management

The playbook uses a hierarchy to find credentials. It checks **Ansible Vault** first, then falls back to **Environment Variables**.

### Defining Variables (`vars/main.yml`)
Configure your variables to support both Vault and Environment lookups:

```yaml
rhsm_username: "{{ vault_rhsm_username | default(lookup('env', 'RHSM_USERNAME'), true) }}"
rhsm_password: "{{ vault_rhsm_password | default(lookup('env', 'RHSM_PASSWORD'), true) }}"
activationkey: "{{ vault_activationkey | default(lookup('env', 'ACTIVATION_KEY'), true) }}"
rhsm_poolid: "{{ vault_rhsm_poolid | default(lookup('env', 'RHSM_POOLID'), true) }}"
cloud_provider: "{{ vault_cloud_provider | default(lookup('env', 'CLOUD_PROVIDER'), true) | default(omit, true) }}"
auto_config: "{{ vault_auto_config | default(lookup('env', 'AUTO_CONFIG'), true) | default(omit, true) }}"
```

#### Option 1: Environment Script (env.sh)
You can source a local script to populate your shell environment:

```bash
#!/bin/bash
export RHSM_USERNAME="your_user"
export RHSM_PASSWORD="your_password"
export CLOUD_PROVIDER="aws"
```

Run `source env.sh` before starting the playbook.

#### Option 2: Ansible Vault (vars_secrets.yml)
Encrypt your sensitive data in a vaulted file:

```yaml
vault_rhsm_username: "your_user"
vault_rhsm_password: "your_secret_password"
```

### Usage
Execute the upgrade using `ansible-navigator`. This command ensures the vault is unlocked and variables are properly mapped.

```yaml
ansible-navigator run upgrade.yml \
  -i inventory.ini \
  -m stdout \
  --vars-file vars_secrets.yml \
  --vault-id @prompt
```

### Workflow Phases
Phase 1: Repositories - Sets RHEL 9.6 as the target and prepares repos.

Phase 2: Update - Brings the current OS to the latest minor version and reboots.

Phase 3: Upgrade - Runs leapp preupgrade, fetches the report for review, and triggers the final RHEL 10 transformation.

