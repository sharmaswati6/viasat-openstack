# viasat-openstack
Optional Designate module, that provides DNSaaS services for Rackspace Openstack Private Cloud

# openstack-ansible Designate Integration

The viasat-openstack repo includes files to integrate Designate module support with the existing [openstack-ansible](https://github.com/openstack/openstack-ansible) set of Ansible playbooks and roles.

# Liberty Support

Assuming stable/liberty openstack-ansible is already up (configured and deployed), the new Designate module can be integrated to the existing setup by following these instructions.

# Openstack-Ansible ('viasatd': Files for Integrating Designate module functionality)

Note:

Some of the editted files have the following statements-
* `# NEW LINE - "VIASAT"`- This statement is added as a comment, against the single-lines to be included for designate
* `# NEW LINES BEGIN - "VIASAT"` - This statement is added as a comment, at the beginning of every multi-lines block included for designate
* `# NEW LINES END - "VIASAT"` - This statement is added as a comment, at the end of every multi-lines block included for designate

Playbooks:

* `os-designate-install.yml` -  NEW file that consists of the Designate components to be installed on the host machines, the designate roles, tasks and vars to be used.
* `setup-openstack.yml` - includes the reference of the new designate playbook `os-designate-install.yml`
* `utility-install.yml` - includes the new pip package `python-designateclient` to be installed
* `run-playbooks.sh` - includes the new designate playbook `os-designate-install.yml`
* `hosts.yml` - includes the parameters for Designate services
* `openstack_services.yml` - includes the Designate Git repo reference
* `haproxy_config.yml` - includes designate_api haproxy settings
* `os_designate role` - NEW directory added, consisting of Designate tasks, template files, default connection strings and designate handlers
* `openrc` - include alias for Designate as well 

Scripts:

* `run-playbooks.sh` - includes Designate Playbook reference in this script, for deploying complete openstack-ansible in one go

# Openstack_Deploy ('viasatd': Files Explained for the Basic Setup of Designate)

* `conf.d/designate.yml` - includes host/node details for designate
* `env.d/designate.yml` - includes the container details for Designate and the designate services mapped to each of them
* `user_extras_secrets.yml` - includes the designate parameter credentials
* `user_variables.yml` - includes the ceilometer setting for designate
* `openstack_user_config.yml` - includes the node settings added for dnsaas_hosts 

# Add & Edit Designate Files

1. Clone the viasat-openstack repository:
   `cd /opt && git clone --recursive https://github.com/sharmaswati6/viasat-openstack`
2. Add new designate files-
   a) `cd viasat-openstack`
   b) `cp viasatd/etc/openstack_deploy/user_extras_*.yml /etc/openstack_deploy`
   c) recursively copy the designate configuration files:
      `cp -R viasatd/etc/openstack_deploy/conf.d /etc/openstack_deploy/conf.d`
   d) recursively copy the designate environment setup files:
      `cp -R viasatd/etc/openstack_deploy/env.d /etc/openstack_deploy/env.d`
   e) copy designate playbook:
      `cp viasatd/opt/openstack-ansible/playbooks/os-designate-install.yml /opt/openstack-ansible/playbooks`
   f) recurisvely copy designate role:
      `cp -R viasatd/opt/openstack-ansible/playbooks/roles/os_designate /opt/openstack-ansible/playbooks/roles`
3. `cd /opt/openstack-ansible`
4. Edit `playbooks/setup-openstack.yml` according to `viasatd/opt/openstack-ansible/playbooks/setup-openstack.yml`
5. Edit `playbooks/utility-install.yml` according to `viasatd/opt/openstack-ansible/playbooks/utility-install.yml`
6. Edit `playbooks/vars/configs/haproxy_config.yml` according to `viasatd/opt/openstack-ansible/playbooks/vars/configs/haproxy_config.yml`
7. Edit `playbooks/defaults/repo_packages/openstack_services.yml` according to `viasatd/opt/openstack-ansible/playbooks/defaults/repo_packages/openstack_services.yml`
8. Edit `playbooks/inventory/group_vars/hosts.yml` according to `viasatd/opt/openstack-ansible/playbooks/inventory/group_vars/hosts.yml`
9. Edit `playbooks/roles/openstack_openrc/templates/openrc` according to `viasatd/opt/openstack-ansible/playbooks/roles/openstack_openrc/templates/openrc`
10. `cd /etc/openstack_deploy`
11. Edit `user_variables.yml` according to `viasatd/etc/openstack_deploy/user_variables.yml`
12. Edit `openstack_user_config.yml` according to `viasatd/etc/openstack_deploy/openstack_user_config.yml`

# Designate Installation Steps with Openstack-Ansible

1. Creates empty designate containers
   `sudo openstack-ansible setup-hosts.yml`
   Suppose, new container is created at <infr002_new_designate_container>
2. Setup expected environment for designate
   `sudo openstack-ansible setup-everything.yml --limit=infr002_ new_designate_container`
   (This will give pip install Error while installing designate) 
3. `ssh inside infr002 and lxc-attach infr002_ new_designate_container`. Run the following commands to install 'designate' inside this container- 
   `cd /usr/local/lib/python2.7/dist-packages`
   `git clone https://github.com/openstack/designate.git`
   `cd designate`
   `git checkout stable/liberty`
   `pip install -r requirements.txt -r test-requirements.txt --isolated`
   `python setup.py develop `
4. Again run, `sudo openstack-ansible setup-everything.yml --limit=infr002_ new_designate_container`
   (This will install the designate playbook 'os-designate-install.yml' successfully on this new container)
5. To Run the designate commands on the new designate container, do lxc-attach <infr002_new_designate_container>
   `source /root/openrc`
6. Try domain creation: `designate domain-create --name testing123.net. --email me@mydomain.com`
