# DSE Ansible Automation
## Basic Overview
These Ansible playbooks deploy a very basic implementation of DataStax Enterprise with Graph, version 6.7 or 6.8. 

For more information on DataStax:
* [DataStax](www.datastax.com)
* [DataStax Graph](https://www.datastax.com/products/datastax-enterprise-graph)

Though very basic, these playbooks can be adapted as needed for more complex situations.

### Playbooks

There are three main playbooks:
* **dseservers.yml** installs, configures and starts DSE on DSE nodes.
* **opscenterserver.yml** installs, configures and starts OpsCenter on Opscenter node.
* **site.yml** runs the two playbooks above in order.


## Initial Setup
### Managed Nodes (Target Server) Setup
These are the servers you will manage with Ansible.  In this case, Ansible is strictly used for configuration and deployment 
not for provisioning infrastructure.  Infrastructure must be provisioned separately.  The following steps are for manually 
provisioning infrastructure via the EC2 Management Console in AWS.

1. Launch instances via the EC2 Management Console, following the recommended types by environment. Set security according 
the DSE documentation and make sure to save your SSH key as something you will remember.
2. Make note of all instance Public DNS's and Public IP's.  Setting up an Elastic IP for each instance is recommended.
3. Determine which nodes will go into each data center and which of those nodes will be seed nodes.  

**Note that these playbooks expect CentOS/RHEL 6 or 7 hosts.**

### Inventory Setup
Inventories are a list of managed nodes. The inventories folder allows for multiple inventories to be used and saved with 
the same playbooks.  This allows flexible separation of different environment, projects and/or versions of DSE.  There are two sample projects 
provided.  The first folder, "dse-6.7-2-dc", represents a two data center cluster that runs DSE 6.7.  Both the associated 
variables and the inventory (hostfile) are included in this folder.

#### Hostfile
The hostfile organizes the inventory into their respective groups and hierarchies.  Inventory can belong to multiple groups.
For example, seed nodes will be in both the [seednodes] group and each will belong to a [datacenter-n] group.  See example 
of a host file below, along with the corresponding hierarchy of the groups.  

```
├── dseservers
│   ├── datacenter-1
│   ├── datacenter-2
│   └── seednodes
└── opscenterserver

```
   

```
[datacenter-1]
public-dns-1-a node_public_ip=public-ip-1-a
public-dns-2-a node_public_ip=public-ip-2-a
public-dns-3-a node_public_ip=public-ip-3-a

[datacenter-1]
public-dns-1-b node_public_ip=public-ip-1-b
public-dns-2-b node_public_ip=public-ip-2-b
public-dns-3-b node_public_ip=public-ip-3-b

[seednodes]
public-dns-1-a
public-dns-1-b

[dseservers:children]
datacenter-1
datacenter-2

[dseservers:vars]
ansible_ssh_common_args='-o StrictHostKeyChecking=no'

[opscenterserver]
public-dns-opsc node_public_ip=public-ip-opsc

[opscenterserver:vars]
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

In order to add nodes to a data center, simply add the additional Public DNS and Public IP to the correct group. 
In order to add additional data centers, update the host file to have a new group (ie, [datacenter-3]) and follow the directions 
below in Variables to ensure correct configuration.  Don't forget to add (at least) one of the nodes from the new data center to the 
[seednodes] group. 


#### Variables
Notice that for each group name, there is an associated variable file under the group_vars folder.  In each of these files
you can set various parameters for dse configuration files, which can be found under the roles/configure_dse/templates folder. Currently, 
only the most basic settings have been parameterized. You can add more by editing the .j2 files in the roles/configure_dse/templates
folder.

If you create a new data center within the hostfile, a corresponding file within the group_vars folder must also be created, using the 
group name as the file name.  Simply copy one of the example data center group variable files (ie, group_vars/datacenter-1), edit the name
and update the parameters to represent the type of data center you would like to bring up.  Make sure you change the name so that
a new data center is created as well.

There is also a variable file named "all," which applies to all of the hosts and groups, including variables such as DSE version and 
basic cluster information.  This is also the file which you must set your DSE username and password for access to DSE yum repos.  Consider using
[Ansible vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html) for storing your password.

If you want to set up a cluster using a version of DSE that does not currently have a public YUM repo (ie, 6.8), email me for 
internal instructions. 

### Control Node Setup
The controller is the computer or node from which the playbooks are run. Currently Ansible can be run from any machine with 
Python 2 (version 2.7) or Python 3 (versions 3.5 and higher) installed. Note, Windows isn’t supported for the control machine.
The preferred way for installing Ansible on Mac is via Pip:

```
sudo pip install ansible
```

## Running the Playbook


To configure and deploy both DSE and DSE Opscenter, run the site.yml playbook with the associate inventory file to deploy to:

```
ansible-playbook -i dse-6.7-2-dc/hosts --private-key PRIVATE_KEY_FILE.pem site.yml 
```

You can also set the private key file in the variables file, or add the key to ssh-agent to avoid using the --private-key
option.

Another useful option is the "Check Mode", which performs a dry run of the playbook.  When ansible-playbook is executed 
with --check it will not make any changes on remote systems. Instead, any module instrumented to support ‘check mode’ 
(which contains most of the primary core modules, but it is not required that all modules do this) will report what changes 
they would have made rather than making them:

 ```
 ansible-playbook -i dse-6.7-2-dc/hosts --check 
 ```









