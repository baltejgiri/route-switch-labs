# Ansible on Linux Virtual Machine

Ansible is required to be installed on the linux virtual machine, I have used **Ubuntu 24.04.03 LTS".

## Network Connectivity

Verify if your VM has connectivity to internet, connection to Internet is important to download packages.

## Update Ubuntu VM
Make sure your VM is up-to-date to make sure there are no dependency issues installing ansible. Ubuntu uses the package called ```apt``` so running following commands can update and upgrade the packages. It is important to make sure VM ha

```bash
$ sudo apt update
$ sudo apt upgrade
```

## Install Ansible

Ansible can be installed on ubuntu following [ansible documentation](https://docs.ansible.com/projects/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-ubuntu).

```bash
$ sudo apt install software-properties-common
$ sudo add-apt-repository --yes --update ppa:ansible/ansible
$ sudo apt install ansible
```
### Verify Ansible

First let's verify if ansible is installed,

```bash
$ ansible-console -version
```

Expected output
```bash
$ ansible-console --version
ansible-console [core 2.16.3]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/girib/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  ansible collection location = /home/girib/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible-console
  python version = 3.12.3 (main, Aug 14 2025, 17:47:21) [GCC 13.3.0] (/usr/bin/python3)
  jinja version = 3.1.2
  libyaml = True
```
At this point, we have successfully verified ansible is installed on the virtual machine.
