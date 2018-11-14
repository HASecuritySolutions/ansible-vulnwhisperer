# Ansible deployment for VulnWhisperer

This is a small collection of [Ansible](https://www.ansible.com/) playbook and roles used to install, configure and manage a
[VulnWhisperer](https://github.com/HASecuritySolutions/vulnwhisperer) installation with the [ELK stack](https://www.elastic.co/elk-stack).

## Code organization

There is a number of ansible playbook used to install and provision each of the hosts (or the same host for different roles). Please refer to
each playbook section below.

#### ansible.cfg

Main ansible configuration file, contains some options like the remote username, where to find the roles directory and
the inventory file name. The only variables that should ever need to be customized are the `remote_user` and `host_key_checking`
to specify the user to use on the remote system (requires *sudo* access) and if the remote host SSH key must be validated
before proceeding.

#### hosts

The `hosts` file contains the list of hosts that we want to deploy Vulnwhisperer on. There is a default `[ec2]` tag under which
any number of hosts can be configured. Please refer to the official [inventory documentation](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)
for further information.

#### provision.yml

The main playbook, it's usually used with the `ansible-playbook` command as follows:

```
ansible-playbook provision.yml
```

The playbook will prompt for an install option that can either be `install` or `update`. Each does what the 
name suggests. A path to the configuration file will be requested. This choice has been made to ensure that 
there is no coupling between the provisioning process and the configuration one, giving the user full control
on what configuration file to use and where to have it reside on the host running the playbook.

The playbook will then perform some basic sanity checking (in the `pre_tasks` section) to make sure the inputted
variables are present and then call the roles that will actually perform the provisioning. 

If you need to customize the `ELK` installation variables please refer to the role section below.

#### ssh.config

The SSH configuration that ansible will use to reach the remote host. The reason for this file is to
allow the user to customize the local SSH configuration, for example to specify the SSH key to use
to authenticate to the remote host (it can be a symbolic link) or a `ProxyCommand`.

## Roles

There are a number of roles in this ansible tree, they are detailed below.

### Elasticsearch

This roles comes from the upstream elastic ansible [repository](https://github.com/elastic/ansible-elasticsearch) and has been installed via
the `ansible-galaxy` tool. Please refer to the [official documentation](https://galaxy.ansible.com/) for more information.
The various configuration options for this role can be specified in the `provision.yml` playbook directly by editing the list of 
variables or can be passed from the command line. Please refer to the upstream role documentation for more information.

*Note* vulnwhisperer at the time of writing only supports Elasticsearch version 5.x

### Kibana

This roles comes from the upstream kibana ansible [repository](https://github.com/geerlingguy/ansible-role-kibana) and has been installed via
the `ansible-galaxy` tool. Please refer to the [official documentation](https://galaxy.ansible.com/) for more information.

### Vulnwhisperer

This is the main role that configures the vulnwhisperer software. It creates a number of directories in which it clones
the repository from mainline, creates a Python [virtualenv](https://virtualenv.pypa.io/en/latest/) virtualenv in which
it installs all the required dependencies and Vulnwhisperer itself. The role supports two tags, `install` and `update` that
do exactly what the name suggests.
