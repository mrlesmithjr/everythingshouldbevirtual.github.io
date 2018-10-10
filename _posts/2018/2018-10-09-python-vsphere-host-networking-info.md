---
title: "Python - vSphere Host Networking Info"
date: "2018-10-09 22:49"
categories:
  - Virtualization
tags:
  - Python
  - vSphere
---

Oh I love [Terraform](https://terraform.io) so so much. But why does vSphere
feel like a second class citizen? Is it because my use case is a real
snowflake? Maybe, maybe not! However, Terraform in most regards when working
with vSphere is awesome. Don't get me wrong even [Ansible](https://ansible.com)
has it downfalls with vSphere. Which is why about a year ago I mashed up a nice
little project [ansible-vsphere-management](https://github.com/mrlesmithjr/ansible-vsphere-management)
with Ansible, Terraform, and wait for it, PowerShell/PowerCLI. I wanted to
revisit this and once again explore where each tooling would hand off to the next
in an automated full stack deployment. The intent is complete hands off other
than kicking off the build and when complete you have a fully provisioned
infrastructure including applications running within VMs as well as containers
on top of Docker Swarm cluster (Eventually this would/could be Kubernetes). Now
do not get me wrong, I know well enough that not one single bit of tooling can
do it all from ground up. Which is where understanding the orchestration of
each tooling comes into play.

> NOTE: This is way before vCenter or anything else exists.

But here is where the fun already starts. In the very beginning of where initial
networking for vSphere is required you need to shuffle things around and lay
down some configurations. Terraform can handle the task of creating new vSwitches,
portgroups and adding pnics but you cannot touch `vSwitch0`. Is this the end
of the world? Nope. But.... I would love the ability to start with Terraform
at the lowest level and build up the stack as far as I can before handing off
to other tooling. As it stands right now, you would have to either bypass Terraform
initially and use Ansible to a level, handoff to Terraform, hand back off to
Ansible, and then potentially either Python or PowerShell/PowerCLI. Now I also
know that one could easily just start with PowerShell/PowerCLI and do the
majority of the heavy lifting but there is an awful lot of logic that needs to
go into this to ensure that nothing breaks the next time the automation kicks in.
But again, not the end of the world. We could even just kickstart the ESXi
installs and get everything to a certain degree before handing off to the
additional tooling that is required. But I digress, as usual.

The real point I am trying to make is that in order to successfully automate
full stack deployments one needs to really understand where each tooling excels
and where it equally fails. So to the point, make sure to explore various
tooling to understand where they come into play.

Now, what lead me to this post as I mentioned was the networking layer. I wanted
to bring [PyVmomi](https://github.com/vmware/pyvmomi) into the picture and do
some initial discovery of some ESXi hosts in regards to networking information.
I did some searching around and damnit if I could find anything close to what I
was looking for. So I started hacking away and finally started to put some useful
(at least for me) bits of information together. So I figured I would share it
with others if the need was there.

```python
#! /usr/bin/env python
from pyVim.connect import SmartConnect, SmartConnectNoSSL, Disconnect
from pyVmomi import vim
import json

vsphere_hosts = ['10.0.101.61', '10.0.101.62']
vsphere_password = 'VMw@re1'
vsphere_username = 'root'


def main():
    esxi_hosts = []
    for vsphere_host in vsphere_hosts:
        serviceInstance = SmartConnectNoSSL(host=vsphere_host,
                                            user=vsphere_username,
                                            pwd=vsphere_password)
        content = serviceInstance.RetrieveContent()

        host_view = content.viewManager.CreateContainerView(
            content.rootFolder, [vim.HostSystem], True)

        hosts = [host for host in host_view.view]
        for host in hosts:
            host_info = dict()
            host_pnics = capture_host_pnics(host)
            host_vnics = capture_host_vnics(host)
            host_vswitches = capture_host_vswitches(host)
            host_info.update(
                {'host': vsphere_host, 'hostname': host.name,
                 'pnics': host_pnics, 'vswitches': host_vswitches,
                 'vnics': host_vnics})
            esxi_hosts.append(host_info)

        Disconnect(serviceInstance)

    print json.dumps(esxi_hosts, indent=4)


# Capture ESXi host physical nics
def capture_host_pnics(host):
    host_pnics = []
    for pnic in host.config.network.pnic:
        pnic_info = dict()
        pnic_info.update(
            {'device': pnic.device, 'driver': pnic.driver, 'mac': pnic.mac})
        host_pnics.append(pnic_info)

    return host_pnics


# Capture ESXi host virtual nics
def capture_host_vnics(host):
    host_vnics = []
    for vnic in host.config.network.vnic:
        vnic_info = dict()
        vnic_info.update(
            {'device': vnic.device, 'portgroup': vnic.portgroup,
             'dhcp': vnic.spec.ip.dhcp, 'ip': vnic.spec.ip.ipAddress,
             'subnet_mask': vnic.spec.ip.subnetMask,
             'mac_address': vnic.spec.mac, 'mtu': vnic.spec.mtu})
        host_vnics.append(vnic_info)
    return host_vnics


# Capture ESXi host virtual switches
def capture_host_vswitches(host):
    host_vswitches = []
    for vswitch in host.config.network.vswitch:
        vswitch_info = dict()
        vswitch_pnics = []
        vswitch_portgroups = []
        for pnic in vswitch.pnic:
            pnic = pnic.replace('key-vim.host.PhysicalNic-', '')
            vswitch_pnics.append(pnic)
        for pg in vswitch.portgroup:
            pg = pg.replace('key-vim.host.PortGroup-', '')
            vswitch_portgroups.append(pg)
        vswitch_info.update(
            {'name': vswitch.name, 'pnics': vswitch_pnics,
             'portgroups': vswitch_portgroups, 'mtu': vswitch.mtu})
        host_vswitches.append(vswitch_info)

    return host_vswitches


if __name__ == "__main__":
    main()
```

The above currently will capture the initial pnics, vnics, vSwitches, and
portgroups for each ESXi host and return them in JSON. See where this might be
going? :)

So if I run the above I would get something in the likes of:

```json
[
  {
    "pnics": [
      {
        "device": "vmnic0",
        "mac": "00:23:7d:33:f0:6e",
        "driver": "bnx2"
      },
      {
        "device": "vmnic1",
        "mac": "00:23:7d:33:f0:5e",
        "driver": "bnx2"
      },
      {
        "device": "vmnic2",
        "mac": "e8:39:35:10:20:b9",
        "driver": "e1000e"
      },
      {
        "device": "vmnic3",
        "mac": "e8:39:35:10:20:b8",
        "driver": "e1000e"
      },
      {
        "device": "vmnic4",
        "mac": "e8:39:35:10:20:bb",
        "driver": "e1000e"
      },
      {
        "device": "vmnic5",
        "mac": "e8:39:35:10:20:ba",
        "driver": "e1000e"
      }
    ],
    "host": "10.0.101.61",
    "hostname": "localhost.localdomain",
    "vnics": [
      {
        "subnet_mask": "255.255.255.0",
        "dhcp": false,
        "mac_address": "00:23:7d:33:f0:6e",
        "device": "vmk0",
        "ip": "10.0.101.61",
        "mtu": 1500,
        "portgroup": "Management Network"
      }
    ],
    "vswitches": [
      {
        "pnics": ["vmnic0", "vmnic1"],
        "portgroups": ["Management Network", "VM Network"],
        "name": "vSwitch0",
        "mtu": 1500
      }
    ]
  },
  {
    "pnics": [
      {
        "device": "vmnic0",
        "mac": "00:23:7d:33:47:f8",
        "driver": "bnx2"
      },
      {
        "device": "vmnic1",
        "mac": "00:23:7d:33:47:ec",
        "driver": "bnx2"
      },
      {
        "device": "vmnic2",
        "mac": "e8:39:35:10:21:dd",
        "driver": "e1000e"
      },
      {
        "device": "vmnic3",
        "mac": "e8:39:35:10:21:dc",
        "driver": "e1000e"
      },
      {
        "device": "vmnic4",
        "mac": "e8:39:35:10:21:df",
        "driver": "e1000e"
      },
      {
        "device": "vmnic5",
        "mac": "e8:39:35:10:21:de",
        "driver": "e1000e"
      }
    ],
    "host": "10.0.101.62",
    "hostname": "localhost.localdomain",
    "vnics": [
      {
        "subnet_mask": "255.255.255.0",
        "dhcp": false,
        "mac_address": "00:23:7d:33:47:f8",
        "device": "vmk0",
        "ip": "10.0.101.62",
        "mtu": 1500,
        "portgroup": "Management Network"
      }
    ],
    "vswitches": [
      {
        "pnics": ["vmnic0", "vmnic1"],
        "portgroups": ["Management Network", "VM Network"],
        "name": "vSwitch0",
        "mtu": 1500
      }
    ]
  }
]
```

So there we have it. Will I use this? Who knows! But as I mentioned, I was not
able to find anything in regards to this so I figured I would put it out here.

Enjoy!
