---
title: "Ansible - IP Sets and DShield Block List"
date: "2018-01-17 16:51"
categories:
  - Automation
tags:
  - Ansible
---

In this post, we are going to look into how we can leverage [Ansible](https://www.ansible.com)
to manage a Linux firewall using [ipset](http://ipset.netfilter.org/) along with
a [DShield block list](https://www.dshield.org/block.txt). Now, why would we
want to do this you might be asking yourself. One good reason for this would be
to ensure that we do not allow any traffic inbound or outbound to any IP subnet
that shows up in the top 20 attacking class C (/24) subnets over the last three
days. But the overall reason is for the hell of it and it is fun!

> NOTE: You can checkout my Ansible role [ansible-ipset](https://github.com/mrlesmithjr/ansible-ipset) for additional usage.

## DShield Block List

As you may or may not already know the DShield block list comes as a text file.
So the first thing we need to do is convert this into a consumable format for
Ansible. In this case, we will be converting it to YAML.

### Example DShield Block List (TXT)

Below we have an example of what a DShield block list looks like in its native
TXT format.

`block.txt`:

```txt
#
#   DShield.org Recommended Block List
#    (c) 2018 DShield.org
#   some rights reserved. Details http://creativecommons.org/licenses/by-nc-sa/2.5/
#   use on your own risk. No warranties implied.
#   primary URL: http://feeds.dshield.org/block.txt
#     PGP Sign.: http://feeds.dshield.org/block.txt.asc
#
#   comments: info@dshield.org
#    updated: Wed Jan 17 22:00:15 2018 UTC
#
#    This list summarizes the top 20 attacking class C (/24) subnets
#   over the last three days. The number of 'attacks' indicates the
#   number of targets reporting scans from this subnet.
#
#
#    Columns (tab delimited):
#
#    (1) start of netblock
#    (2) end of netblock
#    (3) subnet (/24 for class C)
#    (4) number of targets scanned
#    (5) name of Network
#    (6) Country
#    (7) contact email address
#
#    If a range is assigned to multiple users, the first one is listed.
#
Start	End	Netmask	Attacks	Name	Country	email
77.72.82.0	77.72.82.255	24	9424	NETUP-AS ,	RU	aospan@netup.ru
191.101.167.0	191.101.167.255	24	6276	Digital Energy Technologies Chile SpA,	CL	noc@AS61440.NET
181.214.87.0	181.214.87.255	24	6193	WZCOM-US - WZ Communications Inc.,	US	abuse@webazilla.com
5.188.86.0	5.188.86.255	24	5959	PIN-AS ,	RU	abuse@pinspb.ru
77.72.85.0	77.72.85.255	24	4197	NETUP-AS ,	RU	aospan@netup.ru
5.188.203.0	5.188.203.255	24	3819	PIN-AS ,	RU	abuse@pinspb.ru
196.52.43.0	196.52.43.255	24	3647	LEASEWEB-NL Netherlands,	NL	abuse@nl.leaseweb.com
216.158.238.0	216.158.238.255	24	3394	NJIIX-AS-1 - NEW JERSEY INTERNATIONAL INTERNET EXCHANGE LLC,	US	gregg@njiix.net
5.188.10.0	5.188.10.255	24	3309	PIN-AS ,	RU	abuse@pinspb.ru
46.29.162.0	46.29.162.255	24	3186	ASBAXET ,	RU	noc@baxet.ru
109.248.9.0	109.248.9.255	24	3100	DGRID ,	EE	abuse@linxtelecom.net
80.82.70.0	80.82.70.255	24	2196	QUASINETWORKS ,	NL	abuse@quasinetworks.com
185.35.62.0	185.35.62.255	24	2144	KS-ASN1 This ASN is used for Internet security research. Internet-scale port scanning activities are launched from it. Don_t hesitate to contact portscan@nagra.com would you have any question.,	CH	abuse@nagra.com
93.174.93.0	93.174.93.255	24	1985	QUASINETWORKS ,	NL	abuse@quasinetworks.com
141.212.122.0	141.212.122.255	24	1948	UMICH-AS-5 - University of Michigan,	US	abuse@umich.edu
85.93.20.0	85.93.20.255	24	1887	ASGHOSTNET ,	DE	abuse@ghostnet.de
80.82.77.0	80.82.77.255	24	1754	QUASINETWORKS ,	NL	abuse@quasinetworks.com
5.188.11.0	5.188.11.255	24	1443	PIN-AS ,	RU	abuse@pinspb.ru
125.212.217.0	125.212.217.255	24	1360	VIETEL-AS-AP Viettel Corporation,	VN	hm-changed@vnnic.net.vn
104.236.178.0	104.236.178.255	24	1329	DIGITALOCEAN-ASN - Digital Ocean, Inc.,	US	abuse@digitalocean.com
```

### Example DShield Block List (YAML)

Below we have an example of what a DShield block list **COULD** look like in
`YAML` structure.

`block.yml`:

```yaml
---
dshield_block_list:
  - start: 77.72.82.0
    end: 77.72.82.255
    netmask: 24
    attacks: 8204
  - start: 191.101.167.0
    end: 191.101.167.255
    netmask: 24
    attacks: 4533
  - start: 181.214.87.0
    end: 181.214.87.255
    netmask: 24
    attacks: 4405
  - start: 5.188.86.0
    end: 5.188.86.255
    netmask: 24
    attacks: 4226
  - start: 77.72.85.0
    end: 77.72.85.255
    netmask: 24
    attacks: 3279
  - start: 46.29.162.0
    end: 46.29.162.255
    netmask: 24
    attacks: 2999
  - start: 109.248.9.0
    end: 109.248.9.255
    netmask: 24
    attacks: 2790
  - start: 196.52.43.0
    end: 196.52.43.255
    netmask: 24
    attacks: 2667
  - start: 5.188.203.0
    end: 5.188.203.255
    netmask: 24
    attacks: 2592
  - start: 5.188.10.0
    end: 5.188.10.255
    netmask: 24
    attacks: 2575
  - start: 216.158.238.0
    end: 216.158.238.255
    netmask: 24
    attacks: 2382
  - start: 80.82.70.0
    end: 80.82.70.255
    netmask: 24
    attacks: 2046
  - start: 185.35.62.0
    end: 185.35.62.255
    netmask: 24
    attacks: 1933
  - start: 85.93.20.0
    end: 85.93.20.255
    netmask: 24
    attacks: 1664
  - start: 141.212.122.0
    end: 141.212.122.255
    netmask: 24
    attacks: 1473
  - start: 93.174.93.0
    end: 93.174.93.255
    netmask: 24
    attacks: 1433
  - start: 80.82.77.0
    end: 80.82.77.255
    netmask: 24
    attacks: 1411
  - start: 104.236.178.0
    end: 104.236.178.255
    netmask: 24
    attacks: 1156
  - start: 180.97.106.0
    end: 180.97.106.255
    netmask: 24
    attacks: 1104
  - start: 5.188.11.0
    end: 5.188.11.255
    netmask: 24
    attacks: 1088
```

So as you can see from the above we can easily iterate over this structure using
Ansible to define the IP sets for our firewall.

### Example DShield Block List IP Set

Below is an example of what our ipset **COULD** look like once we have iterated
over our `block.yml` structure to generate our ipset rules.

```bash
Name: dshield_block_list
Type: hash:net
Revision: 6
Header: family inet hashsize 1024 maxelem 65536
Size in memory: 1600
References: 2
Members:
5.188.10.0/24
5.188.86.0/24
5.188.203.0/24
85.93.20.0/24
46.29.162.0/24
80.82.70.0/24
185.35.62.0/24
93.174.93.0/24
141.212.122.0/24
216.158.238.0/24
181.214.87.0/24
109.248.9.0/24
191.101.167.0/24
104.236.178.0/24
196.52.43.0/24
5.188.11.0/24
77.72.82.0/24
77.72.85.0/24
180.97.106.0/24
80.82.77.0/24
```

### Converting DShield Block List From TXT To YAML

So the above all looks great but how do we convert the native DShield `block.txt`
to `block.yml`? In order to do the conversion we will execute a few Ansible tasks
along with a `Jinja2` template to do our conversion.

So follow along as we break down each Ansible task below.

#### Default Ansible Variables

Below you will find the default defined Ansible variables in use for the following
tasks:

{% raw %}

```yaml
ipset_config_file: /etc/ipset/ipsets

ipset_dshield_block_list: "{{ lookup('file', ipset_dshield_block_file) }}"

ipset_dshield_block_file: /tmp/block.txt

ipset_dshield_block_file_yaml: /tmp/block.yml

ipset_enable_dshield_block_list: true

ipset_iptables_config_file: /etc/iptables/iptables

ipset_iptables_default_forward_policy: ACCEPT

ipset_iptables_default_input_policy: ACCEPT

ipset_iptables_default_output_policy: ACCEPT

ipset_iptables_dshield_chains:
  - INPUT
  - OUTPUT
```

{% endraw %}

#### Downloading DShield Block List

In this task we will be downloading the DShield Block list to our `localhost`. So
the following Ansible task will do just that for us:

{% raw %}

```yaml
- name: configure | Downloading DShield.org Block List
  get_url:
    url: http://feeds.dshield.org/block.txt
    dest: "{{ ipset_dshield_block_file }}"
  delegate_to: localhost
  when: ipset_enable_dshield_block_list
```

{% endraw %}

#### Generating YAML DShield Block List

In this task we will be using a `Jinja2` template to generate our `block.yml` file
which will also be performed on our `localhost`.

{% raw %}

```yaml
- name: configure | Generating DShield.org YAML Block List
  template:
    src: block.yml.j2
    dest: "{{ ipset_dshield_block_file_yaml }}"
  delegate_to: localhost
  when: ipset_enable_dshield_block_list
```

{% endraw %}

As you can see from the above task we are using a template called `block.yml.j2`.
So of course as you are wondering, what does this template look like. So that is
what you will find below. And as you can see, it is rather quite simple.

`block.yml.j2`:
{% raw %}

```yaml
---
dshield_block_list:
{% for item in ipset_dshield_block_list.split('\n') %}
{%   if '#' not in item %}
{%     set list = item.split('\t') %}
{%     if list[0]|ipaddr %}
  - start: {{ list[0] }}
    end: {{ list[1] }}
    netmask: {{ list[2] }}
    attacks: {{ list[3] }}
{%     endif %}
{%   endif %}
{% endfor %}
```

{% endraw %}

So based on the above template here is what is going on here. First off we are
iterating over the `ipset_dshield_block_list` which from our [default Ansible variables](#default-ansible-variables) this is defined as `{% raw %}"{{ lookup('file', ipset_dshield_block_file) }}"{% endraw %}`. So this is doing a file lookup using
the defined variable of `ipset_dshield_block_file`. And again, if we look at our
[default Ansible variables](#default-ansible-variables) this is defined as
`/tmp/block.txt`.

Now for the remaining bits of the template we are discarding any lines which
contain `#` because these are comments. Then we are splitting the lines which do
not contain `#` by tabs into our elements. And because our element `[0]` is an
IP address we are checking to ensure that is truly an IP address by `if list[0]|ipaddr`.
Then if that is true, we then define our variables which we are looking for. In
this case those would be `start`, `end`, `netmask`, and `attacks`.

#### Importing DShield Block List Variables

Now that our `block.yml` file has been generated. We can now import these variables
into Ansible to use for further consumption.

```yaml
- name: configure | Importing dshield_block_list Variables
  include_vars:
    file: "{{ ipset_dshield_block_file_yaml }}"
  when: ipset_enable_dshield_block_list
```

And once we do the above we now have a variable `dshield_block_list` which is
available for Ansible to iterate over. This variable is defined in our template
`block.yml.j2` which we used to generate our `block.yml` file.

### Generating IP Set Rules

Now that we have our DShield block list converted to `YAML` and imported as a
variable called `dshield_block_list` we are ready to generate our `ipset` rules.

In order to generate our `ipset` rules we will again use a `Jinja2` template to
do this.

`ipsets.j2`:
{% raw %}

```yaml
flush
{% if ipset_enable_dshield_block_list %}
create dshield_block_list hash:net maxelem 65536
{%   for _dshield_block_list in dshield_block_list %}
add dshield_block_list {{ _dshield_block_list['start'] }}-{{ _dshield_block_list['end'] }}
{%   endfor %}
{% endif %}
```

{% endraw %}

As you can see in the above template we are using the variable `dshield_block_list`
to iterate over our DShield block list.

And using the following Ansible task we will use the above template to generate
our ipsets rules.

{% raw %}

```yaml
- name: configure | Generating ipset Rules {{ ipset_config_file }}
  template:
    src: ipsets.j2
    dest: "{{ ipset_config_file }}"
  become: true
```

{% endraw %}

And the following is an example of what our ipsets rules **COULD** look like:

```bash
flush
create dshield_block_list hash:net maxelem 65536
add dshield_block_list 77.72.82.0-77.72.82.255
add dshield_block_list 191.101.167.0-191.101.167.255
add dshield_block_list 181.214.87.0-181.214.87.255
add dshield_block_list 5.188.86.0-5.188.86.255
add dshield_block_list 77.72.85.0-77.72.85.255
add dshield_block_list 46.29.162.0-46.29.162.255
add dshield_block_list 109.248.9.0-109.248.9.255
add dshield_block_list 196.52.43.0-196.52.43.255
add dshield_block_list 5.188.203.0-5.188.203.255
add dshield_block_list 5.188.10.0-5.188.10.255
add dshield_block_list 216.158.238.0-216.158.238.255
add dshield_block_list 80.82.70.0-80.82.70.255
add dshield_block_list 185.35.62.0-185.35.62.255
add dshield_block_list 85.93.20.0-85.93.20.255
add dshield_block_list 141.212.122.0-141.212.122.255
add dshield_block_list 93.174.93.0-93.174.93.255
add dshield_block_list 80.82.77.0-80.82.77.255
add dshield_block_list 104.236.178.0-104.236.178.255
add dshield_block_list 180.97.106.0-180.97.106.255
add dshield_block_list 5.188.11.0-5.188.11.255
```

### Generating IPTables Rules

Now that we have generated our `ipset` rules. We are now ready to generate our
`iptables` rules which will use our `ipset` rules we have generated.

We will again be using a `Jinja2` template to generate our `iptables` rules:

{% raw %}

```yaml
*filter
:INPUT {{ ipset_iptables_default_input_policy|upper }}
:FORWARD {{ ipset_iptables_default_forward_policy|upper }}
:OUTPUT {{ ipset_iptables_default_output_policy|upper }}
{% if ipset_enable_dshield_block_list and ipset_iptables_dshield_chains is defined %}
{%   for _chain in ipset_iptables_dshield_chains %}
{%     if _chain|upper == "INPUT" %}
-A {{ _chain|upper }} -m set --match-set dshield_block_list src -j DROP
{%     elif _chain|upper == "FORWARD" %}
-A {{ _chain|upper }} -m set --match-set dshield_block_list src -j DROP
-A {{ _chain|upper }} -m set --match-set dshield_block_list dst -j DROP
{%     elif _chain|upper == "OUTPUT" %}
-A {{ _chain|upper }} -m set --match-set dshield_block_list dst -j DROP
{%     endif %}
{%   endfor %}
{% endif %}
COMMIT
```

{% endraw %}

And using the following Ansible task we can generate our `iptables` rules using
the above `Jinja2` template:

{% raw %}

```yaml
- name: configure | Generating IPTables Rules {{ ipset_iptables_config_file }}
  template:
    src: iptables.j2
    dest: "{{ ipset_iptables_config_file }}"
  become: true
  register: _ipset_iptables_rules_generated
```

{% endraw %}

And the following is an example of what our `iptables` rules **COULD** look like:

```bash
*filter
:INPUT ACCEPT
:FORWARD ACCEPT
:OUTPUT ACCEPT
-A INPUT -m set --match-set dshield_block_list src -j DROP
-A OUTPUT -m set --match-set dshield_block_list dst -j DROP
COMMIT
```

### Stitching It All Together

Now we are ready to stitch together our `ipset` rules and `iptables` rules.
Because our `iptables` rules rely on our `ipset` rules we must first load our
`ipset` rules. But first we **SHOULD** clear our existing `iptables` and `ipset`
rules.

#### Clearing IPTables Rules and IP Set Rules

We will use the following Ansible task to clear our `iptables` and `ipset` rules:

```yaml
- name: configure | Clearing IPTables Rules and ipset Rules
  shell: >
         iptables -P INPUT ACCEPT &&
         iptables -P FORWARD ACCEPT &&
         iptables -P OUTPUT ACCEPT &&
         iptables -F &&
         iptables -X &&
         ipset destroy
  become: true
```

The above will do the following:

1.  Reset default INPUT policy to ACCEPT
2.  Reset default FORWARD policy to ACCEPT
3.  Reset default OUTPUT policy to ACCEPT
4.  Flush (delete) all iptables rules
5.  Delete all iptables user chains
6.  Destroy all ipset rules

#### Importing IP Set Rules

Now we are ready to import(restore) our generated `ipset` rules. And we will
accomplish this with the following Ansible task:

{% raw %}

```yaml
- name: configure | Restoring ipset Rules {{ ipset_config_file }}
  shell: ipset restore -! < {{ ipset_config_file }}
  become: true
```

{% endraw %}

#### Importing IPTables Rules

We are now finally ready to import(restore) our generated `iptables` rules. Which
we will accomplish by using the following Ansible task:

{% raw %}

```yaml
- name: configure | Restoring IPTables Rules {{ ipset_iptables_config_file }}
  command: iptables-restore {{ ipset_iptables_config_file }}
  become: true
```

{% endraw %}

#### Validating IP Set Rules and IPTables Rules

Now for the final step we can now validate that our `ipset` rules and `iptables`
rules are in place now.

```bash
vagrant@node0:~$ sudo ipset list dshield_block_list
Name: dshield_block_list
Type: hash:net
Revision: 6
Header: family inet hashsize 1024 maxelem 65536
Size in memory: 1600
References: 2
Members:
5.188.10.0/24
5.188.86.0/24
5.188.203.0/24
85.93.20.0/24
46.29.162.0/24
80.82.70.0/24
185.35.62.0/24
93.174.93.0/24
141.212.122.0/24
216.158.238.0/24
181.214.87.0/24
109.248.9.0/24
191.101.167.0/24
104.236.178.0/24
196.52.43.0/24
5.188.11.0/24
77.72.82.0/24
77.72.85.0/24
180.97.106.0/24
80.82.77.0/24
```

```bash
vagrant@node0:~$ sudo iptables -L -v -n
Chain INPUT (policy ACCEPT 163 packets, 107K bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            match-set dshield_block_list src

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 130 packets, 13305 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            match-set dshield_block_list dst
```

### Persistence Across Reboots

And now for the final piece of this. How do we persist our `ipset` rules and
`iptables` rules across reboots. Ah, the million dollar question. Will all of
this still exist after a reboot. Well, of course, and we can do that by yet
again another `Jinja2` template and an Ansible task.

> NOTE: The below example is for `Debian` based systems **ONLY**.

Our `Jinja2` template **COULD** look like below:

{% raw %}

```yaml
#! /usr/bin/env bash

{{ ansible_managed|comment }}

/sbin/ipset restore -! < {{ ipset_config_file }}
/sbin/iptables-restore {{ ipset_iptables_config_file }}
```

{% endraw %}

And our Ansible task would look like below:

{% raw %}

```yaml
- name: configure | Creating Firewall Load On Reboot Script
  template:
    src: etc/network/if-pre-up.d/firewall.j2
    dest: /etc/network/if-pre-up.d/firewall
    owner: root
    group: root
    mode: "u=rwx,g=rwx,o="
  become: true
  when: ansible_os_family == "Debian"
```

{% endraw %}

And there you have it, our `ipset` rules and `iptables` rules will persist across
reboots.

And this concludes this post on doing some fun stuff with our firewall!

Enjoy!
