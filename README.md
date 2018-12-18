# DSE Ansible Automation


* Requires Ansible 1.2 or newer
* Expects CentOS/RHEL 6 or 7 hosts

These playbooks deploy a very basic implementation of Datastax Enterprise, version 6.0.4. To use them, first edit the 
"hosts" inventory file to contain the hostnames of the machines on which you want DSE deployed, and edit the 
group_vars/all file to set any DSE configuration parameters you need.

This playbook creates two data centers

Running the playbook: 
```
ansible-playbook -i hosts site.yml
```



