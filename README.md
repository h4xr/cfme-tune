# cfme-tune

A collection of playbooks for Performance Tuning and health check of CFME/ManageIQ appliances.

**Table of Contents**
========
- [Installation](#installation)
- [Configuration](#configure)
 - [SSH Config File](#ssh-config-file---ssh-configlocal)
 - [Inventory File](#inventory-file---hostslocal)
 - [Group_vars all.yml](#group_vars---alllocalyml)
- [Playbook Directories](#playbook-directories)
- [Which playbooks should I use?](#which-playbooks-should-i-use)
- [Adhoc Scripts](#adhoc-scripts)

## Installation
To install:  
1. git clone this repository  
2. Install Ansible  
  * http://docs.ansible.com/ansible/intro_installation.html#getting-ansible
3. Configure your playbooks by editing ssh-config.local, hosts.local, and group_vars/all.local.yml

## Configure
The playbooks are broken up into several directories to organize them.  They all depend on several files which must be setup correctly before running the playbooks.  These files are ssh-config.local, hosts.local, and group_vars/all.local.yml  Templates of these files are provided: ssh-config.local.template, hosts.local.template, and group_vars/all.yml

To begin using the playbooks copy the templates and edit your copies as follows:

```
[root@perf ansible]# cp ssh-config.local.template ssh-config.local
[root@perf ansible]# cp hosts.local.template hosts.local
[root@perf ansible]# cp group_vars/all.yml group_vars/all.local.yml
```

Now edit each of the 3 files starting with ssh-config.local, then hosts.local and lastly group_vars/all.local.yml

### SSH Config File - ssh-config.local

Your ssh-config.local file informs ssh how to connect to a specified host via a name (Rather than DNS).  Note that the contents in the template is just an example, you must at a minimum adjust the ip address to map to your appliance. Your inventory file (hosts.local) must map to the hosts in your ssh config file.  The ansible.cfg file informs ansible to use your ssh-config.local file to correctly map a host in the inventory to what ssh is configured to connect to.

[Example ssh-config File](ssh-config.local.template)

### Inventory File - hosts.local

Again, make sure each entry in the ssh-config.local file maps to a host listed in your ansible inventory file (hosts.local).  See templates for mapping.  Appliances that run a database should be placed under "[cfme-vmdb]".  Appliances that only host workers and connect to an external database should be placed under "[cfme-worker]".  Appliances that you want to deploy All-In-One Performance Monitoring should be placed under "[cfme-all-in-one]".  Hosts that run RHEVM and are used to provision other appliances and templates should be placed under "[rhevm]".  Appliances that have both rhevm and vdsmfake and are used to manage simulated environments should be placed under "[fake-rhevm]".  As you are working you can "comment" out appliances (via "#" in front of the name) as needed to make sure playbooks are only applied against the expected appliances/machines.

[Example Inventory File](hosts.local.template)

### Group_vars - all.local.yml

The ansible vars file is located in group_vars/all.yml.  It should be copied and then edited to parameters which match your environment.  This allows you to make overrides to defaults in the all.yml file without introducing them as git changes.  Each playbook will override default variables by including group_vars/all.local.yml last.  This also gives you the option to only add the variables you want to override to your local vars file.

Each entry under appliances must match an entry in your ansible inventory file (hosts.local) as the ansible variable `inventory_hostname` is used to determine if the configuration is setup for that specific appliance.  Since using ip addresses would be less than ideal, This repo uses an ssh config file (ssh-config.local) to address each machine as a name.  That inventory name is also used for the hostname to make it easy to identify what appliance you have ssh-ed to.

[Example group_vars/all.local.yml](group_vars/all.yml)


## Playbook Directories

### [Workloads](workloads/)

**WIP** - These playbooks will automate adding a provider, configuring roles, and configuring advanced settings on the appliance.  It will also include some basic workloads such as queuing a refresh, provisioning, smart state analysis, etc...


## Which playbooks should I use?

* If you want to create a new virtual machine:
  * [Create](create/)
* If you want to install CFME/ManageIQ to a blank/new virtual machine:
  * [Configure](configure/)
* If you want to clean logs/reset a machine to the point after configuration:
  * [Cleanup](cleanup/)
* If you want to install tools used for testing:
  * [Install](install/)
* If you want to test the performance of CFME/ManageIQ:
  * The [Python Test Framework](../cfme-performance/) currently provides this functionality instead.

## Adhoc scripts

You could use the scripts under `scripts/` folder as following:

1. `count_msg_state_in_range.sh`: Run this `./count_msg_state_in_range.sh -t ok` to find last 1 hour's worth of delivered messages in [ok] state. This script handles both text and compressed (.gz) evm log files. Other usage examples: https://gist.github.com/arcolife/6d6bda31ad3685e05cb1f91c34d97d4b

2. `cleanup_cfme.sh`: Run it to cleanup an appliance and start fresh. Check if your postgresql services name matches `rh-postgresql95-postgresql` by default. For Cloudforms 4.5 and 4.6, this should be default

3. `nodes_ops.sh`: not a "script" script. More like a helper catalogue of shell scripts to handle nodes on an openshift infrastructure

4. `mem_script.sh`: List of bash hacks to produce RSS / SWAP output.

5. `get_ems_refresh_avg.sh|get_metric_capture_avg.sh`: old scripts to get EMS Refresh / C&U timings. Could either modify them or use this excellent pack of Ruby scripts developed by pemcg -> https://github.com/RHsyseng/cfme-log-parsing

