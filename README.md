# vagrant-hadoop

Use Vagrant to create Hadoop virtual machines and use Ansible to install all requirement softwares

## Requirements

* Ruby 2
* Vagrant
* Ansible

## Start

All VMs will default install below softwares

* Ubuntu 16.04
* Hadoop 2.8.1
* Java 8
* Python 3

### Start buil VMs
```
vagrant --provision up
```
### Vagrant

* Start VM

```
vagrant up
```

* Run provision
```
vagrant provision
```

* Shutdown VM
```
vagrant halt
```

* Destroy VM
```
vagrant destroy -f
```

### Ansible

You can install software through ansibile cli

```
ansible-playbook -i hosts main.yml
```

## How to change your SSH keys

Put you ssh private and public keys in roles/common/templates, these keys will deploy to VMs if you want to use yourself.