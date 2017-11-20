# Ansible Role: Packer Debian/Ubuntu Configuration for Vagrant VirtualBox and VMware Workstation

[![Build Status](https://travis-ci.org/it-praktyk/ansible-role-packer-debian.svg?branch=master)](https://travis-ci.org/it-praktyk/ansible-role-packer-debian)

This role configures Debian/Ubuntu (either minimal or full install) in preparation for it to be packaged as part of a .box file for Vagrant/VirtualBox or Vagrant/Vmware_desktop deployment using [Packer](http://www.packer.io/).

The role was initially created by Jeff Geerling aka [geerlingguy](https://github.com/geerlingguy) but due to lack of accepting constructive updates ([PR #17](https://github.com/geerlingguy/ansible-role-packer-debian/pull/17), [PR #18](https://github.com/geerlingguy/ansible-role-packer-debian/pull/18)) I decided to fork it.

## Requirements

Prior to running this role via Packer, you need to make sure Ansible is installed via a shell provisioner, and that preliminary VM configuration (like adding a vagrant user to the appropriate group and the sudoers file) is complete, generally by using a Kickstart installation file (e.g. `ks.cfg`) or [preseeding](https://help.ubuntu.com/lts/installation-guide/s390x/apbs02.html) with Packer. An example array of provisioners for your Packer .json template looks something like:

```json
"provisioners": [
  {
    "type": "shell",
    "execute_command": "echo 'vagrant' | {{.Vars}} sudo -S -E bash '{{.Path}}'",
    "script": "scripts/ansible.sh"
  },
  {
    "type": "ansible-local",
    "playbook_file": "ansible/main.yml",
    "role_paths": [
      "/Users/jgeerling/Dropbox/VMs/roles/geerlingguy.packer-debian",
    ]
  }
],
```

The files should contain, at a minimum:

**scripts/ansible.sh**:

An example for Ubuntu 16.04

```bash
#!/bin/bash -eux
# Install Ansible repository and Ansible.
apt -y install software-properties-common
apt-add-repository ppa:ansible/ansible
apt-get update
apt-get install ansible
```

An example for Debian 8.8

```bash
#!/bin/bash -eux
# Install Ansible repository and Ansible.
apt -y install software-properties-common
echo "deb http://ppa.launchpad.net/ansible/ansible/ubuntu trusty main" | tee -a /etc/apt/sources.list
apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 93C4A3FD7BB9C367
apt -y update
apt -y install ansible
```

**ansible/main.yml**:

```yaml
---
- hosts: all
  sudo: yes
  gather_facts: yes
  roles:
    - it-praktyk.packer-debian
```

You might also want to add another shell provisioner to run cleanup, erasing free space using `dd`, but this is not required (it will just save a little disk space in the Packer-produced .box file).

If you'd like to add additional roles, make sure you add them to the `role_paths` array in the template .json file, and then you can include them in `main.yml` as you normally would. The Ansible configuration will be run over a local connection from within the Linux environment, so all relevant files need to be copied over to the VM; configuratin for this is in the template .json file. Read more: [Ansible Local Provisioner](http://www.packer.io/docs/provisioners/ansible-local.html).

## Role Variables

Available variables are listed below, along with default values (see defaults/main.yml):

  -  vmware_install_open_vm_tools: no

      (VMware only) Using the `vmware_install_open_vm_tools` variable, you can select what kind of integration components will be installed into the VMware box. The default (`no`) installs VMware Tools, and not `open-vm-tools`.

      Read more:

      - [open-vm-tools](https://sourceforge.net/projects/open-vm-tools/)
      - [open-vm-tools on GitHub](https://github.com/vmware/open-vm-tools)
      - [VMware support for Open VM Tools (2073803)](https://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2073803)
      - [VMware Tools](https://kb.vmware.com/selfservice/search.do?cmd=displayKC&docType=kc&docTypeID=DT_KB_1_1&externalId=340)

  - remove_guest_tools_iso_file: yes

      If set to no allows retain VirtualBox Additions or VMware Tools iso file inside created Vagrant box - it allows retry build on error or update kernel and recompile required software based on the sources from these iso files.



## Dependencies

None.

## Example Playbook

```yaml
---
- hosts: all
  roles:
    - it-praktyk.packer-debian
```

## [Changelog](CHANGELOG.MD)

## License

MIT

## Authors Information

This role was created in 2014 by [Jeff Geerling](https://www.jeffgeerling.com/), author of [Ansible for DevOps](https://www.ansiblefordevops.com/).

Forked at November 17, 2017 by Wojciech Sciesinski
