# Shared Ansible Roles

This repository contains some Ansible roles I use across servers (mostly local development servers managed with Vagrant). They all assume that the servers are running Ubuntu 16.04.

## Using Common Roles

The easiest way to use these roles is to clone the repo then add an "ansible.cfg" file in the current directory where you run ansible (same directory as Vagrantfile if your're using Vagrant) that contains the following.

```
[defaults]
roles_path = /path/to/shared-ansible-roles
```

For more information on the ansible.cfg file see the [ansible docs](http://docs.ansible.com/ansible/intro_configuration.html#configuration-file).

## PHP Version

In order to be able to use different versions of PHP the variable "php_version" must be set. This can be 5.6, 7.0, 7.1, or 7.2 (any version supported by Ubuntu 16.04).

## Example

For a simple Vagrant setup for Drupal development using "foo" as the server/hostname do the following.

```
git clone git@github.com:karlkedrovsky/shared-ansible-roles.git
mkdir -p foo/provisioning
cd foo
```

Create the following files.

Vagrantfile. Adjust settings (e.g. hostname, IP, NFS settings) as needed.

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

module OS
    def OS.windows?
        (/cygwin|mswin|mingw|bccwin|wince|emx/ =~ RUBY_PLATFORM) != nil
    end

    def OS.mac?
        (/darwin/ =~ RUBY_PLATFORM) != nil
    end

    def OS.unix?
        !OS.windows?
    end

    def OS.linux?
        OS.unix? and not OS.mac?
    end
end

Vagrant.configure("2") do |config|
  config.vm.box = "geerlingguy/ubuntu1604"
  
  config.ssh.insert_key = false
  config.ssh.forward_agent = true

  config.vm.provider :virtualbox do |v|
    v.memory = 2048
    v.cpus = 1
    v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
  end

  config.vm.hostname = "foo"
  config.vm.network :private_network, ip: "10.1.0.20"

  # Assuming we're running on a mac or a linux host machine
  if OS.mac?
    config.vm.synced_folder "/Users/kkedrovsky/projects", "/projects", type: "nfs", nfs_export: false, nfs_udp: false
  else
    config.vm.synced_folder "/projects", "/projects", type: "nfs", nfs_version: 4, nfs_udp: false, nfs_export: false
  end

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "provisioning/playbook.yml"
    ansible.inventory_path = "provisioning/inventory"
    ansible.limit = "all"
  end
  
end
```

ansible.cfg

```
[defaults]
roles_path = ../shared-ansible-roles
```

provisioning/playbook.yml

```
---
- hosts: ua
  become: yes
  vars_files:
    - vars/drupal.yml
  vars:
    php_version: 7.1
  roles:
    - mysql
    - devtools
    - nginx
    - php-fpm
    - php-xdebug
    - node
    - drupal-db
    - drupal-web
```

provisioning/host_vars/foo

```
---
hostname: foo
```

provisioning/vars/drupal.yml

```
---
drupal_sites:
  foo:
    docroot: /projects/foo/docroot
```

After all that's done just just run the following to set up the VM and provision it.

```
vagrant up
```
