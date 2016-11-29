# Shared Ansible Roles

This repository contains some Ansible roles I use across servers (mostly local development servers managed with Vagrant). They all assume that the servers are running Ubuntu 16.04.

## Using Common Roles

The easiest way to use these roles is to clone the repo then add an "ansible.cfg" file in the current directory where you run ansible (same directory as Vagrantfile if your're using Vagrant) that contains the following.

```
[defaults]
roles_path = /path/to/shared-ansible-roles
```

For more information on the ansible.cfg file see the [ansible docs](http://docs.ansible.com/ansible/intro_configuration.html#configuration-file).
