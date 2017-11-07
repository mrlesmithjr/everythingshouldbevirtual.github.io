---
title: "StackStorm Series - Getting Started"
date: "2017-11-06 00:54"
categories:
  - Series
tags:
  - StackStorm
---

As mentioned in the previous post [StackStorm Series - Intro](https://everythingshouldbevirtual.com/series/stackstorm-series-intro/)
we will be working through a series of posts in regards to [StackStorm](https://stackstorm.com/).
In this post we will review the underlying components which will be used to build
out our test environment. This environment is based on [Vagrant](https://www.vagrantup.com/)
and all of the provisioning will be done with [Ansible](https://www.ansible.com).
We will be using the [ansible-stackstorm](https://github.com/mrlesmithjr/ansible-stackstorm)
Ansible role that I created about 2 years ago and from time to time keep it updated
to ensure that it is still functional. This repo also contains all of the components
required to get us started and begin our little journey with the series.

**So let's get started!**

## Setup Project Folder

The first thing we need to do is clone the [ansible-stackstorm](https://github.com/mrlesmithjr/ansible-stackstorm)
repo to our project folder. I will be using `~/projects/stackstorm-series`
for this series. Feel free to use whatever location you choose but make sure to
adjust for your location throughout the series.

### Create Project Folder

Let's create our project folder:

```bash
mkdir -p ~/projects/stackstorm-series
```

Now change directories into `~/projects/stackstorm-series`:

```bash
cd ~/projects/stackstorm-series
```

### Clone Repo

Now we need to clone the [ansible-stackstorm](https://github.com/mrlesmithjr/ansible-stackstorm)
repo into our project folder:

```bash
git clone https://github.com/mrlesmithjr/ansible-stackstorm
...
Cloning into 'ansible-stackstorm'...
remote: Counting objects: 622, done.
remote: Total 622 (delta 0), reused 0 (delta 0), pack-reused 622
Receiving objects: 100% (622/622), 151.56 KiB | 1.70 MiB/s, done.
Resolving deltas: 100% (203/203), done.
```

## Explore Project Folder

Now that we have cloned down the repo into our project folder let's explore the
contents a bit.

You will first want to change directories into `ansible-stackstorm`:

```bash
cd ansible-stackstorm
```

### Parent Repo Structure

The parent repo structure is actually the Ansible role for StackStorm. We
will be using this role for our series but it will be used a little bit differently.
I will explain this more further down. But to elaborate on the parent folder
structure, you will notice it is just a standard Ansible role folder structure
with the exception of the `Vagrant` folder.

### Vagrant Folder

Within the Vagrant folder you will find all of the goodies we will use for the
series.

> NOTE: If you are interested in a little more background on how I use Vagrant
> for Ansible scenario testing and etc. you can checkout the following
> post [Vagrant Box Templates](https://everythingshouldbevirtual.com/virtualization/vagrant-box-templates/).

If you checkout `Vagrant/roles` you will see that all of the roles required for
our series are included here. And you will also notice that the `ansible-stackstorm`
role is actually a symlink to the parent repo folder:

```bash
total 8
drwxr-xr-x  11 larry  staff  374 Nov  6 01:07 .
drwxr-xr-x  15 larry  staff  510 Nov  6 01:07 ..
drwxr-xr-x  11 larry  staff  374 Nov  6 01:07 ansible-etc-hosts
drwxr-xr-x   8 larry  staff  272 Nov  6 01:07 ansible-manage-ssh-keys
drwxr-xr-x  11 larry  staff  374 Nov  6 01:07 ansible-mongodb
drwxr-xr-x  14 larry  staff  476 Nov  6 01:07 ansible-nginx
drwxr-xr-x  10 larry  staff  340 Nov  6 01:07 ansible-nodejs
drwxr-xr-x  15 larry  staff  510 Nov  6 01:07 ansible-postgresql
drwxr-xr-x  16 larry  staff  544 Nov  6 01:07 ansible-rabbitmq
lrwxr-xr-x   1 larry  staff    6 Nov  6 01:07 ansible-stackstorm -> ../../
drwxr-xr-x  10 larry  staff  340 Nov  6 01:07 ansible-users
```

This is just a little trick to allow for testing and also not to include the
parent role which ends up with a never ending folder tree if we were to actually
pull the `ansible-stackstorm` role down and place it along with the rest of the
roles.

### Vagrant Environment

The first thing we will look at is what we will be spinning up initially for
our series. This will no doubt change a bit as we go through the series but will
be as below for our initial deployment.

#### nodes.yml

We have a `nodes.yml` file which defines our environment, so let's take a quick
look at that.

`nodes.yml`

```yaml
---
- name: node0
  ansible_groups:
    - stackstorm_server
  box: mrlesmithjr/xenial64
  desktop: false
  # disks:
  #   - size: 10
  #     controller: "SATA Controller"
  #   - size: 10
  #     controller: "SATA Controller"
  interfaces:
    - ip: 192.168.250.10
      auto_config: true
      method: static
  #   - ip: 192.168.1.10
  #     auto_config: false
  #     method: static
  #     network_name: network-1
  mem: 2048
  provision: true
  vcpu: 1
  # port_forwards:
  #   - guest: 80
  #     host: 8080
  #   - guest: 443
  #     host: 4433
- name: node1
  ansible_groups:
    - stackstorm_client
  box: mrlesmithjr/xenial64
  desktop: false
  # disks:
  #   - size: 10
  #     controller: "SATA Controller"
  #   - size: 10
  #     controller: "SATA Controller"
  interfaces:
    - ip: 192.168.250.11
      auto_config: true
      method: static
  #   - ip: 192.168.1.10
  #     auto_config: false
  #     method: static
  #     network_name: network-1
  mem: 512
  provision: true
  vcpu: 1
  # port_forwards:
  #   - guest: 80
  #     host: 8080
  #   - guest: 443
  #     host: 4433
```

What we have defined above is that we will be spinning up the following nodes:

-   node0
    -   stackstorm_server
    -   host only network with ip `192.168.250.10`
    -   2GB memory
    -   1 vcpu
-   node1
    -   stackstorm_client
    -   host only network with ip `192.168.250.11`
    -   512MB memory
    -   1 vcpu

The `ansible_groups` defines which Ansible groups the node will be a member of
when we spin up the environmet for our provisioning to run against.

#### playbook.yml

The following is the Ansible playbook that will be executed when we spin up the
environment.

{% raw %}

```yaml
---
- hosts: all
  become: true
  vars:
    etc_hosts_add_all_hosts: true
    pri_domain_name: 'test.vagrant.local'
  roles:
    - role: ansible-etc-hosts

- hosts: stackstorm_server
  become: true
  vars:
    nodejs_version: 6.x
    pri_domain_name: 'test.vagrant.local'
    stackstorm_install_packs: true
  roles:
    - role: ansible-mongodb
    - role: ansible-nginx
    - role: ansible-postgresql
    - role: ansible-rabbitmq
    - role: ansible-nodejs
    - role: ansible-stackstorm
  tasks:

- hosts: stackstorm_client
  become: true
  vars:
    create_users:
      - user: '{{ stackstorm_user }}'
        comment: 'Stackstorm SSH User'
        generate_keys: true
        # P@55w0rd
        pass: '$6$8tMUxKP33/$Fb/hZBaYvyzGubO9nrlRJMjUnt3aajXZwxCifH9NYqrhjMlC9COWmNNFiMpnyNGsgmDeNCCn2wKNh0G1E1BBV0'
        preseed_user: false
        state: 'present'
        sudo: true
        system_account: false
    enable_manage_ssh_keys: true
    manage_ssh_keys:
      - remote_user: '{{ stackstorm_user }}'
        state: present
        keys:
          - ./{{ stackstorm_user }}@node0.pub
    pri_domain_name: 'test.vagrant.local'
    stackstorm_user: 'stanley'
  roles:
    - role: ansible-users
      tags:
        - "stackstorm-ssh-keys"
        - "stackstorm-user"
    - role: ansible-manage-ssh-keys
      tags:
        - "stackstorm-ssh-keys"
        - "stackstorm-user"
  tasks:
```

{% endraw %}

So there you have it, we have just covered the basics of the underlying environment
that we will be spinning up for this series. In the next post we will be spinning
up the environment so that we can begin to start exploring StackStorm!

Enjoy!
