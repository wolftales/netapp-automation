na_ontap_ldap_config
=========

Configure one or more of the following ONTAP settings:

LDAP Client Config
LDAP Config
Manage User, Group & Netgroup DN
Manage name service Dbs


Requirements
------------

Since this uses the NetApp ONTAP modules it will require the python library netapp-lib

Role Variables
--------------

hostname: <ONTAP mgmt ip or fqdn>
username: <ONTAP admin account>
password: <ONTAP admin account password>

Based on if Variables != or == None determins if a section runs.  Each variable will take one or more dictonary entries.  The following would run all sections

# LDAP Configuration
ldap_config:
- { svm: '{{ svm }}', config_name: '{{ ldap_configname }}', ad_domain: '{{ ad_domain }}', schema: '{{ ldap_schema }}', port: '{{ ldap_port }}', query_timeout: '{{ ldap_query_timeout }}', min_bind_level: '{{ ldap_min_bind_level }}', use_start_tls: '{{ ldap_use_start_tls }}', session_security: '{{ldap_session_security }}' , base_dn: '{{ ldap_base_dn }}', base_scope: '{{ ldap_base_scope }}', bind_dn: '{{ ldap_bind_dn }}' }

# LDAP Client-User, Group, Netgroup DN Config
ldap_client_config:
- { command: 'ldap client modify modify -vserver {{ svm }} -client-config {{ ldap_configname }} -ad-domain {{ ldap_ad_domain }} -user-dn {{ ldap_user_dn }} -group-dn {{ ldap_group_dn }} -netgroup-dn {{ ldap_netgroup_dn }}' }

# LDAP NSS Configuration
nss_config:
- { svm: '{{ svm }}', db_type: <db-name>, sources: 'files,ldap' }


Dependencies
------------

None

Example Playbook
----------------

I use a globals file to hold my variables.

globals.yml
---
cluster_name:         vsim
svm:                  vs0

netapp_hostname:      172.30.10.182
netapp_username:      admin
netapp_password:      netapp123

ldap_configname:      'test'
ldap_ad_domain:       ldap.hq.company.com
ldap_servers:         10.10.10.110
ldap_schema:          'RFC-2307'
ldap_port:            389
ldap_query_timeout:   3
ldap_min_bind_level:  'simple'
ldap_use_start_tls:   'false'
ldap_session_security: 'none'

ldap_base_dn:         "dc=lab,ou=bu,o=company"
ldap_base_scope:      'subtree'
ldap_bind_dn:         "dc=lab,ou=bu,o=company"

ldap_user_dn:         "cn=accounts,dc=dept,dc=company,dc=com"
ldap_group_dn:        "cn=groups,cd=accounts,dc=dept,dc=company,dc=com"
ldap_netgroup_dn:    "cn=netgroup,dc=lab,ou=bu,o=company"

# LDAP Configuration
ldap_config:
  - { svm: '{{ svm }}', config_name: '{{ ldap_configname }}', ad_domain: '{{ ad_domain }}', schema: '{{ ldap_schema }}', port: '{{ ldap_port }}', query_timeout: '{{ ldap_query_timeout }}', min_bind_level: '{{ ldap_min_bind_level }}', use_start_tls: '{{ ldap_use_start_tls }}', session_security: '{{ldap_session_security }}' , base_dn: '{{ ldap_base_dn }}', base_scope: '{{ ldap_base_scope }}', bind_dn: '{{ ldap_bind_dn }}' }

# LDAP Client-User, Group, Netgroup DN Config
ldap_client_config:
  - { command: 'ldap client modify modify -vserver {{ svm }} -client-config {{ ldap_configname }} -ad-domain {{ ldap_ad_domain }} -user-dn {{ svm_ldap_user_dn }} -group-dn {{ svm_ldap_group_dn }} -netgroup-dn {{ svm_ldap_netgroup_dn }}' }

# LDAP NSS Configuration
nss_config:
  - { svm: '{{ svm }}', db_type: namedb, sources: 'files,ldap' }
  - { svm: '{{ svm }}', db_type: hosts, sources: 'files,ldap' }
  - { svm: '{{ svm }}', db_type: group, sources: 'files,ldap' }
  - { svm: '{{ svm }}', db_type: passwd, sources: 'files,ldap' }
  - { svm: '{{ svm }}', db_type: netgroup, sources: 'files,ldap' }

cluster_config.yml
---
- hosts: localhost
  vars_files:
    - globals.yml
  vars:
    input: &input
      hostname: "{{ netapp_hostname }}"
      username: "{{ netapp_username }}"
      password: "{{ netapp_password }}"
      https: true
      validate_certs: false
  tasks:
  - import_role:
      name: na_ontap_ldap_config
    vars:
      <<: *input

License
-------

GNU v3

Author Information
------------------
NetApp
http://www.netapp.io
