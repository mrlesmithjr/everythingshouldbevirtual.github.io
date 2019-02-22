---
title: "VMware Workstation - Enter Serial Number From Linux CLI"
date: "2019-02-21 19:05:00"
categories:
  - Virtualization
tags:
  - VMware Workstation
---

Well, this was a fun exercise. While building out some VMs to execute Packer
image builds for vSphere, I installed VMware Workstation via Ansible and forgot
to add the serial number as part of the installation. Yes, you read that correct,
running Packer builds inside a vSphere VM, within VMware Workstation. All kicked
off via GitLab CI pipelines. So, anyways, back to the issue at hand. I needed
to enter a functional license key to make Packer happy with VMware Workstation.
Well, something that one would think would be relatively easy, was not so much.
At least for me. Because there was not a GUI, I had to figure out the CLI command
to execute. Google did not provide me with anything very useful, but finally
pieced some stuff together which ended up working. So, here it is, the overly
complex command:

```bash
sudo /usr/lib/vmware/bin/licenseTool enter XXXXX-XXXXX-XXXXX-XXXXX-XXXXX "" "" 15.0+ "VMware Workstation" /usr/lib/vmware
```

There is a command `vmware-license-enter.sh` which did not work at all, but
looking at the contents of the script, allowed me to figure out the correct
command to run.

So, there you have it! Hopefully this will be of some use to others, but more of
a bookmark for me!

Enjoy!
