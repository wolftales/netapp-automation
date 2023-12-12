# Ansible Workshop Live Demo Script

### Login to demo ENV

## ENV Setup

===

### Pre-Check

  1. Login: cluster1.demo.netapp.com
  2. SM - no aggr and spare drives
  3. Github - schmots1

### Initialize ENV

1. Login to the Ansible host as ansible
2. Increase font size
3. Update `pip2 & pip3`: 
   1. `python2 -m pip install --upgrade pip`
   2. `python3 -m pip install --upgrade pip`

Note: Use FQ-Path to run python3 & pip3

`sudo /usr/local/bin/pip3 install ansible`  # add ansible==2.10.7

`sudo /usr/local/bin/pip3 install netapp-lib requests`

`ansible --version`

Galaxy.ansible.com
Galaxy.ansible.com/netapp

`sudo /usr/local/bin/ansible-galaxy collection install netapp.ontap -p /usr/share/ansible/collections`

`sudo chmod -R +x /usr/share/ansible/collections`

`git clone https://github.com/wolftales/ansible_workshop.git`

`cd ansible_workshop`

`ansible-playbook setup.yml`

## Catch Everyone up

----

1. <https://github.com/schmots1/ansible_workshop.git>
2. Lab Exercise Guide:
3. Ensure everyone has credentials and this Lab Exercise Guide

- Review playbook output
- less setup.yml

## Updates to setup.yml file

Setup.yml fails with:
“Error when validating options: nodes is required when disk_count is present”
Workaround:
- **use_rest: never**
- Add nodes

“Missing required arguments: rule_index”
Workaround:
- use_rest: never
- Add **rule_index: 1**

## Begin Playbook Demo

`mkdir work`

`cd work`

`ansible-docs netapp.ontap.na_ontap_volume`

`vi volume.yml`

with volume.yml:

- Create - simple vol create, no vars
- Update - Idempotency
- Delete - Introduce absent
- Variables - create 3, add some vars
- Loop - Simple then Compound, add more vars
- ticket & --extra-vars
- vars_files

# Advanced Ansible Techniques

## YAML Alias

```yml
vars:
  login: &login
    hostname: "{{ netapp_hostname }}"
    username: "{{ netapp_username }}"
    password: "{{ netapp_password }}"
    https: true
    validate_certs: false

tasks:
- name: Add this alias line to parameters
    <<: *login
```

## Module Defaults

```yml
module_defaults:
    group/netapp.ontap.netapp_ontap:
      # hostname: "{{ netapp_hostname }}"
      username: "{{ netapp_username }}"
      password: "{{ netapp_password }}"
      https: true
      validate_certs: false
      use_rest: always
```

## Credential Mgmt

1. clear text
2. VARS Prompt - Password
3. Ansible-Vault - encrypt_string
4. Certificate Authentication - SSL Certs

## VARS Prompt - Password

```yml
vars_prompt:
- name: "netapp_password"
  prompt: "Enter the NetApp Admin password"
  private: yes
  unsafe: no
```

## Ansible-Vault - encrypt_string

### Create the encrypted string

`ansible-vault encrypt_string netapp123 --name 'password' >> password.yml`
`New Vault password: demo`
`Confirm New Vault password: demo`

### save decrypt 'key'

`echo demo >> decrypt`

### Place reference to encrypted varible in playbook

```
vars_files:
- password.yml
```

### Running a playbook with encrypted password

**Interactive**  
`ansible-playbook --ask-vault-pass myplaybook.yml`

**Non-interactive - Necessary for scripting**  
`ansible-playbook --vault-id decrypt myplaybook.yml`

## Certificate Authentication - SSL Certs

### Create SSL Cert for admin

`openssl req -x509 -nodes -days 1095 -newkey rsa:2048 -keyout admin.key -out admin.pem -subj “/C=US/ST=NC/L=RTP/O=NetApp/CN=admin”`

**Note: copy the contents of the .pem file:  including the —–BEGIN CERTIFICATE—— and —–END CERTIFICATE—– parts**

### Install client-ca SSL Cert for admin on ONTAP

`security certificate install -type client-ca -cert-name admin -vserver cluster1`

### Enable client-ca SSL Cert for admin

`security ssl modify -vserver cluster1 -client-enabled true`

### Enable SSL Cert auth method for admin

`security login create -user-or-group-name admin -application http -authentication-method cert`
`security login create -user-or-group-name admin -application ontapi -authentication-method cert`

#### Leveraging SSL authentication

* Original section example:

  ```yml
  hostname: Ansible01
  username: admin
  password: netapp123
  https: true
  validate_certs: false
  ```

**Note**: With the .pem and .key files in the same directory as the playbook you could instead use this:*

- Updated section example:

  ```yml
  hostname: Ansible01
  validate_certs: false
  cert_filepath: admin.pem
  key_filepath: admin.key
  ```

#### SSL Cert Highlights

- username not required
- https not required
- local or FQ-Path to filepath(s)

### Role

- create role dir: `vol_mgmt` & `defaults`, `tasks`
- copy `vars_files.yml` to `defaults/main.yml`
- modify `main.yml` for role:
  - remove most varibles  
- copy `volume.yml` & `export_mgmt.yml` to `tasks/main.yml`
- modify `main.yml` for role:
  - remove playbook header info
  - reduce indentation of tasks
- create new `role_mgmt.yml` playbook:
  - contains: `vars_files.yml`
  - remove everything below `tasks:` and add:

    ```
    roles:
     - volume_mgmt
    ```

### MOTD

- `connection: local` @ top
- Work with admin vserver: `cluster1`
- use either `ansible_host` or `inventory_hostname` for `hostname`
- use `shortname` for `vserver`

# -=- AWX -=-

----

## AWX / Tower Inventory

* Leverage `Demo Inventory`
- Show `localhost` definition
- Map back to *ansible-base* inventory discussion: Simularities & Differences

## Create Credentials

1. `credential type` - ONTAP Username
2. `Create user from profile` - ONTAP Admin

### ONTAP Login - Credential Format for NetApp ONTAP Systems

##### Input Configuration

```yml
fields:
  - id: netapp_username
    type: string
    label: Username
  - id: netapp_password
    type: string
    label: Password
    secret: true
```

##### Injector Configuration

```yml
extra_vars:
  netapp_username: '{{ netapp_username }}' 
  netapp_password: '{{ netapp_password }}'
  ```

### ONTAP Login - User created from credential format above

Create `user` leveraging the above credential format (profile)

## Create Project

- Get link for ansible_workshop: <https://github.com/schmots1/ansible_workshop.git>
- highlight "Job History" - now that something has been done (SCM Update)

## Create Job Template

- Create export, rule & volume job templates

## Create Workflow

- chain export, rule & jobs together in a Workflow
- Create user & delegate
