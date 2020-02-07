---
title: "Manjaro Packer and VMware Workstation"
date: "2019-11-02 11:30:00"
categories:
  - Automation
tags:
  - Packer
---

```bash
==> vmware-iso: Connecting to VM via VNC (127.0.0.1:5901)
==> vmware-iso: Error detecting host IP: Could not find vmnetdhcp conf file:
==> vmware-iso: Stopping virtual machine...
==> vmware-iso: Deleting output directory...
Build 'vmware-iso' errored: Error detecting host IP: Could not find vmnetdhcp conf file:
```

```bash
sudo vmware-networks --stop
...
Stopped Bridged networking on vmnet0
Stopped DHCP service on vmnet1
Disabled hostonly virtual adapter on vmnet1
Stopped DHCP service on vmnet8
Stopped NAT service on vmnet8
Disabled hostonly virtual adapter on vmnet8
Stopped all configured services on all networks
```

```bash
sudo vi /etc/vmware/netmap.conf
```

```bash
network0.name = "Bridged"
network0.device = "vmnet0"
network1.name = "HostOnly"
network1.device = "vmnet1"
network2.name = "NAT"
network2.device = "vmnet8"
```

```bash
sudo vmware-networks --start
...
Started Bridge networking on vmnet0
Enabled hostonly virtual adapter on vmnet1
Started DHCP service on vmnet1
Started NAT service on vmnet8
Enabled hostonly virtual adapter on vmnet8
Started DHCP service on vmnet8
Started all configured services on all networks
```

```bash
sudo vmware-networks --status
...
Bridge networking on vmnet0 is running
DHCP service on vmnet1 is running
Hostonly virtual adapter on vmnet1 is enabled
DHCP service on vmnet8 is running
NAT service on vmnet8 is running
Hostonly virtual adapter on vmnet8 is enabled
All the services configured on all the networks are running
```

```bash
yay -S --noconfirm --needed vagrant-vmware-utility
```

```bash
cannot load such file -- vagrant/vmware/desktop
```
