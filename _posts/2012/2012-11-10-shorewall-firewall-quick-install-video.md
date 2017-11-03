---
  title: Shorewall firewall quick install video
  date: 2012-11-10 19:28:17
---

Installing shorewall firewall on Ubuntu 12.04 for a simple single
network configuration.

```bash
sudo nano /etc/network/interfaces
```

assumption is that eth0 is your internet facing interface using dhcp\
add the following

```bash
iface auto eth1
address 192.168.2.2
netmask 255.255.255.0

sudo apt-get install shorewall
sudo nano /etc/default/shorewall
```

change _startup=0_ to _startup=1_

```bash
sudo nano /etc/shorewall/shorewall.conf
```

change _STARTUP_ENABLED=No_ to _STARTUP_ENABLED=Yes_

```bash
sudo cp /usr/share/doc/shorewall/default-config/* /etc/shorewall/
sudo nano /etc/shorewall/zones

fw firewall
net ipv4
loc ipv4

sudo nano /etc/shorewall/interfaces
net eth0 detect dhcp,routefilter,norfc1918,logmartians,nosmurfs,tcpflags
loc eth1 detect tcpflags
sudo nano /etc/shorewall/policy
net all DROP info
loc all ACCEPT
fw all ACCEPT
# Last Policy rule. Must be last
all all REJECT info

sudo nano /etc/rules
```

\*\* There are no firewall rules in this as the default rule for the
local network is to accept all outgoing traffic.\*\*

```bash
sudo nano /etc/shorewall/masq
eth0 eth1

sudo touch /var/log/messages
```

<https://youtu.be/ZbjKMJQq6Z0>
