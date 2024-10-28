# NetApp ONTAP Automation with Ansible

This repository provides an Ansible-based automation environment for managing NetApp ONTAP clusters. The inventory is organized to streamline variable management, making it easier to configure and automate tasks across multiple ONTAP clusters.

## Table of Contents
1. [Overview](#overview)
2. [Inventory Structure](#inventory-structure)
3. [Variable Management](#variable-management)
4. [Playbook Structure](#playbook-structure)
5. [Getting Started](#getting-started)
6. [Running Playbooks](#running-playbooks)
7. [Best Practices](#best-practices)
8. [Contributing](#contributing)

---

## Overview

This Ansible setup enables automated configuration, deployment, and management of ONTAP clusters. It includes a structured inventory for organizing hosts and variables, making it adaptable to multiple clusters and reusable across environments.

### Key Features
- Organized inventory layout for scalable ONTAP management
- Centralized variable management for different environments and clusters
- Secure handling of sensitive data using Ansible Vault
- Playbooks tailored for NetApp ONTAP automation

---

## Inventory Structure

The inventory is structured to automatically load variables from `group_vars` and `host_vars` directories. Here’s a breakdown of the `inventory/` directory:

```txt
inventory/
├── group_vars/
│   ├── all/
│   │   ├── all.yml
│   │   └── vault.yml
│   ├── ontap.yml
├── host_vars/
│   ├── ontap-cl01.yml
│   ├── redshirt.yml
│   └── sandbox.yml
└── hosts.yml
```

### `inventory/hosts.yml`

The main inventory file, `hosts.yml`, organizes ONTAP clusters and environments into groups. An example structure:

```yaml
# inventory/hosts.yml
all:
  children:
    ontap:
      children:
        ontap-cl01:
          hosts:
            ontap-cl01:
              ansible_host: 192.168.7.180
        sandbox:
          hosts:
            sandbox:
              ansible_host: 192.168.7.200
        redshirt:
          hosts:
            redshirt:
              ansible_host: 192.168.8.240
```

- **Groups**: `ontap` is the parent group, containing child groups for each cluster (`sandbox`, `ontap-cl01`, `redshirt`).
- **Hosts**: Each host is defined under its respective group with its `ansible_host` for connectivity.

---

## Variable Management

Variables are organized into `group_vars` and `host_vars` to allow for inheritance and environment-specific settings.

- **Global Variables (`all.yml`)**: Variables that apply to all hosts (e.g., default connection settings, common login credentials).
- **ONTAP Variables (`ontap.yml`)**: Settings shared by all ONTAP clusters, like licensing, time servers, and DNS configurations.
- **Defaults (`defaults.yml`)**: Optional file for fallback/default values, used across clusters if not overridden.
- **Sensitive Data (`vault.yml`)**: Encrypted file containing sensitive information (passwords, tokens) that is accessible across the environment.

### Example Variable Files

#### `inventory/group_vars/all/defaults.yml`

```yaml
# inventory/group_vars/all/defaults.yml
netapp_username: admin
validate_certs: false  # Set to bypass security error due to self-signed certs
https: true
```

#### `inventory/group_vars/all/vault.yml`

This file should be encrypted using Ansible Vault to securely store sensitive data.

```yaml
# inventory/group_vars/all/vault.yml
netapp_password: !vault |
  $ANSIBLE_VAULT;1.1;AES256
  39313731346566363564326563643033623364353030383334646135623862616636663465333936
  ...
```

---

## Playbook Structure

Playbooks are designed for modular automation of ONTAP tasks, such as configuration, deployment, and routine maintenance.

### Example Playbook

```yaml
---
- name: NetApp ONTAP Authentication Test
  hosts: ontap
  gather_facts: false
  collections:
    - netapp.ontap

  tasks:
    - name: "Configuring ONTAP Cluster: {{ inventory_hostname }}"
      ansible.builtin.debug:
        msg:
          - "NetApp ONTAP Target {{ inventory_hostname }}"
          - "inventory_hostname is {{ inventory_hostname }}"
          - "netapp_hostname is {{ netapp_hostname | default('<|HOSTNAME UNDEFINED|>') }}"
          - "netapp_username is {{ netapp_username | default('<|USERNAME UNDEFINED|>') }}"
          - "netapp_password is {{ '<|PASSWORD SET|>' if netapp_password is defined else '<|PASSWORD UNDEFINED|>' }}"
```

### Ansible Configuration
Ensure your `ansible.cfg` is correctly configured to use the vault password file:

```ini
[defaults]
inventory = inventory/hosts.yml
vault_password_file = vars/.vault-pass
```

---

## Getting Started

### Prerequisites

- **Ansible**: Ensure Ansible is installed (`ansible --version` to check).
- **NetApp Ansible Collection**: Install the NetApp ONTAP collection:

  ```bash
  ansible-galaxy collection install netapp.ontap
  ```

- **Ansible Vault**: To manage encrypted data, initialize Ansible Vault by setting up `vault.yml`:

  ```bash
  ansible-vault encrypt inventory/group_vars/vault.yml
  ```

### Setting Up Credentials

Store sensitive credentials like `netapp_password` in `vault.yml`. Use `--ask-vault-pass` when running playbooks to decrypt.

---

## Running Playbooks

Run playbooks with the `--ask-vault-pass` flag if using Ansible Vault. Target specific groups or hosts as needed:

```bash
ansible-playbook -i inventory/hosts.yml playbook.yml --ask-vault-pass
```

#### Example Commands

- Target `sandbox` environment:

  ```bash
  ansible-playbook -i inventory/hosts.yml playbook.yml -l sandbox --ask-vault-pass
  ```

- Run in check mode (dry run):

  ```bash
  ansible-playbook -i inventory/hosts.yml playbook.yml --check --ask-vault-pass
  ```

---

## Best Practices

- **Use `group_vars` and `host_vars`**: Organize variables to minimize redundancy.
- **Encrypt Sensitive Data**: Always use `vault.yml` for sensitive data, accessible with `--ask-vault-pass`.
- **Dry Run**: Test playbooks with `--check` to simulate changes before applying them.
- **Modular Playbooks**: Keep tasks modular, with separate playbooks for setup, configuration, and maintenance.

---

## Contributing

Feel free to submit pull requests to improve ONTAP automation workflows, add new tasks, or enhance variable management.

---

This setup should provide an efficient, secure, and flexible way to manage NetApp ONTAP clusters using Ansible. For further documentation, see [Ansible Documentation](https://docs.ansible.com/) and [NetApp Ansible Collection Documentation](https://galaxy.ansible.com/netapp/ontap).
