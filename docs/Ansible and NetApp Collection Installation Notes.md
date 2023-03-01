## Installing / Updating python3

`sudo apt install python3`

`python3 --version`

## Installing / Updating pip

`sudo apt install python3-pip`
or
`sudo python -m pip install --upgrade pip`

`pip3 --version`

## Installing Ansible:

Installing Ansible from Pypi - Recommended latest source

`sudo pip install ansible`

Alternate from package management system - preference for Ansible 2.10+
`yum`
`apt`
etc.

Installing NetApp Library

`sudo pip install requests`
`sudo pip install netapp-lib`

## Installing NetApp Collection(s)

### Installing latest collection version:

`sudo ansible-galaxy collection install netapp.ontap -f -p /usr/local/share/ansible/collections/`

Process install dependency map
Starting collection install process
Installing 'netapp.ontap:21.2.0' to '/usr/share/ansible/collections/ansible_collections/netapp/ontap'

`sudo chmod +rx -R /usr/share/ansible/collections/`

### Installing / Updating Ansible collection:

sudo ansible-galaxy collection install netapp.ontap -p /usr/share/ansible/collections --force

`sudo chmod +rx -R /usr/share/ansible/collections/`

## Testing NetApp Collection & Module Availability

```
pip list | grep ansible 
pip list | grep netapp-lib
ansible-galaxy collection list | grep netapp
```

### Updating pip & pip packages

`sudo python -m pip install --upgrade pip`
`sudo pip install --upgrade netapp-lib`

## Working with vCenter, vms & ontap/sg vsims

Leveraging the **deploy** playbooks requires additional libraries:

`sudo pip install pyvmomi`
`sudo ansible-galaxy collection install community.vmware -f -p /usr/share/ansible/collections/`