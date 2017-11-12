---
title: "Terraform - vSphere - CentOS 7 Customization Issues"
date: "2017-11-12 01:04"
categories:
  - Automation
tags:
  - Terraform
  - vSphere
  - CentOS
---

I was recently working on a project to spin up about ~50 VMs which were all based
on [CentOS](https://www.centos.org/) 7.4 and ran into an issue during the
customization phase failing. The error I was receiving was:

    "Customization of the guest operating system 'centos64guest' is not supported
    in this configuration. Microsoft Vista and Linux guests with a Logical
    Volume Manager are support only for recent ESX host and VMware Tools versions"

Now this was not the first time I have seen this over the years but honestly it
had been a while. So I am documenting this here in case I run into this again
and of course forget how to resolve the issue.

The vSphere template that I was using initially did have LVM for the volume manager
so I built a new fresh template using the [CentOS Minimal ISO](http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1708.iso)
and did not use LVM but rather just standard partitioning. I then installed
`open-vm-tools` and tried provisioning with Terraform once again. And of course
ran into the same issue again. After a bit of trying (~3 hours) to figure out
if the issue was with Terraform or vSphere I finally narrowed down that it was
actually on the vSphere side. My next attempt I converted the template back to
a VM, uninstalled `open-vm-tools` and then decided to use the `open-vm-tools-deploypkg`
instead. The steps to install `open-vm-tools-deploypkg` is as follows:

```bash
yum install wget
wget https://packages.vmware.com/tools/keys/VMWARE-PACKAGING-GPG-DSA-KEY.pub
wget https://packages.vmware.com/tools/keys/VMWARE-PACKAGING-GPG-RSA-KEY.pub
rpm --import VMWARE-PACKAGING-GPG-DSA-KEY.pub
rpm --import VMWARE-PACKAGING-GPG-RSA-KEY.pub
```

Now create `/etc/yum.repos.d/vmware-tools.repo` with the following:

```bash
[vmware-tools]
name = VMware Tools
baseurl = https://packages.vmware.com/packages/rhel7/x86_64/
enabled = 1
gpgcheck = 1
```

And now install `open-vm-tools-deploypkg`:

```bash
yum install open-vm-tools-deploypkg
```

After installing the `open-vm-tools-deploypkg` I converted the VM to a template
once again and attempted to provision using Terraform. This time I got past the
error that I was originally getting but this time the VMs would spin up and the
customization would run but the VM hostnames and static IP addresses were not
being configured. I was making progress but why was this not working still? So
I did some more Google searches and found a blurb that triggered my memory on this,
**Perl**, `Perl` is a requirement for the customization to work correctly. So I converted
the template to a VM once again and spun it up, and sure enough, Perl was not
installed. So after installing perl:

```bash
yum install perl
```

I converted the VM back to a template and kicked off the Terraform provisioning
once again. And sure enough, there it went, worked as it should and only took
about ~5 minutes to spin up all of the VMs, configure their hostnames and
static IP addresses as desired.

So there you have it. Hopefully I will remember this post or at least find it
through Google whenever I run into this issue again. Or maybe it will also be
useful to someone else.

Enjoy!
