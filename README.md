# netapp-automation
Automation leveraged for demo, prototyping, and lab use-cases

# Ansible & Ansible AWX
Ansible 2.9+  
Ansible AWX 13.0  
Collections:  
* netapp.ontap
* netapp.storagegrid

## Requirements for VSIM (OVA's) deploy role:
* community.vmware: `ansible-galaxy collection install community.vmware`
* pyvmoni - Offical version: `pip install --upgrade pyvmomi`

### ONTAP
Standardized Day 1 ONTAP configuration with limited external dependancies, i.e. external integration and orchestration requirements.

This illustrates the use of NetApp vanilla Ansible modules & roles to accomplish standard configuration items.

To install  the  role:
```
ansible-galaxy role install -f -p ./roles git+https://github.com/madlabber/deploy_ovf_vsim.git,main
```

Known Limitations:
* broadcast-domains may not come out correct
    * verify broadcast-domains
    * second node is normally  the where this happens,  but on the first's  extra network adapters it can happen too
    * second node sometimes doesn't join

### StorageGRID
StorageGRID 11.6 OVA's
* Admin node (x2)
* Load-balancer (gateway) node (x1)
* Storage nodes (x4)

To install  the  role:
```
ansible-galaxy role install -f -p ./roles git+https://github.com/madlabber/deploy_ovf_storagegrid.git,main
```


### ActiveIQ Unified Manager
AIQUM 9.9 OVA (Currently running 9.10x2 EVP)

To install  the  role:
```
ansible-galaxy role install -f -p ./roles git+https://github.com/madlabber/deploy_ovf_aiqum.git,main
```

