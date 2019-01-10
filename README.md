# DSE Ansible Automation


* Requires Ansible 1.2 or newer
* Expects CentOS/RHEL 6 or 7 hosts

These playbooks deploy a very basic implementation of Datastax Enterprise, version 6.0.4. To use them, first edit the 
"hosts" inventory file to contain the hostnames of the machines on which you want DSE deployed, and edit the 
group_vars/all file to set any DSE configuration parameters you need.  Within the configure_dse/templates folder, you will find
all of configuration files that can be updated for specific configurations. This example specifically creates two data centers, 
whose configuration parameters are set in group_vars/datacenter-1 and group_vars/datacenter-2.  More datacenters can be 
added by following this pattern.

Running the playbook: 
```
ansible-playbook -i hosts site.yml
```



