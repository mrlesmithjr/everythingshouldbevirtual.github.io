---
title: KVM - VM Templates
categories:
  - Virtualization
tags:
  - KVM
redirect_from:
  - /kvm-vm-templates
---

It has been a minute since I had done any KVM based VMs so I wanted to
share some little tidbits on creating your KVM templates. I will be
focusing on Debian/Ubuntu here for now so YMMV.

**Update: 01/07/2017 - CentOS added**

First, let's install a few pre-req packages:

```bash
sudo apt-get install -y virtinst libosinfo-bin libguestfs-tools virt-top
```

Below is a shell script that I have created which will allow me to
provision my baseline template easily. You will need to adjust based on
your needs. I am using **_br0_** for my **_NETWORK_BRIDGE_** which
allows me to connect directly to my external network rather than a
default NAT connection. So you may want to use the built-in
default **_virbr0_** instead.

{% gist mrlesmithjr/16a3bd7fc312e155eeb0c4f79df8ffa8 %}

Below are _preseed_ files which work for Ubuntu 16.04 and Debian Jessie
as they are based on systemd and on reboot using virsh console the login
prompt is enabled to work correctly. That bit of magic works in the
**_late_command_** section at the end. You will also want to change the
default user and password as they are defined as **_ubuntu/ubuntu_** and
**_debian/debian_**.

{% gist mrlesmithjr/20ee8c8add73cde213fc498c0f167f72 %}

{% gist mrlesmithjr/db036fe3c7426307a5bb58550ea57aa3 %}

And this little script can be executed within your VM in order to prep
it to be used as a template by cleaning up some important things. Or
better yet, if you look closely at the above _preseed_ files I have
included those important cleanup steps in the **_late_command_** steps,
so you do not have to run this script at all if you use the _preseed_
files.

{% gist mrlesmithjr/8670ff7bdcec5ba05b619b46d51da1ea %}

So how do we wrap all of this up into an easy format to consume? You can
quite simply copy/paste the script and appropriate _preseed_ file into
your folder structure that you use. Make sure to name the preseed file
as **_preseed.cfg_** otherwise it will not work with **_virt-install_**.
Also, make sure to modify the variables
for **\*DISK_PATH**,\* **_PRESEED_FILE_**, and **_NETWORK_BRIDGE _**as
appropriate. And once you have completed that you are ready to run the
script and watch the VM build and get ready for template usage. Also
note, that when using the _preseed_ file with our script, the VM will
actually shutdown at the end of the installation. This is the desired
state as we want everything including our non-generated SSH keys to be
clear for template usage.

Build template:

```bash
chmod +x libvirt_install_vm.sh
sudo ./libvirt_install_vm.sh
```

Clone template (Note: Replace OURNEWVM with the name you desire for the
cloned VM):

```bash
sudo virt-clone -o ubuntu1604 -n OURNEWVM -f /var/lib/libvirt/images/OURNEWVM.qcow2
sudo virsh start OURNEWVM --console
```

And there you have it. A new shiny KVM based Ubuntu template ready for
cloning. Stay tuned as I will be taking this process further with some
Ansible automation to fully deploy and etc.

**Update: CentOS Info**

Below you will find the script and ks.cfg files to use with CentOS.

{% gist mrlesmithjr/293d8314c7f08cfc6412e8df40e53a91 %}

{% gist mrlesmithjr/c3b5acd5e1861fca593663b045cc75b8 %}

Enjoy!
