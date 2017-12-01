---
title: "Ansible - Defining Variables As Dictionaries"
date: "2017-11-30 22:25"
categories:
  - Automation
tags:
  - Ansible
---

This post is to go through an example of defining [Ansible](https://www.ansible.com)
variables as [dictionaries](http://docs.ansible.com/ansible/latest/YAMLSyntax.html#yaml-basics)
rather than lists. I wanted to throw this together as I have not found a decent
example of doing this.

> NOTE: Dictionaries are composed of `key: value` pairs.

When using Ansible you will generally find examples which use lists for variable
definitions rather than dictionaries. So below I will show an example of a list
and then compare it to a dictionary.

In this example I will be defining what would end up becoming the `/etc/network/interfaces`
file on a `Debian` based system. The example of the list definitions will be a
short excerpt compared to the example of using dictionaries.

## Lists

When using lists all definitions begin with a `"- "`:

> NOTE: Notice the space after the `-`.

```yaml
network_interfaces:
  - name: enp0s3
    configure: true
    method: dhcp
    parameters:
      - param: pre-up sleep
        val: 2
  - name: enp0s8
    configure: true
    method: static
    address: 192.168.250.10
    netmask: 255.255.255.0
  - name: enp0s9
    configure: true
    comment: bond0 member
    method: manual
    parameters:
      - param: bond_master
        val: bond0
  - name: enp0s10
    configure: true
    comment: bond0 member
    method: manual
    parameters:
      - param: bond_master
        val: bond0
  - name: enp0s16
    configure: true
    comment: br0 member
    method: manual
```

As you can see from the above definitions this tends to be the traditional way
of defining variables in Ansible. (Even for me!!)

## Dictionaries

When using dictionaries the defintions are in a simple `key: value` pair.

> NOTE: Notice the space after the `:`.

Below is an example of the same definitions in our list example but changed to
be used as dictionaries.

```yaml
network_interfaces:
  enp0s3:
    configure: true
    method: dhcp
    parameters:
      pre-up: sleep 2
  enp0s8:
    address: 10.0.0.100
    configure: true
    gateway: 10.0.0.1
    method: static
    netmask: 255.255.255.0
  enp0s09:
    configure: true
    method: manual
    parameters:
      bond_master: bond0
  enp0s10:
    configure: true
    method: manual
    parameters:
      bond_master: bond0
  enp0s16:
    configure: true
    method: manual
  enp0s17:
    configure: true
    method: manual
    parameters:
      pre-up: ifconfig $IFACE up
      post-down: ifconfig $IFACE down
  lo:
    configure: true
    method: loopback
```

As you can see, the format is extremely different but tends to be a bit cleaner
looking in my opinion.

## Usage

So how could we use dictionaries along with a `Jinja2` template to generate our
`/etc/network/interfaces` file? Let's look at the example below.

### Definitions

Let's first define a more elaborate set of variables using dictionaries.

```yaml
---
network_bonds:
  bond0:
    address: 192.168.1.10
    comment: Bond Group 0
    configure: true
    method: static
    netmask: 255.255.255.0
    parameters:
      bond_mode: active-backup
      bond_miimon: 100
      primary: enp0s9
      slaves:
        - enp0s9
        - enp0s10

network_bridges:
  br0:
    address: 192.168.1.11
    comment: Bridge 0
    configure: true
    method: static
    netmask: 255.255.255.0
    gateway: 192.168.1.1
    parameters:
     bridge_stp: off
     bridge_fd: 0
     ports:
       - enp0s16

network_interfaces:
  enp0s3:
    configure: true
    method: dhcp
    parameters:
      pre-up: sleep 2
  enp0s8:
    address: 10.0.0.100
    configure: true
    gateway: 10.0.0.1
    method: static
    netmask: 255.255.255.0
  enp0s09:
    configure: true
    method: manual
    parameters:
      bond_master: bond0
  enp0s10:
    configure: true
    method: manual
    parameters:
      bond_master: bond0
  enp0s16:
    configure: true
    method: manual
  enp0s17:
    configure: true
    method: manual
    parameters:
      pre-up: ifconfig $IFACE up
      post-down: ifconfig $IFACE down
  lo:
    configure: true
    method: loopback
```

We have now defined some interfaces, bonds, and bridges.

### Template

Let's now look at an example `Jinja2` template on how to use our defintions to
generate our `/etc/network/interfaces` file.

> NOTE: Pay close attention to some of the `Jinja2` voodoo going on here. There
> are a few interesting tricks going on.

{% raw %}

```yaml
###
# Beginning of network interfaces
###

{% for int in network_interfaces %}
{%   set _int = network_interfaces[int] %}
{%   if _int['configure'] %}
{%     if _int['comment'] is defined %}
# {{ _int['comment'] }}
{%     endif %}
auto {{ int }}
iface {{ int }} inet {{ _int['method'] }}
{%     if _int['method']|lower == "static" or _int['method']|lower == "manual" %}
{%       if _int['address'] is defined %}
  address {{ _int['address'] }}
{%       endif %}
{%       if _int['netmask'] is defined %}
  netmask {{ _int['netmask'] }}
{%       endif %}
{%       if _int['gateway'] is defined %}
  gateway {{ _int['gateway'] }}
{%       endif %}
{%     endif %}
{%   endif %}
{%   if _int['parameters'] is defined %}
{%     for k, v in _int['parameters'].iteritems() %}
  {{ k }} {% if v is not iterable or v is string %}{{ v }}{% elif v is iterable and v is not string %}{{ v|join(" ") }}{% endif %}

{%     endfor %}
{%   endif %}

{% endfor %}
###
# End of network interfaces
###

###
# Beginning of network bonds
###

{% for bond in network_bonds %}
{%   set _bond = network_bonds[bond] %}
{%   if _bond['configure'] %}
{%     if _bond['comment'] is defined %}
# {{ _bond['comment'] }}
{%     endif %}
auto {{ bond }}
iface {{ bond }} inet {{ _bond['method'] }}
{%     if _bond['method']|lower == "static" or _bond['method']|lower == "manual" %}
{%       if _bond['address'] is defined %}
  address {{ _bond['address'] }}
{%       endif %}
{%       if _bond['netmask'] is defined %}
  netmask {{ _bond['netmask'] }}
{%       endif %}
{%       if _bond['gateway'] is defined %}
  gateway {{ _bond['gateway'] }}
{%       endif %}
{%     endif %}
{%     if _bond['parameters'] is defined %}
{%       for k, v in _bond['parameters'].iteritems() %}
  {{ k }} {% if v is not iterable or v is string %}{{ v }}{% elif v is iterable and v is not string %}{{ v|join(" ") }}{% endif %}

{%       endfor %}
{%     endif %}
{%   endif %}

{% endfor %}

###
# Beginning of network bonds
###

{% for bridge in network_bridges %}
{%   set _bridge = network_bridges[bridge] %}
{%   if _bridge['configure'] %}
{%     if _bridge['comment'] is defined %}
# {{ _bridge['comment'] }}
{%     endif %}
auto {{ bridge }}
iface {{ bridge }} inet {{ _bridge['method'] }}
{%     if _bridge['method']|lower == "static" or _bridge['method']|lower == "manual" %}
{%       if _bridge['address'] is defined %}
  address {{ _bridge['address'] }}
{%       endif %}
{%       if _bridge['netmask'] is defined %}
  netmask {{ _bridge['netmask'] }}
{%       endif %}
{%       if _bridge['gateway'] is defined %}
  gateway {{ _bridge['gateway'] }}
{%       endif %}
{%     endif %}
{%     if _bridge['parameters'] is defined %}
{%       for k, v in _bridge['parameters'].iteritems() %}
  {{ k }} {% if v is not iterable or v is string %}{{ v }}{% elif v is iterable and v is not string %}{{ v|join(" ") }}{% endif %}

{%       endfor %}
{%     endif %}
{%   endif %}

{% endfor %}
###
# End of network bonds
###
```

{% endraw %}

### Final Configuration

Now that we have our dictionaries defined and our Jinja2 template defined, we can
see what our final configuration would look like below:

```bash
###
# Beginning of network interfaces
###

auto enp0s3
iface enp0s3 inet dhcp
  pre-up sleep 2

auto enp0s8
iface enp0s8 inet static
  address 10.0.0.100
  netmask 255.255.255.0
  gateway 10.0.0.1

auto enp0s09
iface enp0s09 inet manual
  bond_master bond0

auto enp0s10
iface enp0s10 inet manual
  bond_master bond0

auto enp0s16
iface enp0s16 inet manual

auto enp0s17
iface enp0s17 inet manual
  pre-up ifconfig $IFACE up
  post-down ifconfig $IFACE down

auto lo
iface lo inet loopback

###
# End of network interfaces
###

###
# Beginning of network bonds
###

# Bond Group 0
auto bond0
iface bond0 inet static
  address 192.168.1.10
  netmask 255.255.255.0
  bond_mode active-backup
  bond_miimon 100
  primary enp0s9
  slaves enp0s9 enp0s10


###
# Beginning of network bonds
###

# Bridge 0
auto br0
iface br0 inet static
  address 192.168.1.11
  netmask 255.255.255.0
  gateway 192.168.1.1
  bridge_stp off
  bridge_fd 0
  ports enp0s16

###
# End of network bonds
###
```

So there you have it, we now have a very cleanly defined `/etc/network/interfaces`
file by using Ansible variables defined as dictionaries.

Enjoy!
