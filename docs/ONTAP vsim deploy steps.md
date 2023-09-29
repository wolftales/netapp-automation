# ONTAP vsim Deploy Process

This is a summary of the steps required to take a new ONTAP vsim deployed on VMware with the [madlabber - deploy_ovf_vsim](https://github.com/madlabber/deploy_ovf_vsim)

The current version of this role can create a 2 node HA-pair. There are some manual steps adding the second node to the cluster, and some broadcast-domain clean-up that should be done prior to proceeding with additional automated configuration. 

## Preparation

Steps running the automation:

1. Download the ONTAP Simulator version you want to setup within ESXi or vCenter.
2. Place *ovf* in `files/ontap` directory
3. Adjust variable inputs for `ansible-playbook`
4. Run *Ansible* playbook

## **vsim** updates to get nodes joined together

On node #2

```shell
cluster setup  # join
broadcast-domain add-ports -broadcast-domain Default -ports sandbox-02:e0d,sandbox-02:e0d,sandbox-02:e0e,sandbox-02:e0f
network interface modify -vserver sandbox -lif sandbox-02_mgmt1 -home-port e0d
broadcast-domain add-ports -broadcast-domain Default -ports sandbox-02:e0c
network interface modify -vserver sandbox -lif sandbox-02_mgmt1 -home-port e0c
network interface migrate -vserver sandbox -lif sandbox-02_mgmt1 -destination-node sandbox-02 -destination-port e0c

```