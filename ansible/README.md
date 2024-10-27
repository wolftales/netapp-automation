# Ansible NetApp ONTAP Configuration

This repository contains Ansible playbooks and inventory files for configuring NetApp ONTAP clusters. The inventory is structured to automatically load variables from `group_vars` and `host_vars` directories.

## Inventory Structure

The inventory is organized as follows:

inventory/
├── group_vars/
│   ├── all/
│   │   ├── all.yml
│   │   └── vault.yml
│   ├── ontap.yml
├── host_vars/
│   ├── ontap-cl01.yml
|   ├── redshirt.yml
│   └── sandbox.yml
└── hosts.yml


### `inventory/hosts.yml`

This file defines the inventory of hosts and groups.

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


```yaml
# inventory/group_vars/all/defaults.yml
netapp_username:  admin
validate_certs:   false           # Set to bypass security error due to self-signed certs
https:            true
```

```yaml
# inventory/group_vars/all/vault.yml
netapp_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          39313731346566363564326563643033623364353030383334646135623862616636663465333936
          3133306263663439656236626366653963653035333061610a373734383235343631633134353534
          62396132313638643765323264373430623166373163303566353630633265663834393537346261
          3633666632373764350a646537326530633438643939306331323965306132656166383331343230
          6665
```

```yaml
---
- name: NetApp ONTAP Authentication Test
  hosts: ontap
  gather_facts: false
  collections:
    - netapp.ontap

  tasks:
    - name: Print netapp_hostname via cluster.net.mgmt lookup
      debug:
        msg:
          - "inventory_hostname is {{ inventory_hostname }}"
          - "netapp_hostname is {{ cluster.net.mgmt | default('<|HOSTNAME UNDEFINED|>') }}"
          - "netapp_username is {{ netapp_username | default('<|USERNAME UNDEFINED|>') }}"
          - "netapp_password is {{ netapp_password | default('<|PASSWORD UNDEFINED|>') }}"
          - "validate_certs is {{ validate_certs | default('<|VALIDATE_CERTS UNDEFINED|>') }}"
          - "https is {{ https | default('<|HTTPS UNDEFINED|>') }}"
```
### Ansible Configuration
Ensure your ansible.cfg is correctly configured to use the vault password file:

```ini
[defaults]
inventory = inventory/hosts.yml
vault_password_file = vars/.vault-pass
```



This [README.md](README.md) file provides detailed information about the inventory structure, variable files, playbook configuration, and troubleshooting steps. It should help document the setup and provide guidance for future reference.