---
  title: Hey, I can DevOPS my Network too! -- Auto-configured OSPF (Part 4)
  categories:
    - Automation
  tags:
    - Ansible
  redirect_from:
    - /hey-i-can-devops-my-network-too-auto-configured-ospf-part-4
---

In the last
[post](https://everythingshouldbevirtual.com/hey-i-can-devops-my-network-too-vagrant-up-part-3)
we spun up (Vagrant Up) our environment. So at this point we are ready to start
exploring and seeing how easy it is to bring up OSPF auto-configured by just
configuring a few parameters within our Ansible settings.

So if you are not already connected to router1 (r1) over ssh let's do so now.

```bash
vagrant ssh r1
```

Now change to the vagrant synced folder that we went over earlier in
this series. It is here where we will be manipulating and running our
Ansible tasks from.

```bash
cd /vagrant
```

In this post we will be running the following Ansible playbook.
playbook.yml

```yaml
---
- hosts: quagga-routers
  remote_user: vagrant
  sudo: true
  vars:
  roles:
    - mrlesmithjr.quagga
  tasks:
    - name: installing packages
      apt: name={{ item }} state=present
      with_items:
        - traceroute
```

All that this playbook does is install the _mrlesmithjr.quagga_ role and
installs traceroute. The _mrlesmithjr.quagga_ role was installed from
Ansible Galaxy during our _bootstrap.yml_ play and is installed to
/etc/ansible/roles/, we will be exploring this role here soon.

```bash
ls -l /etc/ansible/roles
....
total 12
drwxr-xr-x 9 root root 4096 Sep  8 15:29 mrlesmithjr.base
drwxr-xr-x 8 root root 4096 Sep  8 15:29 mrlesmithjr.bootstrap
drwxr-xr-x 8 root root 4096 Sep  8 15:29 mrlesmithjr.quagga
```

And being that this role is what we will be focusing on throughout this
series as Quagga is what we will be using for all of our routing
capabilities. I will show you the default variables which are included
with this role and then how we will be using our
`group_vars/quagga-routers` variables to manipulate this role and allow
us to change up our environment based on our needs.
So let's look at the default variables which are included in the
mrlesmithjr.quagga role.

```bash
    cat /etc/ansible/roles/mrlesmithjr.quagga/defaults/main.yml
    ....
```

```yaml
---
# defaults file for ansible-quagga
config_glusterfs: false
config_interfaces: false
config_keepalived: false
config_quagga: false
keepalived_scripts:
  - backup_quagga.sh
  - fault_quagga.sh
  - master_quagga.sh
  - primary-backup.sh
keepalived_scripts_home: /opt/scripts
net_config_dir: /etc/network/interfaces.d
#quagga_bgp_router_configs:
#  - name: r1
#    local_as: 123
#    router_id: 1.1.1.1
#    neighbors:
#      - neighbor: 192.168.12.12
#        remote_as: 123
#      - neighbor: 192.168.31.13
#        remote_as: 123
#      - neighbor: 192.168.14.14
#        remote_as: 141
#      - neighbor: 192.168.15.15
#        remote_as: 151
#    network_advertisements:  #networks to advertise and/or define redistribute options
#      - 1.1.1.0/24
#      - 192.168.12.0/24
#      - 192.168.14.0/24
#      - 192.168.15.0/24
#    redistribute:
#      - connected
#      - isis
#      - kernel
#      - rip
#      - static
#  - name: r2
#    local_as: 123
#    router_id: 2.2.2.2
#    neighbors:
#      - neighbor: 192.168.12.11
#        remote_as: 123
#      - neighbor: 192.168.23.13
#        remote_as: 123
#    network_advertisements:  #networks to advertise and/or define redistribute options
#      - 2.2.2.0/24
#      - 192.168.12.0/24
#      - 192.168.23.0/24
#    redistribute:
#      - connected
#      - isis
#      - kernel
#      - rip
#      - static
quagga_bgp_redistribute:
  - connected
  - kernel
#  - static
#  - isis
#  - rip
quagga_config_bgpd: false #defines if quagga bgpd should be configured based on quagga_bgp_router_configs...makes it easy to disable auto routing in order to define your routes manually
quagga_config_ospfd: false  #defines if quagga ospfd should be configured based on quagga_ospf_ vars...makes it easy to disable auto routing in order to define your routes manually
quagga_configs:
  - daemons
  - debian.conf
  - vtysh.conf
  - zebra.conf
quagga_enable_bgpd: false
quagga_enable_ospfd: false
quagga_enable_password: quagga #define here or in group_vars/group
#quagga_interfaces_lo:
#  - int: lo
#    method: loopback
#    ip_address: 192.168.70.240/32
#  - int: lo
#    method: loopback
#    ip_address: 192.168.70.241/32
quagga_ospf_area: 51  #defines the desired area mapping for OSPF routing with upstream OSPF routers...define here or in group_vars/group
quagga_ospf_area_config:
  - network: '{{ quagga_ospf_routerid }}/24'
    area: '{{ quagga_ospf_area }}'
quagga_ospf_redistribute:
  - connected
#  - kernel
#  - static
#  - isis
#  - rip
quagga_ospf_routerid: '{{ ansible_default_ipv4.address }}'  #defines the router id IP address for OSPF...define here or in group_vars/group
quagga_password: quagga #define in group_vars/all/accounts
quagga_root_dir: /etc/quagga
sysctl_network_settings:
  - name: net.ipv4.ip_forward
    value: 1
  - name: net.ipv4.conf.all.forwarding
    value: 1
  - name: net.ipv4.conf.default.forwarding
    value: 1
  - name: net.ipv4.tcp_tw_reuse
    value: 1
  - name: net.ipv4.ip_local_port_range
    value: "1024 65023"
  - name: net.ipv4.tcp_max_syn_backlog
    value: 40000
  - name: net.ipv4.tcp_max_tw_buckets
    value: 400000
  - name: net.ipv4.tcp_max_orphans
    value: 60000
  - name: net.ipv4.tcp_syncookies
    value: 1
  - name: net.ipv4.tcp_synack_retries
    value: 3
  - name: net.core.somaxconn
    value: 40000
  - name: net.ipv4.tcp_fin_timeout
    value: 5
```

One thing to note in the above is that like most other configuration
files or scripts the '#' at the beginning of lines represents a
commented out line. There is a lot going on in this file and many
variables are for additional roles outside of the scope of this series
but I will explain a bit. To the right of many of the variables are
notes that are included but not all of them (I need to change that). So
I will explain a few of these without comments below. Mainly the ones
that we will not be manipulating during this series.

-   config_glusterfs: This defines if we are deploying onto a
    [GlusterFS](http://www.gluster.org/) backed environment. This would
    allow us to sync our Quagga configurations between nodes if changes
    were made post Ansible configurations.
-   config_interfaces: This defines if we want to configure all of our
    interfaces manually and create individual configuration files per
    interface.
-   config_keepalived: This defines if we would like to configure
    [KeepAliveD](http://www.keepalived.org/) for VIP configuration(s).

Now, I have also included as stated above included a
`group_vars/quagga-routers` variables file which was included in the
GitHub repo that you have forked. This is meant as a starting point to
adjusting configurations without having to modify the defaults file
included in the mrlesmithjr.quagga role. So we will now look at this
file and feel free to compare the differences but we will cover the
majority of these as the series continues when they are relevant to the
topic.
Assuming you still have an ssh session to router1 (r1) do the following.
If you do not, establish that session at this time (reference further up
in this post).

```bash
cd /vagrant
cat group_vars/quagga-routers
....
```

```yaml
---
config_quagga: false
quagga_bgp_router_configs:
  - name: r1
    local_as: 123
    router_id: 1.1.1.1
    neighbors:
      - neighbor: 192.168.12.12
        remote_as: 123
      - neighbor: 192.168.31.13
        remote_as: 123
      - neighbor: 192.168.14.14
        remote_as: 141
      - neighbor: 192.168.15.15
        remote_as: 151
#    network_advertisements:  #networks to advertise and/or define redistribute options
#      - 1.1.1.0/24
#      - 192.168.12.0/24
#      - 192.168.14.0/24
#      - 192.168.15.0/24
    redistribute:
      - connected
#      - isis
      - kernel
#      - rip
#      - static
  - name: r2
    local_as: 123
    router_id: 2.2.2.2
    neighbors:
      - neighbor: 192.168.12.11
        remote_as: 123
      - neighbor: 192.168.23.13
        remote_as: 123
#    network_advertisements:  #networks to advertise and/or define redistribute options
#      - 2.2.2.0/24
#      - 192.168.12.0/24
#      - 192.168.23.0/24
    redistribute:
      - connected
#      - isis
      - kernel
#      - rip
#      - static
  - name: r3
    local_as: 123
    router_id: 3.3.3.3
    neighbors:
      - neighbor: 192.168.23.12
        remote_as: 123
      - neighbor: 192.168.31.11
        remote_as: 123
#    network_advertisements:  #networks to advertise and/or define redistribute options
#      - 3.3.3.0/24
#      - 192.168.23.0/24
#      - 192.168.31.0/24
    redistribute:
      - connected
#      - isis
      - kernel
#      - rip
#      - static
  - name: r4
    local_as: 141
    router_id: 4.4.4.4
    neighbors:
      - neighbor: 192.168.14.11
        remote_as: 123
#    network_advertisements:  #networks to advertise and/or define redistribute options
#      - 4.4.4.0/24
#      - 192.168.14.0/24
#      - 192.168.41.0/24
    redistribute:
      - connected
#      - isis
      - kernel
#      - rip
#      - static
#quagga_bgp_redistribute:
#  - connected
#  - kernel
#  - static
#  - isis
#  - rip
quagga_config_bgpd: false #defines if quagga bgpd should be configured based on quagga_bgp_router_configs...makes it easy to disable auto routing in order to define your routes manually
quagga_config_ospfd: false  #defines if quagga ospfd should be configured based on quagga_ospf_ vars...makes it easy to disable auto routing in order to define your routes manually
quagga_enable_bgpd: false
quagga_enable_ospfd: false
quagga_enable_password: quagga
quagga_ospf_routerid: '{{ ansible_eth1.ipv4.address }}'
quagga_password: quagga
```

As you can see not all of the same variables are included here, and that
is because we will not need to make adjustments to those default
settings unless necessary. If we had a need to then we could simply add
the relevant variable(s) to this file and adjust accordingly. The
relevance of this file is the
[hierarchy](http://docs.ansible.com/ansible/playbooks_best_practices.html)
of Ansible variables. This file takes precedence over the defaults
included within the role itself. So some other things you will notice is
that we have a variable _quagga_bgp_router_configs_ with many
different levels but relevant to each router that we built out using
Vagrant (with the exception of router5 (r5) - which we will be adding
the config for when we get to the BGP post). The reason I point this out
is because that in our defaults we only have router1 (r1) defined but
the variable itself including the router definitions are all commented
out but in our `group_vars/quagga-routers` file we do not which shows how
we can manipulate a default variable and adjust per environment our
specific needs.

Now that we have covered some of the aspects of how Ansible variables
are defined and can be manipulated based on hierarchy we are now ready
to proceed with configuring our environment auto-configured for OSPF.

Again, assuming you have an ssh session with router1 (r1) still we will
execute our Ansible _playbook.yml_ play.

```bash
cd /vagrant
ansible-playbook -i hosts playbook.yml
....
PLAY [quagga-routers] *********************************************************
skipping: no hosts matched

PLAY RECAP ********************************************************************
```

Hey that was quick, but wait....nothing actually happened. Why is that?
Well the answer is in the message which states that no hosts matched. So
if you look at the playbook we just ran you will see that hosts:
quagga-routers is defined.

```yaml
---
- hosts: quagga-routers
  remote_user: vagrant
  sudo: true
  vars:
  roles:
    - mrlesmithjr.quagga
  tasks:
    - name: installing packages
      apt: name={{ item }} state=present
      with_items:
        - traceroute
```

So how do we solve this? Quite simply and all that we need to do is modify our
hosts file. But first let's look at it.

```bash
cat hosts
....
# Generated by Vagrant

r1 ansible_ssh_host=127.0.0.1 ansible_ssh_port=2222 ansible_ssh_private_key_file=/Users/larrysmith/projects/vagrant-ansible-routing-template/.vagrant/machines/r1/virtualbox/private_key
r2 ansible_ssh_host=127.0.0.1 ansible_ssh_port=2200 ansible_ssh_private_key_file=/Users/larrysmith/projects/vagrant-ansible-routing-template/.vagrant/machines/r2/virtualbox/private_key
r3 ansible_ssh_host=127.0.0.1 ansible_ssh_port=2201 ansible_ssh_private_key_file=/Users/larrysmith/projects/vagrant-ansible-routing-template/.vagrant/machines/r3/virtualbox/private_key
r4 ansible_ssh_host=127.0.0.1 ansible_ssh_port=2202 ansible_ssh_private_key_file=/Users/larrysmith/projects/vagrant-ansible-routing-template/.vagrant/machines/r4/virtualbox/private_key
r5 ansible_ssh_host=127.0.0.1 ansible_ssh_port=2203 ansible_ssh_private_key_file=/Users/larrysmith/projects/vagrant-ansible-routing-template/.vagrant/machines/r5/virtualbox/private_key
```

From the above we see that all of our nodes are defined but our group
quagga-routers is missing. We will now define this group and you will
see how defining this group can define what hosts a specific Ansible
play runs against. I should also note..that we could simply change
hosts: quagga-routers to hosts: all and then our play would work but
that is not what we want in this series.
So let's now add our quagga-routers group to the end of the file.

```bash
nano hosts
....
[quagga-routers]
r1
r2
r3
r4
r5
```

And now save the file. There is also a shortcut way of defining the
nodes which are part of a group and we could simply do the following
instead of adding each node individually.

```bash
[quagga-routers]
r[1:5]
```

Now your hosts file should look like the following.

```bash
# Generated by Vagrant

r1 ansible_ssh_host=127.0.0.1 ansible_ssh_port=2222 ansible_ssh_private_key_file=/Users/larrysmith/projects/vagrant-ansible-routing-template/.vagrant/machines/r1/virtualbox/private_key
r2 ansible_ssh_host=127.0.0.1 ansible_ssh_port=2200 ansible_ssh_private_key_file=/Users/larrysmith/projects/vagrant-ansible-routing-template/.vagrant/machines/r2/virtualbox/private_key
r3 ansible_ssh_host=127.0.0.1 ansible_ssh_port=2201 ansible_ssh_private_key_file=/Users/larrysmith/projects/vagrant-ansible-routing-template/.vagrant/machines/r3/virtualbox/private_key
r4 ansible_ssh_host=127.0.0.1 ansible_ssh_port=2202 ansible_ssh_private_key_file=/Users/larrysmith/projects/vagrant-ansible-routing-template/.vagrant/machines/r4/virtualbox/private_key
r5 ansible_ssh_host=127.0.0.1 ansible_ssh_port=2203 ansible_ssh_private_key_file=/Users/larrysmith/projects/vagrant-ansible-routing-template/.vagrant/machines/r5/virtualbox/private_key

[quagga-routers]
r[1:5]
```

With this now being added we are now ready to run our playbook once
again.

```raw
ansible-playbook -i hosts playbook.yml
....
PLAY [quagga-routers] *********************************************************

GATHERING FACTS ***************************************************************
ok: [r2]
ok: [r3]
ok: [r5]
ok: [r4]
ok: [r1]

TASK: [mrlesmithjr.quagga | debian | installing quagga pre-reqs] **************
changed: [r2] => (item=vlan)
changed: [r1] => (item=vlan)
changed: [r4] => (item=vlan)
changed: [r5] => (item=vlan)
changed: [r3] => (item=vlan)

TASK: [mrlesmithjr.quagga | debian | installing quagga] ***********************
changed: [r1]
changed: [r5]
changed: [r4]
changed: [r2]
changed: [r3]

TASK: [mrlesmithjr.quagga | debian | enabling quagga] *************************
changed: [r1]
changed: [r3]
changed: [r5]
changed: [r2]
changed: [r4]

TASK: [mrlesmithjr.quagga | configuring interfaces and vlans with dhcp mgmt] ***
skipping: [r1] => (item=vlan_config)
skipping: [r2] => (item=vlan_config)
skipping: [r3] => (item=vlan_config)
skipping: [r4] => (item=vlan_config)
skipping: [r5] => (item=vlan_config)

TASK: [mrlesmithjr.quagga | configuring interfaces and vlans with static mgmt] ***
skipping: [r2] => (item=vlan_config)
skipping: [r1] => (item=vlan_config)
skipping: [r3] => (item=vlan_config)
skipping: [r4] => (item=vlan_config)
skipping: [r5] => (item=vlan_config)

TASK: [mrlesmithjr.quagga | Create the directory for interface cfg files] *****
skipping: [r3]
skipping: [r2]
skipping: [r1]
skipping: [r4]
skipping: [r5]

TASK: [mrlesmithjr.quagga | Create the network configuration file for ethernet devices] ***
skipping: [r3] => (item=vlan_config)
skipping: [r1] => (item=vlan_config)
skipping: [r2] => (item=vlan_config)
skipping: [r4] => (item=vlan_config)
skipping: [r5] => (item=vlan_config)

TASK: [mrlesmithjr.quagga | reinitializing interfaces] ************************
skipping: [r1] => (item={u'skipped': True, u'changed': False})
skipping: [r2] => (item={u'skipped': True, u'changed': False})
skipping: [r3] => (item={u'skipped': True, u'changed': False})
skipping: [r5] => (item={u'skipped': True, u'changed': False})
skipping: [r4] => (item={u'skipped': True, u'changed': False})

TASK: [mrlesmithjr.quagga | config_glusterfs | checking to see if /etc/quagga has already been moved] ***
skipping: [r1]
skipping: [r3]
skipping: [r2]
skipping: [r4]
skipping: [r5]

TASK: [mrlesmithjr.quagga | config_glusterfs | moving existing /etc/quagga] ***
skipping: [r2]
skipping: [r1]
skipping: [r3]
skipping: [r4]
skipping: [r5]

TASK: [mrlesmithjr.quagga | config_glusterfs | checking again if /etc/quagga has already been moved] ***
skipping: [r1]
skipping: [r3]
skipping: [r4]
skipping: [r2]
skipping: [r5]

TASK: [mrlesmithjr.quagga | config_glusterfs | touching file in quagga_backup_dir] ***
skipping: [r1]
skipping: [r2]
skipping: [r3]
skipping: [r4]
skipping: [r5]

TASK: [mrlesmithjr.quagga | config_glusterfs | mounting gluster volumes - quagga] ***
skipping: [r1] => (item={'mountpoint': u'{# quagga_home #}', 'src': u'{# primary_gfs_server #}:/{# quagga_mnt #}', 'options': u'defaults,_netdev,backupvolfile-server={# secondary_gfs_server #}', 'fstype': 'glusterfs'})
skipping: [r2] => (item={'mountpoint': u'{# quagga_home #}', 'src': u'{# primary_gfs_server #}:/{# quagga_mnt #}', 'options': u'defaults,_netdev,backupvolfile-server={# secondary_gfs_server #}', 'fstype': 'glusterfs'})
skipping: [r3] => (item={'mountpoint': u'{# quagga_home #}', 'src': u'{# primary_gfs_server #}:/{# quagga_mnt #}', 'options': u'defaults,_netdev,backupvolfile-server={# secondary_gfs_server #}', 'fstype': 'glusterfs'})
skipping: [r5] => (item={'mountpoint': u'{# quagga_home #}', 'src': u'{# primary_gfs_server #}:/{# quagga_mnt #}', 'options': u'defaults,_netdev,backupvolfile-server={# secondary_gfs_server #}', 'fstype': 'glusterfs'})
skipping: [r4] => (item={'mountpoint': u'{# quagga_home #}', 'src': u'{# primary_gfs_server #}:/{# quagga_mnt #}', 'options': u'defaults,_netdev,backupvolfile-server={# secondary_gfs_server #}', 'fstype': 'glusterfs'})

TASK: [mrlesmithjr.quagga | config_glusterfs | configuring quagga] ************
skipping: [r1] => (item={'dest': 'daemons', 'src': 'daemons.j2'})
skipping: [r1] => (item={'dest': 'debian.conf', 'src': 'debian.conf.j2'})
skipping: [r1] => (item={'dest': 'vtysh.conf', 'src': 'vtysh.conf.j2'})
skipping: [r1] => (item={'dest': 'zebra.conf', 'src': 'zebra.conf.j2'})

TASK: [mrlesmithjr.quagga | config_glusterfs | configuring ospf] **************
skipping: [r1]

TASK: [mrlesmithjr.quagga | config_quagga | ensuring vlan package is installed] ***
skipping: [r2]
skipping: [r3]
skipping: [r1]
skipping: [r4]
skipping: [r5]

TASK: [mrlesmithjr.quagga | config_quagga | configuring network settings] *****
skipping: [r2] => (item={'name': 'net.ipv4.ip_forward', 'value': 1})
skipping: [r3] => (item={'name': 'net.ipv4.ip_forward', 'value': 1})
skipping: [r1] => (item={'name': 'net.ipv4.ip_forward', 'value': 1})
skipping: [r1] => (item={'name': 'net.ipv4.conf.all.forwarding', 'value': 1})
skipping: [r2] => (item={'name': 'net.ipv4.conf.all.forwarding', 'value': 1})
skipping: [r3] => (item={'name': 'net.ipv4.conf.all.forwarding', 'value': 1})
skipping: [r3] => (item={'name': 'net.ipv4.conf.default.forwarding', 'value': 1})
skipping: [r4] => (item={'name': 'net.ipv4.ip_forward', 'value': 1})
skipping: [r4] => (item={'name': 'net.ipv4.conf.all.forwarding', 'value': 1})
skipping: [r1] => (item={'name': 'net.ipv4.conf.default.forwarding', 'value': 1})
skipping: [r1] => (item={'name': 'net.ipv4.tcp_tw_reuse', 'value': 1})
skipping: [r1] => (item={'name': 'net.ipv4.ip_local_port_range', 'value': '1024 65023'})
skipping: [r2] => (item={'name': 'net.ipv4.conf.default.forwarding', 'value': 1})
skipping: [r2] => (item={'name': 'net.ipv4.tcp_tw_reuse', 'value': 1})
skipping: [r2] => (item={'name': 'net.ipv4.ip_local_port_range', 'value': '1024 65023'})
skipping: [r5] => (item={'name': 'net.ipv4.ip_forward', 'value': 1})
skipping: [r5] => (item={'name': 'net.ipv4.conf.all.forwarding', 'value': 1})
skipping: [r5] => (item={'name': 'net.ipv4.conf.default.forwarding', 'value': 1})
skipping: [r5] => (item={'name': 'net.ipv4.tcp_tw_reuse', 'value': 1})
skipping: [r2] => (item={'name': 'net.ipv4.tcp_max_syn_backlog', 'value': 40000})
skipping: [r3] => (item={'name': 'net.ipv4.tcp_tw_reuse', 'value': 1})
skipping: [r3] => (item={'name': 'net.ipv4.ip_local_port_range', 'value': '1024 65023'})
skipping: [r3] => (item={'name': 'net.ipv4.tcp_max_syn_backlog', 'value': 40000})
skipping: [r4] => (item={'name': 'net.ipv4.conf.default.forwarding', 'value': 1})
skipping: [r4] => (item={'name': 'net.ipv4.tcp_tw_reuse', 'value': 1})
skipping: [r1] => (item={'name': 'net.ipv4.tcp_max_syn_backlog', 'value': 40000})
skipping: [r1] => (item={'name': 'net.ipv4.tcp_max_tw_buckets', 'value': 400000})
skipping: [r1] => (item={'name': 'net.ipv4.tcp_max_orphans', 'value': 60000})
skipping: [r2] => (item={'name': 'net.ipv4.tcp_max_tw_buckets', 'value': 400000})
skipping: [r2] => (item={'name': 'net.ipv4.tcp_max_orphans', 'value': 60000})
skipping: [r2] => (item={'name': 'net.ipv4.tcp_syncookies', 'value': 1})
skipping: [r5] => (item={'name': 'net.ipv4.ip_local_port_range', 'value': '1024 65023'})
skipping: [r5] => (item={'name': 'net.ipv4.tcp_max_syn_backlog', 'value': 40000})
skipping: [r5] => (item={'name': 'net.ipv4.tcp_max_tw_buckets', 'value': 400000})
skipping: [r4] => (item={'name': 'net.ipv4.ip_local_port_range', 'value': '1024 65023'})
skipping: [r4] => (item={'name': 'net.ipv4.tcp_max_syn_backlog', 'value': 40000})
skipping: [r3] => (item={'name': 'net.ipv4.tcp_max_tw_buckets', 'value': 400000})
skipping: [r3] => (item={'name': 'net.ipv4.tcp_max_orphans', 'value': 60000})
skipping: [r2] => (item={'name': 'net.ipv4.tcp_synack_retries', 'value': 3})
skipping: [r2] => (item={'name': 'net.core.somaxconn', 'value': 40000})
skipping: [r1] => (item={'name': 'net.ipv4.tcp_syncookies', 'value': 1})
skipping: [r1] => (item={'name': 'net.ipv4.tcp_synack_retries', 'value': 3})
skipping: [r5] => (item={'name': 'net.ipv4.tcp_max_orphans', 'value': 60000})
skipping: [r5] => (item={'name': 'net.ipv4.tcp_syncookies', 'value': 1})
skipping: [r4] => (item={'name': 'net.ipv4.tcp_max_tw_buckets', 'value': 400000})
skipping: [r4] => (item={'name': 'net.ipv4.tcp_max_orphans', 'value': 60000})
skipping: [r4] => (item={'name': 'net.ipv4.tcp_syncookies', 'value': 1})
skipping: [r3] => (item={'name': 'net.ipv4.tcp_syncookies', 'value': 1})
skipping: [r3] => (item={'name': 'net.ipv4.tcp_synack_retries', 'value': 3})
skipping: [r3] => (item={'name': 'net.core.somaxconn', 'value': 40000})
skipping: [r2] => (item={'name': 'net.ipv4.tcp_fin_timeout', 'value': 5})
skipping: [r1] => (item={'name': 'net.core.somaxconn', 'value': 40000})
skipping: [r5] => (item={'name': 'net.ipv4.tcp_synack_retries', 'value': 3})
skipping: [r5] => (item={'name': 'net.core.somaxconn', 'value': 40000})
skipping: [r5] => (item={'name': 'net.ipv4.tcp_fin_timeout', 'value': 5})
skipping: [r4] => (item={'name': 'net.ipv4.tcp_synack_retries', 'value': 3})
skipping: [r4] => (item={'name': 'net.core.somaxconn', 'value': 40000})
skipping: [r1] => (item={'name': 'net.ipv4.tcp_fin_timeout', 'value': 5})
skipping: [r3] => (item={'name': 'net.ipv4.tcp_fin_timeout', 'value': 5})
skipping: [r4] => (item={'name': 'net.ipv4.tcp_fin_timeout', 'value': 5})

TASK: [mrlesmithjr.quagga | config_quagga | configuring quagga] ***************
skipping: [r1] => (item=daemons)
skipping: [r2] => (item=daemons)
skipping: [r1] => (item=debian.conf)
skipping: [r3] => (item=daemons)
skipping: [r3] => (item=debian.conf)
skipping: [r2] => (item=debian.conf)
skipping: [r2] => (item=vtysh.conf)
skipping: [r4] => (item=daemons)
skipping: [r4] => (item=debian.conf)
skipping: [r1] => (item=vtysh.conf)
skipping: [r1] => (item=zebra.conf)
skipping: [r5] => (item=daemons)
skipping: [r5] => (item=debian.conf)
skipping: [r5] => (item=vtysh.conf)
skipping: [r3] => (item=vtysh.conf)
skipping: [r2] => (item=zebra.conf)
skipping: [r4] => (item=vtysh.conf)
skipping: [r4] => (item=zebra.conf)
skipping: [r3] => (item=zebra.conf)
skipping: [r5] => (item=zebra.conf)

TASK: [mrlesmithjr.quagga | config_quagga | configuring ospf] *****************
skipping: [r1]
skipping: [r3]
skipping: [r2]
skipping: [r4]
skipping: [r5]

TASK: [mrlesmithjr.quagga | config_quagga | configuring bgp] ******************
skipping: [r1]
skipping: [r2]
skipping: [r3]
skipping: [r4]
skipping: [r5]

TASK: [mrlesmithjr.quagga | config_quagga | configuring bgp] ******************
skipping: [r3] => (item={'router_id': '1.1.1.1', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.12.12'}, {'remote_as': 123, 'neighbor': '192.168.31.13'}, {'remote_as': 141, 'neighbor': '192.168.14.14'}, {'remote_as': 151, 'neighbor': '192.168.15.15'}], 'redistribute': ['connected', 'kernel'], 'local_as': 123, 'name': 'r1'})
skipping: [r1] => (item={'router_id': '1.1.1.1', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.12.12'}, {'remote_as': 123, 'neighbor': '192.168.31.13'}, {'remote_as': 141, 'neighbor': '192.168.14.14'}, {'remote_as': 151, 'neighbor': '192.168.15.15'}], 'redistribute': ['connected', 'kernel'], 'local_as': 123, 'name': 'r1'})
skipping: [r2] => (item={'router_id': '1.1.1.1', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.12.12'}, {'remote_as': 123, 'neighbor': '192.168.31.13'}, {'remote_as': 141, 'neighbor': '192.168.14.14'}, {'remote_as': 151, 'neighbor': '192.168.15.15'}], 'redistribute': ['connected', 'kernel'], 'local_as': 123, 'name': 'r1'})
skipping: [r2] => (item={'router_id': '2.2.2.2', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.12.11'}, {'remote_as': 123, 'neighbor': '192.168.23.13'}], 'redistribute': ['connected', 'kernel'], 'local_as': 123, 'name': 'r2'})
skipping: [r2] => (item={'router_id': '3.3.3.3', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.23.12'}, {'remote_as': 123, 'neighbor': '192.168.31.11'}], 'redistribute': ['connected', 'kernel'], 'local_as': 123, 'name': 'r3'})
skipping: [r4] => (item={'router_id': '1.1.1.1', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.12.12'}, {'remote_as': 123, 'neighbor': '192.168.31.13'}, {'remote_as': 141, 'neighbor': '192.168.14.14'}, {'remote_as': 151, 'neighbor': '192.168.15.15'}], 'redistribute': ['connected', 'kernel'], 'local_as': 123, 'name': 'r1'})
skipping: [r4] => (item={'router_id': '2.2.2.2', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.12.11'}, {'remote_as': 123, 'neighbor': '192.168.23.13'}], 'redistribute': ['connected', 'kernel'], 'local_as': 123, 'name': 'r2'})
skipping: [r1] => (item={'router_id': '2.2.2.2', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.12.11'}, {'remote_as': 123, 'neighbor': '192.168.23.13'}], 'redistribute': ['connected', 'kernel'], 'local_as': 123, 'name': 'r2'})
skipping: [r1] => (item={'router_id': '3.3.3.3', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.23.12'}, {'remote_as': 123, 'neighbor': '192.168.31.11'}], 'redistribute': ['connected', 'kernel'], 'local_as': 123, 'name': 'r3'})
skipping: [r1] => (item={'router_id': '4.4.4.4', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.14.11'}], 'redistribute': ['connected', 'kernel'], 'local_as': 141, 'name': 'r4'})
skipping: [r5] => (item={'router_id': '1.1.1.1', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.12.12'}, {'remote_as': 123, 'neighbor': '192.168.31.13'}, {'remote_as': 141, 'neighbor': '192.168.14.14'}, {'remote_as': 151, 'neighbor': '192.168.15.15'}], 'redistribute': ['connected', 'kernel'], 'local_as': 123, 'name': 'r1'})
skipping: [r5] => (item={'router_id': '2.2.2.2', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.12.11'}, {'remote_as': 123, 'neighbor': '192.168.23.13'}], 'redistribute': ['connected', 'kernel'], 'local_as': 123, 'name': 'r2'})
skipping: [r5] => (item={'router_id': '3.3.3.3', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.23.12'}, {'remote_as': 123, 'neighbor': '192.168.31.11'}], 'redistribute': ['connected', 'kernel'], 'local_as': 123, 'name': 'r3'})
skipping: [r2] => (item={'router_id': '4.4.4.4', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.14.11'}], 'redistribute': ['connected', 'kernel'], 'local_as': 141, 'name': 'r4'})
skipping: [r4] => (item={'router_id': '3.3.3.3', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.23.12'}, {'remote_as': 123, 'neighbor': '192.168.31.11'}], 'redistribute': ['connected', 'kernel'], 'local_as': 123, 'name': 'r3'})
skipping: [r4] => (item={'router_id': '4.4.4.4', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.14.11'}], 'redistribute': ['connected', 'kernel'], 'local_as': 141, 'name': 'r4'})
skipping: [r3] => (item={'router_id': '2.2.2.2', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.12.11'}, {'remote_as': 123, 'neighbor': '192.168.23.13'}], 'redistribute': ['connected', 'kernel'], 'local_as': 123, 'name': 'r2'})
skipping: [r3] => (item={'router_id': '3.3.3.3', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.23.12'}, {'remote_as': 123, 'neighbor': '192.168.31.11'}], 'redistribute': ['connected', 'kernel'], 'local_as': 123, 'name': 'r3'})
skipping: [r5] => (item={'router_id': '4.4.4.4', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.14.11'}], 'redistribute': ['connected', 'kernel'], 'local_as': 141, 'name': 'r4'})
skipping: [r3] => (item={'router_id': '4.4.4.4', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.14.11'}], 'redistribute': ['connected', 'kernel'], 'local_as': 141, 'name': 'r4'})

TASK: [mrlesmithjr.quagga | config_quagga | setting permissions on files within /etc/quagga] ***
skipping: [r2]
skipping: [r3]
skipping: [r1]
skipping: [r4]
skipping: [r5]

TASK: [mrlesmithjr.quagga | config_quagga | setting permissions on folder /etc/quagga] ***
skipping: [r1]
skipping: [r2]
skipping: [r3]
skipping: [r4]
skipping: [r5]

TASK: [mrlesmithjr.quagga | config_keepalived | reconfiguring keepalived] *****
skipping: [r2]
skipping: [r1]
skipping: [r3]
skipping: [r4]
skipping: [r5]

TASK: [mrlesmithjr.quagga | config_keepalived | copying keepalived scripts] ***
skipping: [r3] => (item=backup_quagga.sh)
skipping: [r1] => (item=backup_quagga.sh)
skipping: [r2] => (item=backup_quagga.sh)
skipping: [r2] => (item=fault_quagga.sh)
skipping: [r3] => (item=fault_quagga.sh)
skipping: [r3] => (item=master_quagga.sh)
skipping: [r4] => (item=backup_quagga.sh)
skipping: [r4] => (item=fault_quagga.sh)
skipping: [r1] => (item=fault_quagga.sh)
skipping: [r1] => (item=master_quagga.sh)
skipping: [r1] => (item=primary-backup.sh)
skipping: [r2] => (item=master_quagga.sh)
skipping: [r5] => (item=backup_quagga.sh)
skipping: [r3] => (item=primary-backup.sh)
skipping: [r4] => (item=master_quagga.sh)
skipping: [r4] => (item=primary-backup.sh)
skipping: [r2] => (item=primary-backup.sh)
skipping: [r5] => (item=fault_quagga.sh)
skipping: [r5] => (item=master_quagga.sh)
skipping: [r5] => (item=primary-backup.sh)

TASK: [installing packages] ***************************************************
changed: [r3] => (item=traceroute)
changed: [r2] => (item=traceroute)
changed: [r1] => (item=traceroute)
changed: [r5] => (item=traceroute)
changed: [r4] => (item=traceroute)

PLAY RECAP ********************************************************************
r1                         : ok=15   changed=4    unreachable=0    failed=0
r2                         : ok=15   changed=4    unreachable=0    failed=0
r3                         : ok=15   changed=4    unreachable=0    failed=0
r4                         : ok=15   changed=4    unreachable=0    failed=0
r5                         : ok=15   changed=4    unreachable=0    failed=0
```

Now we are getting somewhere. But notice that the majority of our tasks
were skipped. This is because our variables are defined as false for the
majority of our tasks. Quagga itself was installed but not configured.

A quick look at our quagga folder shows only the following.

```bash
ls -l /etc/quagga
....
total 8
-rw-r--r-- 1 root root 990 Mar 21  2014 daemons
-rw-r--r-- 1 root root 945 Mar 21  2014 debian.conf
```

So before we go any further let's look at our interfaces and routes.
This is just to show a before and after view as we start to configure
OSPF.

```bash
ip addr
....
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:c5:c8:e6 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fec5:c8e6/64 scope link
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:d7:96:14 brd ff:ff:ff:ff:ff:ff
    inet 192.168.250.101/24 brd 192.168.250.255 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fed7:9614/64 scope link
       valid_lft forever preferred_lft forever
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:56:c0:94 brd ff:ff:ff:ff:ff:ff
    inet 192.168.12.11/24 brd 192.168.12.255 scope global eth2
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe56:c094/64 scope link
       valid_lft forever preferred_lft forever
5: eth3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:b7:d9:7a brd ff:ff:ff:ff:ff:ff
    inet 192.168.14.11/24 brd 192.168.14.255 scope global eth3
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:feb7:d97a/64 scope link
       valid_lft forever preferred_lft forever
6: eth4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:f5:3b:af brd ff:ff:ff:ff:ff:ff
    inet 192.168.15.11/24 brd 192.168.15.255 scope global eth4
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fef5:3baf/64 scope link
       valid_lft forever preferred_lft forever
7: eth5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:29:4d:45 brd ff:ff:ff:ff:ff:ff
    inet 192.168.31.11/24 brd 192.168.31.255 scope global eth5
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe29:4d45/64 scope link
       valid_lft forever preferred_lft forever
8: eth6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:53:e1:d1 brd ff:ff:ff:ff:ff:ff
    inet 1.1.1.10/24 brd 1.1.1.255 scope global eth6
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe53:e1d1/64 scope link
       valid_lft forever preferred_lft forever

ip route
....
default via 10.0.2.2 dev eth0
1.1.1.0/24 dev eth6  proto kernel  scope link  src 1.1.1.10
10.0.2.0/24 dev eth0  proto kernel  scope link  src 10.0.2.15
192.168.12.0/24 dev eth2  proto kernel  scope link  src 192.168.12.11
192.168.14.0/24 dev eth3  proto kernel  scope link  src 192.168.14.11
192.168.15.0/24 dev eth4  proto kernel  scope link  src 192.168.15.11
192.168.31.0/24 dev eth5  proto kernel  scope link  src 192.168.31.11
192.168.250.0/24 dev eth1  proto kernel  scope link  src 192.168.250.101
```

So now let's actually configure Quagga by adjusting our Ansible
variable but not for any routing but rather just the quagga daemon.

```bash
cd /vagrant
nano group_vars/quagga-routers
```

At the very top of the file we are going to change config_quagga: false
to config_quagga: true
Before:

```yaml
---
config_quagga: false
```

After:

```yaml
---
config_quagga: true
```

Now save the file.

Now let's run our playbook once again.

```raw
cd /vagrant
ansible-playbook -i hosts playbook.yml
....
PLAY [quagga-routers] *********************************************************

GATHERING FACTS ***************************************************************
ok: [r2]
ok: [r3]
ok: [r1]
ok: [r5]
ok: [r4]

TASK: [mrlesmithjr.quagga | debian | installing quagga pre-reqs] **************
ok: [r2] => (item=vlan)
ok: [r1] => (item=vlan)
ok: [r4] => (item=vlan)
ok: [r5] => (item=vlan)
ok: [r3] => (item=vlan)

TASK: [mrlesmithjr.quagga | debian | installing quagga] ***********************
ok: [r1]
ok: [r4]
ok: [r3]
ok: [r2]
ok: [r5]

TASK: [mrlesmithjr.quagga | debian | enabling quagga] *************************
changed: [r2]
changed: [r1]
changed: [r4]
changed: [r3]
changed: [r5]

TASK: [mrlesmithjr.quagga | configuring interfaces and vlans with dhcp mgmt] ***
skipping: [r2] => (item=vlan_config)
skipping: [r3] => (item=vlan_config)
skipping: [r1] => (item=vlan_config)
skipping: [r4] => (item=vlan_config)
skipping: [r5] => (item=vlan_config)

TASK: [mrlesmithjr.quagga | configuring interfaces and vlans with static mgmt] ***
skipping: [r2] => (item=vlan_config)
skipping: [r1] => (item=vlan_config)
skipping: [r3] => (item=vlan_config)
skipping: [r4] => (item=vlan_config)
skipping: [r5] => (item=vlan_config)

TASK: [mrlesmithjr.quagga | Create the directory for interface cfg files] *****
skipping: [r1]
skipping: [r2]
skipping: [r3]
skipping: [r5]
skipping: [r4]

TASK: [mrlesmithjr.quagga | Create the network configuration file for ethernet devices] ***
skipping: [r3] => (item=vlan_config)
skipping: [r2] => (item=vlan_config)
skipping: [r4] => (item=vlan_config)
skipping: [r1] => (item=vlan_config)
skipping: [r5] => (item=vlan_config)

TASK: [mrlesmithjr.quagga | reinitializing interfaces] ************************
skipping: [r1] => (item={u'skipped': True, u'changed': False})
skipping: [r2] => (item={u'skipped': True, u'changed': False})
skipping: [r3] => (item={u'skipped': True, u'changed': False})
skipping: [r4] => (item={u'skipped': True, u'changed': False})
skipping: [r5] => (item={u'skipped': True, u'changed': False})

TASK: [mrlesmithjr.quagga | config_glusterfs | checking to see if /etc/quagga has already been moved] ***
skipping: [r3]
skipping: [r1]
skipping: [r2]
skipping: [r4]
skipping: [r5]

TASK: [mrlesmithjr.quagga | config_glusterfs | moving existing /etc/quagga] ***
skipping: [r2]
skipping: [r1]
skipping: [r3]
skipping: [r4]
skipping: [r5]

TASK: [mrlesmithjr.quagga | config_glusterfs | checking again if /etc/quagga has already been moved] ***
skipping: [r1]
skipping: [r2]
skipping: [r3]
skipping: [r4]
skipping: [r5]

TASK: [mrlesmithjr.quagga | config_glusterfs | touching file in quagga_backup_dir] ***
skipping: [r2]
skipping: [r3]
skipping: [r1]
skipping: [r4]
skipping: [r5]

TASK: [mrlesmithjr.quagga | config_glusterfs | mounting gluster volumes - quagga] ***
skipping: [r1] => (item={'mountpoint': u'{# quagga_home #}', 'src': u'{# primary_gfs_server #}:/{# quagga_mnt #}', 'options': u'defaults,_netdev,backupvolfile-server={# secondary_gfs_server #}', 'fstype': 'glusterfs'})
skipping: [r2] => (item={'mountpoint': u'{# quagga_home #}', 'src': u'{# primary_gfs_server #}:/{# quagga_mnt #}', 'options': u'defaults,_netdev,backupvolfile-server={# secondary_gfs_server #}', 'fstype': 'glusterfs'})
skipping: [r4] => (item={'mountpoint': u'{# quagga_home #}', 'src': u'{# primary_gfs_server #}:/{# quagga_mnt #}', 'options': u'defaults,_netdev,backupvolfile-server={# secondary_gfs_server #}', 'fstype': 'glusterfs'})
skipping: [r3] => (item={'mountpoint': u'{# quagga_home #}', 'src': u'{# primary_gfs_server #}:/{# quagga_mnt #}', 'options': u'defaults,_netdev,backupvolfile-server={# secondary_gfs_server #}', 'fstype': 'glusterfs'})
skipping: [r5] => (item={'mountpoint': u'{# quagga_home #}', 'src': u'{# primary_gfs_server #}:/{# quagga_mnt #}', 'options': u'defaults,_netdev,backupvolfile-server={# secondary_gfs_server #}', 'fstype': 'glusterfs'})

TASK: [mrlesmithjr.quagga | config_glusterfs | configuring quagga] ************
skipping: [r1] => (item={'dest': 'daemons', 'src': 'daemons.j2'})
skipping: [r1] => (item={'dest': 'debian.conf', 'src': 'debian.conf.j2'})
skipping: [r1] => (item={'dest': 'vtysh.conf', 'src': 'vtysh.conf.j2'})
skipping: [r1] => (item={'dest': 'zebra.conf', 'src': 'zebra.conf.j2'})

TASK: [mrlesmithjr.quagga | config_glusterfs | configuring ospf] **************
skipping: [r1]

TASK: [mrlesmithjr.quagga | config_quagga | ensuring vlan package is installed] ***
ok: [r1]
ok: [r3]
ok: [r2]
ok: [r4]
ok: [r5]

TASK: [mrlesmithjr.quagga | config_quagga | configuring network settings] *****
changed: [r2] => (item={'name': 'net.ipv4.ip_forward', 'value': 1})
changed: [r3] => (item={'name': 'net.ipv4.ip_forward', 'value': 1})
changed: [r5] => (item={'name': 'net.ipv4.ip_forward', 'value': 1})
changed: [r4] => (item={'name': 'net.ipv4.ip_forward', 'value': 1})
changed: [r1] => (item={'name': 'net.ipv4.ip_forward', 'value': 1})
changed: [r2] => (item={'name': 'net.ipv4.conf.all.forwarding', 'value': 1})
changed: [r3] => (item={'name': 'net.ipv4.conf.all.forwarding', 'value': 1})
changed: [r4] => (item={'name': 'net.ipv4.conf.all.forwarding', 'value': 1})
changed: [r5] => (item={'name': 'net.ipv4.conf.all.forwarding', 'value': 1})
changed: [r1] => (item={'name': 'net.ipv4.conf.all.forwarding', 'value': 1})
changed: [r2] => (item={'name': 'net.ipv4.conf.default.forwarding', 'value': 1})
changed: [r5] => (item={'name': 'net.ipv4.conf.default.forwarding', 'value': 1})
changed: [r3] => (item={'name': 'net.ipv4.conf.default.forwarding', 'value': 1})
changed: [r4] => (item={'name': 'net.ipv4.conf.default.forwarding', 'value': 1})
changed: [r2] => (item={'name': 'net.ipv4.tcp_tw_reuse', 'value': 1})
changed: [r1] => (item={'name': 'net.ipv4.conf.default.forwarding', 'value': 1})
changed: [r1] => (item={'name': 'net.ipv4.tcp_tw_reuse', 'value': 1})
changed: [r5] => (item={'name': 'net.ipv4.tcp_tw_reuse', 'value': 1})
changed: [r3] => (item={'name': 'net.ipv4.tcp_tw_reuse', 'value': 1})
changed: [r2] => (item={'name': 'net.ipv4.ip_local_port_range', 'value': '1024 65023'})
changed: [r4] => (item={'name': 'net.ipv4.tcp_tw_reuse', 'value': 1})
changed: [r1] => (item={'name': 'net.ipv4.ip_local_port_range', 'value': '1024 65023'})
changed: [r5] => (item={'name': 'net.ipv4.ip_local_port_range', 'value': '1024 65023'})
changed: [r2] => (item={'name': 'net.ipv4.tcp_max_syn_backlog', 'value': 40000})
changed: [r4] => (item={'name': 'net.ipv4.ip_local_port_range', 'value': '1024 65023'})
changed: [r3] => (item={'name': 'net.ipv4.ip_local_port_range', 'value': '1024 65023'})
changed: [r1] => (item={'name': 'net.ipv4.tcp_max_syn_backlog', 'value': 40000})
changed: [r2] => (item={'name': 'net.ipv4.tcp_max_tw_buckets', 'value': 400000})
changed: [r3] => (item={'name': 'net.ipv4.tcp_max_syn_backlog', 'value': 40000})
changed: [r5] => (item={'name': 'net.ipv4.tcp_max_syn_backlog', 'value': 40000})
changed: [r4] => (item={'name': 'net.ipv4.tcp_max_syn_backlog', 'value': 40000})
changed: [r1] => (item={'name': 'net.ipv4.tcp_max_tw_buckets', 'value': 400000})
changed: [r5] => (item={'name': 'net.ipv4.tcp_max_tw_buckets', 'value': 400000})
changed: [r3] => (item={'name': 'net.ipv4.tcp_max_tw_buckets', 'value': 400000})
changed: [r2] => (item={'name': 'net.ipv4.tcp_max_orphans', 'value': 60000})
changed: [r4] => (item={'name': 'net.ipv4.tcp_max_tw_buckets', 'value': 400000})
changed: [r1] => (item={'name': 'net.ipv4.tcp_max_orphans', 'value': 60000})
changed: [r3] => (item={'name': 'net.ipv4.tcp_max_orphans', 'value': 60000})
changed: [r2] => (item={'name': 'net.ipv4.tcp_syncookies', 'value': 1})
changed: [r4] => (item={'name': 'net.ipv4.tcp_max_orphans', 'value': 60000})
changed: [r1] => (item={'name': 'net.ipv4.tcp_syncookies', 'value': 1})
changed: [r5] => (item={'name': 'net.ipv4.tcp_max_orphans', 'value': 60000})
changed: [r2] => (item={'name': 'net.ipv4.tcp_synack_retries', 'value': 3})
changed: [r4] => (item={'name': 'net.ipv4.tcp_syncookies', 'value': 1})
changed: [r5] => (item={'name': 'net.ipv4.tcp_syncookies', 'value': 1})
changed: [r1] => (item={'name': 'net.ipv4.tcp_synack_retries', 'value': 3})
changed: [r3] => (item={'name': 'net.ipv4.tcp_syncookies', 'value': 1})
changed: [r2] => (item={'name': 'net.core.somaxconn', 'value': 40000})
changed: [r1] => (item={'name': 'net.core.somaxconn', 'value': 40000})
changed: [r5] => (item={'name': 'net.ipv4.tcp_synack_retries', 'value': 3})
changed: [r4] => (item={'name': 'net.ipv4.tcp_synack_retries', 'value': 3})
changed: [r3] => (item={'name': 'net.ipv4.tcp_synack_retries', 'value': 3})
changed: [r2] => (item={'name': 'net.ipv4.tcp_fin_timeout', 'value': 5})
changed: [r1] => (item={'name': 'net.ipv4.tcp_fin_timeout', 'value': 5})
changed: [r3] => (item={'name': 'net.core.somaxconn', 'value': 40000})
changed: [r5] => (item={'name': 'net.core.somaxconn', 'value': 40000})
changed: [r4] => (item={'name': 'net.core.somaxconn', 'value': 40000})
changed: [r4] => (item={'name': 'net.ipv4.tcp_fin_timeout', 'value': 5})
changed: [r3] => (item={'name': 'net.ipv4.tcp_fin_timeout', 'value': 5})
changed: [r5] => (item={'name': 'net.ipv4.tcp_fin_timeout', 'value': 5})

TASK: [mrlesmithjr.quagga | config_quagga | configuring quagga] ***************
changed: [r1] => (item=daemons)
changed: [r2] => (item=daemons)
changed: [r5] => (item=daemons)
changed: [r3] => (item=daemons)
changed: [r4] => (item=daemons)
changed: [r1] => (item=debian.conf)
changed: [r3] => (item=debian.conf)
changed: [r5] => (item=debian.conf)
changed: [r4] => (item=debian.conf)
changed: [r2] => (item=debian.conf)
changed: [r1] => (item=vtysh.conf)
changed: [r4] => (item=vtysh.conf)
changed: [r3] => (item=vtysh.conf)
changed: [r5] => (item=vtysh.conf)
changed: [r2] => (item=vtysh.conf)
changed: [r1] => (item=zebra.conf)
changed: [r3] => (item=zebra.conf)
changed: [r4] => (item=zebra.conf)
changed: [r5] => (item=zebra.conf)
changed: [r2] => (item=zebra.conf)

TASK: [mrlesmithjr.quagga | config_quagga | configuring ospf] *****************
skipping: [r2]
skipping: [r3]
skipping: [r1]
skipping: [r4]
skipping: [r5]

TASK: [mrlesmithjr.quagga | config_quagga | configuring bgp] ******************
skipping: [r1]
skipping: [r3]
skipping: [r2]
skipping: [r4]
skipping: [r5]

TASK: [mrlesmithjr.quagga | config_quagga | configuring bgp] ******************
skipping: [r3] => (item={'router_id': '1.1.1.1', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.12.12'}, {'remote_as': 123, 'neighbor': '192.168.31.13'}, {'remote_as': 141, 'neighbor': '192.168.14.14'}, {'remote_as': 151, 'neighbor': '192.168.15.15'}], 'redistribute': ['connected', 'kernel'], 'local_as': 123, 'name': 'r1'})
skipping: [r2] => (item={'router_id': '1.1.1.1', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.12.12'}, {'remote_as': 123, 'neighbor': '192.168.31.13'}, {'remote_as': 141, 'neighbor': '192.168.14.14'}, {'remote_as': 151, 'neighbor': '192.168.15.15'}], 'redistribute': ['connected', 'kernel'], 'local_as': 123, 'name': 'r1'})
skipping: [r1] => (item={'router_id': '1.1.1.1', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.12.12'}, {'remote_as': 123, 'neighbor': '192.168.31.13'}, {'remote_as': 141, 'neighbor': '192.168.14.14'}, {'remote_as': 151, 'neighbor': '192.168.15.15'}], 'redistribute': ['connected', 'kernel'], 'local_as': 123, 'name': 'r1'})
skipping: [r4] => (item={'router_id': '1.1.1.1', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.12.12'}, {'remote_as': 123, 'neighbor': '192.168.31.13'}, {'remote_as': 141, 'neighbor': '192.168.14.14'}, {'remote_as': 151, 'neighbor': '192.168.15.15'}], 'redistribute': ['connected', 'kernel'], 'local_as': 123, 'name': 'r1'})
skipping: [r4] => (item={'router_id': '2.2.2.2', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.12.11'}, {'remote_as': 123, 'neighbor': '192.168.23.13'}], 'redistribute': ['connected', 'kernel'], 'local_as': 123, 'name': 'r2'})
skipping: [r5] => (item={'router_id': '1.1.1.1', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.12.12'}, {'remote_as': 123, 'neighbor': '192.168.31.13'}, {'remote_as': 141, 'neighbor': '192.168.14.14'}, {'remote_as': 151, 'neighbor': '192.168.15.15'}], 'redistribute': ['connected', 'kernel'], 'local_as': 123, 'name': 'r1'})
skipping: [r1] => (item={'router_id': '2.2.2.2', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.12.11'}, {'remote_as': 123, 'neighbor': '192.168.23.13'}], 'redistribute': ['connected', 'kernel'], 'local_as': 123, 'name': 'r2'})
skipping: [r2] => (item={'router_id': '2.2.2.2', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.12.11'}, {'remote_as': 123, 'neighbor': '192.168.23.13'}], 'redistribute': ['connected', 'kernel'], 'local_as': 123, 'name': 'r2'})
skipping: [r3] => (item={'router_id': '2.2.2.2', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.12.11'}, {'remote_as': 123, 'neighbor': '192.168.23.13'}], 'redistribute': ['connected', 'kernel'], 'local_as': 123, 'name': 'r2'})
skipping: [r4] => (item={'router_id': '3.3.3.3', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.23.12'}, {'remote_as': 123, 'neighbor': '192.168.31.11'}], 'redistribute': ['connected', 'kernel'], 'local_as': 123, 'name': 'r3'})
skipping: [r5] => (item={'router_id': '2.2.2.2', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.12.11'}, {'remote_as': 123, 'neighbor': '192.168.23.13'}], 'redistribute': ['connected', 'kernel'], 'local_as': 123, 'name': 'r2'})
skipping: [r3] => (item={'router_id': '3.3.3.3', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.23.12'}, {'remote_as': 123, 'neighbor': '192.168.31.11'}], 'redistribute': ['connected', 'kernel'], 'local_as': 123, 'name': 'r3'})
skipping: [r2] => (item={'router_id': '3.3.3.3', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.23.12'}, {'remote_as': 123, 'neighbor': '192.168.31.11'}], 'redistribute': ['connected', 'kernel'], 'local_as': 123, 'name': 'r3'})
skipping: [r1] => (item={'router_id': '3.3.3.3', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.23.12'}, {'remote_as': 123, 'neighbor': '192.168.31.11'}], 'redistribute': ['connected', 'kernel'], 'local_as': 123, 'name': 'r3'})
skipping: [r1] => (item={'router_id': '4.4.4.4', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.14.11'}], 'redistribute': ['connected', 'kernel'], 'local_as': 141, 'name': 'r4'})
skipping: [r5] => (item={'router_id': '3.3.3.3', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.23.12'}, {'remote_as': 123, 'neighbor': '192.168.31.11'}], 'redistribute': ['connected', 'kernel'], 'local_as': 123, 'name': 'r3'})
skipping: [r4] => (item={'router_id': '4.4.4.4', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.14.11'}], 'redistribute': ['connected', 'kernel'], 'local_as': 141, 'name': 'r4'})
skipping: [r3] => (item={'router_id': '4.4.4.4', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.14.11'}], 'redistribute': ['connected', 'kernel'], 'local_as': 141, 'name': 'r4'})
skipping: [r2] => (item={'router_id': '4.4.4.4', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.14.11'}], 'redistribute': ['connected', 'kernel'], 'local_as': 141, 'name': 'r4'})
skipping: [r5] => (item={'router_id': '4.4.4.4', 'neighbors': [{'remote_as': 123, 'neighbor': '192.168.14.11'}], 'redistribute': ['connected', 'kernel'], 'local_as': 141, 'name': 'r4'})

TASK: [mrlesmithjr.quagga | config_quagga | setting permissions on files within /etc/quagga] ***
changed: [r2]
changed: [r1]
changed: [r3]
changed: [r5]
changed: [r4]

TASK: [mrlesmithjr.quagga | config_quagga | setting permissions on folder /etc/quagga] ***
changed: [r1]
changed: [r2]
changed: [r3]
changed: [r4]
changed: [r5]

TASK: [mrlesmithjr.quagga | config_keepalived | reconfiguring keepalived] *****
skipping: [r2]
skipping: [r1]
skipping: [r3]
skipping: [r4]
skipping: [r5]

TASK: [mrlesmithjr.quagga | config_keepalived | copying keepalived scripts] ***
skipping: [r3] => (item=backup_quagga.sh)
skipping: [r2] => (item=backup_quagga.sh)
skipping: [r1] => (item=backup_quagga.sh)
skipping: [r1] => (item=fault_quagga.sh)
skipping: [r3] => (item=fault_quagga.sh)
skipping: [r3] => (item=master_quagga.sh)
skipping: [r2] => (item=fault_quagga.sh)
skipping: [r4] => (item=backup_quagga.sh)
skipping: [r4] => (item=fault_quagga.sh)
skipping: [r4] => (item=master_quagga.sh)
skipping: [r1] => (item=master_quagga.sh)
skipping: [r1] => (item=primary-backup.sh)
skipping: [r2] => (item=master_quagga.sh)
skipping: [r2] => (item=primary-backup.sh)
skipping: [r5] => (item=backup_quagga.sh)
skipping: [r5] => (item=fault_quagga.sh)
skipping: [r5] => (item=master_quagga.sh)
skipping: [r3] => (item=primary-backup.sh)
skipping: [r4] => (item=primary-backup.sh)
skipping: [r5] => (item=primary-backup.sh)

TASK: [installing packages] ***************************************************
ok: [r1] => (item=traceroute)
ok: [r2] => (item=traceroute)
ok: [r3] => (item=traceroute)
ok: [r5] => (item=traceroute)
ok: [r4] => (item=traceroute)

NOTIFIED: [mrlesmithjr.quagga | restart quagga] *******************************
changed: [r2]
changed: [r3]
changed: [r4]
changed: [r1]
changed: [r5]

PLAY RECAP ********************************************************************
r1                         : ok=19   changed=6    unreachable=0    failed=0
r2                         : ok=19   changed=6    unreachable=0    failed=0
r3                         : ok=19   changed=6    unreachable=0    failed=0
r4                         : ok=19   changed=6    unreachable=0    failed=0
r5                         : ok=19   changed=6    unreachable=0    failed=0
```

As you now notice is that there were changes applied this time. But
again no routing configurations made. However our quagga daemon is now
running and it is running on TCP port 2601 on our loopback address. We
can actually connect to this and look at our configuration (Cisco like).

```bash
telnet localhost 2601
....
Connected to localhost.
Escape character is '^]'.

Hello, this is Quagga (version 0.99.22.4).
Copyright 1996-2005 Kunihiro Ishiguro, et al.


User Access Verification

Password:
```

Our password is defined in our defaults and `group_vars/quagga-routers`
as the following.

```yaml
quagga_enable_password: quagga
quagga_password: quagga
```

So if we enter quagga at our password prompt we will be logged in.

```bash
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.

Hello, this is Quagga (version 0.99.22.4).
Copyright 1996-2005 Kunihiro Ishiguro, et al.


User Access Verification

Password:
r1>
```

Now just like a Cisco router do the following to enter enable mode.
Remember our enable password is quagga as well.

```bash
en
....
r1> en
Password:
....
r1> en
Password:
r1#
```

Now do a quick show run to view the configuration.

```bash
sh run
....
Current configuration:
!
hostname r1
password 8 uDmze5f4ocFeE
enable password 8 o9JymP3xvy8Fc
log file /var/log/quagga/zebra.log
log syslog
service password-encryption
!
debug zebra events
debug zebra packet
!
interface eth0
 ipv6 nd suppress-ra
!
interface eth1
 ipv6 nd suppress-ra
!
interface eth2
 ipv6 nd suppress-ra
!
interface eth3
 ipv6 nd suppress-ra
!
interface eth4
 ipv6 nd suppress-ra
!
interface eth5
 ipv6 nd suppress-ra
!
interface eth6
 ipv6 nd suppress-ra
!
interface lo
!
ip forwarding
ipv6 forwarding
!
!
line vty
!
end
```

Now type exit to exit the daemon so we may proceed. But again before
proceeding let's look at our routes to ensure that they are the same as
they were previously and there is absolutely no OSPF configured at this
time. And also do some ping checks to validate that we cannot
successfully reach some of our other router interfaces.

```bash
ip route
....
default via 10.0.2.2 dev eth0
1.1.1.0/24 dev eth6  proto kernel  scope link  src 1.1.1.10
10.0.2.0/24 dev eth0  proto kernel  scope link  src 10.0.2.15
192.168.12.0/24 dev eth2  proto kernel  scope link  src 192.168.12.11
192.168.14.0/24 dev eth3  proto kernel  scope link  src 192.168.14.11
192.168.15.0/24 dev eth4  proto kernel  scope link  src 192.168.15.11
192.168.31.0/24 dev eth5  proto kernel  scope link  src 192.168.31.11
192.168.250.0/24 dev eth1  proto kernel  scope link  src 192.168.250.101
```

Let's attempt to ping some of other router interfaces by looking at our
diagram below to identify some of those interfaces. Feel free to do some
additional testing and validate as to why or why not the ping is
successful.

![rp_ossrouting-bgp-drawing-New-Page-300x2321-300x232.png](../../assets/ossrouting-bgp-drawing-New-Page-300x2321-300x2321-300x232.png)

```bash
ping -c 4 2.2.2.10
....
PING 2.2.2.10 (2.2.2.10) 56(84) bytes of data.
--- 2.2.2.10 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3017ms
....
ping -c 4 192.168.23.13
....
PING 192.168.23.13 (192.168.23.13) 56(84) bytes of data.

--- 192.168.23.13 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3000ms
....
ping -c 4. 192.168.12.12
....
PING 192.168.12.12 (192.168.12.12) 56(84) bytes of data.
64 bytes from 192.168.12.12: icmp_seq=1 ttl=64 time=0.502 ms
64 bytes from 192.168.12.12: icmp_seq=2 ttl=64 time=0.412 ms
64 bytes from 192.168.12.12: icmp_seq=3 ttl=64 time=0.305 ms
64 bytes from 192.168.12.12: icmp_seq=4 ttl=64 time=0.288 ms

--- 192.168.12.12 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 2999ms
rtt min/avg/max/mdev = 0.288/0.376/0.502/0.089 ms
```

Can you figure out why the last ping was successful?

Now let the routing begin :)

Let's go ahead and define the variables required to configure our OSPF
configuration and setup all OSPF routing automatically for us.

```bash
nano group_vars/quagga-routers
....
```

Now change the following two variables from false to true.

```yaml
quagga_config_ospfd: false  #defines if quagga ospfd should be configured based on quagga_ospf_ vars...makes it easy to disable auto routing in order to define your routes manually
quagga_enable_ospfd: false
....
quagga_config_ospfd: true  #defines if quagga ospfd should be configured based on quagga_ospf_ vars...makes it easy to disable auto routing in order to define your routes manually
quagga_enable_ospfd: true
```

Now save the file and run the playbook again.

```bash
cd /vagrant
ansible-playbook -i hosts playbook.yml
....
```

And if you were to scroll up and look at what happened you will notice
the following.

```raw
TASK: [mrlesmithjr.quagga | config_quagga | configuring ospf] *****************
changed: [r1]
changed: [r5]
changed: [r2]
changed: [r3]
changed: [r4]
```

That is where all the magic happened. So let's take a look and see what
was changed.
If we now take a look at our routes.

```bash
ip route
....
default via 10.0.2.2 dev eth0
1.1.1.0/24 dev eth6  proto kernel  scope link  src 1.1.1.10
2.2.2.0/24 via 192.168.250.102 dev eth1  proto zebra  metric 20
3.3.3.0/24 via 192.168.250.103 dev eth1  proto zebra  metric 20
4.4.4.0/24 via 192.168.250.104 dev eth1  proto zebra  metric 20
5.5.5.0/24 via 192.168.250.105 dev eth1  proto zebra  metric 20
10.0.2.0/24 dev eth0  proto kernel  scope link  src 10.0.2.15
192.168.12.0/24 dev eth2  proto kernel  scope link  src 192.168.12.11
192.168.14.0/24 dev eth3  proto kernel  scope link  src 192.168.14.11
192.168.15.0/24 dev eth4  proto kernel  scope link  src 192.168.15.11
192.168.23.0/24  proto zebra  metric 20
    nexthop via 192.168.250.102  dev eth1 weight 1
    nexthop via 192.168.250.103  dev eth1 weight 1
192.168.31.0/24 dev eth5  proto kernel  scope link  src 192.168.31.11
192.168.41.0/24 via 192.168.250.104 dev eth1  proto zebra  metric 20
192.168.51.0/24 via 192.168.250.105 dev eth1  proto zebra  metric 20
192.168.250.0/24 dev eth1  proto kernel  scope link  src 192.168.250.101
```

We now have routes to all of the interfaces on our other routers for
connected interfaces. In our
`/etc/ansible/roles/mrlesmithjr.quagga/defaults/main.yml` file we have a defined
variable quagga_ospf_redistribute which looks like the following.

```yaml
quagga_ospf_redistribute:
  - connected
#  - kernel
#  - static
#  - isis
#  - rip
```

The above is what configured our OSPF setup to inject connected
interfaces into our routing topology. Maybe you don't want this so you
can copy that variable into `group_vars/quagga-routers` and change it up
and see how each different setting impacts our OSPF routing
configurations. Feel free to experiment and learn. So if you remember from above
that we initiated a telnet session in order to look at our configuration in a
Cisco like manner and that was TCP 2601. Now that we have configured and enabled
OSPF we have another port (TCP 2604) which is where we will connect to view our OSPF
configuration as well as look at some OSPF information. This is unlike a Cisco
router whereas you would view all configurations within one ssh session for
example. Quagga splits out each routing daemon as a different process and port.
So let's now connect to our OSPF daemon and take a look. (Remember our passwords
are quagga in this example.)

```bash
telnet localhost 2604
....
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.

Hello, this is Quagga (version 0.99.22.4).
Copyright 1996-2005 Kunihiro Ishiguro, et al.


User Access Verification

Password:
r1> en
Password:
r1#
```

To show our OSPF configuration.

```bash
sh run
....
Current configuration:
!
hostname r1
password 8 DdYaczUwqeugA
enable password 8 IlaeYuY8ycsaI
log file /var/log/quagga/ospfd.log
log stdout
log syslog
service password-encryption
!
debug ospf event
debug ospf packet all
!
!
interface eth0
!
interface eth1
!
interface eth2
!
interface eth3
!
interface eth4
!
interface eth5
!
interface eth6
!
interface lo
!
router ospf
 ospf router-id 192.168.250.101
 log-adjacency-changes
 redistribute connected
 network 192.168.250.101/24 area 0.0.0.51
!
line vty
!
end
```

One thing to note above is that our OSPF area is defined as 51 in this
setup.

```bash
network 192.168.250.101/24 area 0.0.0.51
```

We can adjust this OSPF area ID to anything that is required or desired
by simply adding the following variable to our `group_vars/quagga-routers` file
and changing from 51 to anything else. Feel free to experiment and see how
changing this applies to all of our routers and adjusts on the fly for us.

```yaml
quagga_ospf_area: 51
```

Let's look at our OSPF neighbors.

```bash
sh ip ospf neighbor
....
    Neighbor ID Pri State           Dead Time Address         Interface            RXmtL RqstL DBsmL
192.168.250.102   1 2-Way/DROther     32.227s 192.168.250.102 eth1:192.168.250.101     0     0     0
192.168.250.103   1 2-Way/DROther     32.167s 192.168.250.103 eth1:192.168.250.101     0     0     0
192.168.250.104   1 Full/Backup       32.208s 192.168.250.104 eth1:192.168.250.101     0     0     0
192.168.250.105   1 Full/DR           32.153s 192.168.250.105 eth1:192.168.250.101     0     0     0
```

Now let's look at a few more OSPF stats.

```bash
sh ip ospf interface
....
eth0 is up
  ifindex 2, MTU 1500 bytes, BW 0 Kbit <UP,BROADCAST,RUNNING,MULTICAST>
  OSPF not enabled on this interface
eth1 is up
  ifindex 3, MTU 1500 bytes, BW 0 Kbit <UP,BROADCAST,RUNNING,MULTICAST>
  Internet Address 192.168.250.101/24, Broadcast 192.168.250.255, Area 0.0.0.51
  MTU mismatch detection:enabled
  Router ID 192.168.250.101, Network Type BROADCAST, Cost: 10
  Transmit Delay is 1 sec, State DROther, Priority 1
  Designated Router (ID) 192.168.250.105, Interface Address 192.168.250.105
  Backup Designated Router (ID) 192.168.250.104, Interface Address 192.168.250.104
  Multicast group memberships: OSPFAllRouters
  Timer intervals configured, Hello 10s, Dead 40s, Wait 40s, Retransmit 5
    Hello due in 3.625s
  Neighbor Count is 4, Adjacent neighbor count is 2
eth2 is up
  ifindex 4, MTU 1500 bytes, BW 0 Kbit <UP,BROADCAST,RUNNING,MULTICAST>
  OSPF not enabled on this interface
eth3 is up
  ifindex 5, MTU 1500 bytes, BW 0 Kbit <UP,BROADCAST,RUNNING,MULTICAST>
  OSPF not enabled on this interface
eth4 is up
  ifindex 6, MTU 1500 bytes, BW 0 Kbit <UP,BROADCAST,RUNNING,MULTICAST>
  OSPF not enabled on this interface
eth5 is up
  ifindex 7, MTU 1500 bytes, BW 0 Kbit <UP,BROADCAST,RUNNING,MULTICAST>
  OSPF not enabled on this interface
eth6 is up
  ifindex 8, MTU 1500 bytes, BW 0 Kbit <UP,BROADCAST,RUNNING,MULTICAST>
  OSPF not enabled on this interface
lo is up
  ifindex 1, MTU 65536 bytes, BW 0 Kbit <UP,LOOPBACK,RUNNING>
  OSPF not enabled on this interface

sh ip ospf route
....
============ OSPF network routing table ============
N    192.168.250.0/24      [10] area: 0.0.0.51
                           directly attached to eth1

============ OSPF router routing table =============
R    192.168.250.102       [10] area: 0.0.0.51, ASBR
                           via 192.168.250.102, eth1
R    192.168.250.103       [10] area: 0.0.0.51, ASBR
                           via 192.168.250.103, eth1
R    192.168.250.104       [10] area: 0.0.0.51, ASBR
                           via 192.168.250.104, eth1
R    192.168.250.105       [10] area: 0.0.0.51, ASBR
                           via 192.168.250.105, eth1

============ OSPF external routing table ===========
N E2 2.2.2.0/24            [10/20] tag: 0
                           via 192.168.250.102, eth1
N E2 3.3.3.0/24            [10/20] tag: 0
                           via 192.168.250.103, eth1
N E2 4.4.4.0/24            [10/20] tag: 0
                           via 192.168.250.104, eth1
N E2 5.5.5.0/24            [10/20] tag: 0
                           via 192.168.250.105, eth1
N E2 10.0.2.0/24           [10/20] tag: 0
                           via 192.168.250.102, eth1
                           via 192.168.250.103, eth1
                           via 192.168.250.104, eth1
                           via 192.168.250.105, eth1
N E2 192.168.12.0/24       [10/20] tag: 0
                           via 192.168.250.102, eth1
N E2 192.168.14.0/24       [10/20] tag: 0
                           via 192.168.250.104, eth1
N E2 192.168.15.0/24       [10/20] tag: 0
                           via 192.168.250.105, eth1
N E2 192.168.23.0/24       [10/20] tag: 0
                           via 192.168.250.102, eth1
                           via 192.168.250.103, eth1
N E2 192.168.31.0/24       [10/20] tag: 0
                           via 192.168.250.103, eth1
                           via 192.168.250.104, eth1
N E2 192.168.41.0/24       [10/20] tag: 0
                           via 192.168.250.104, eth1
N E2 192.168.51.0/24       [10/20] tag: 0
                           via 192.168.250.105, eth1
```

Now to validate that our routes are indeed working we will run the same
ping tests we did previously. So type exit to exit our OSPF daemon.

```bash
ping -c 4 2.2.2.10
....
PING 2.2.2.10 (2.2.2.10) 56(84) bytes of data.
64 bytes from 2.2.2.10: icmp_seq=1 ttl=64 time=0.117 ms
64 bytes from 2.2.2.10: icmp_seq=2 ttl=64 time=0.362 ms
64 bytes from 2.2.2.10: icmp_seq=3 ttl=64 time=0.310 ms
64 bytes from 2.2.2.10: icmp_seq=4 ttl=64 time=0.283 ms

--- 2.2.2.10 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 2999ms
rtt min/avg/max/mdev = 0.117/0.268/0.362/0.091 ms
....
ping -c 4 192.168.23.13
....
PING 192.168.23.13 (192.168.23.13) 56(84) bytes of data.
64 bytes from 192.168.23.13: icmp_seq=2 ttl=64 time=0.382 ms
64 bytes from 192.168.23.13: icmp_seq=4 ttl=64 time=0.288 ms

--- 192.168.23.13 ping statistics ---
4 packets transmitted, 2 received, 50% packet loss, time 3016ms
rtt min/avg/max/mdev = 0.288/0.335/0.382/0.047 ms
....
ping -c 4 192.168.12.12
....
PING 192.168.12.12 (192.168.12.12) 56(84) bytes of data.
64 bytes from 192.168.12.12: icmp_seq=1 ttl=64 time=0.240 ms
64 bytes from 192.168.12.12: icmp_seq=2 ttl=64 time=0.305 ms
64 bytes from 192.168.12.12: icmp_seq=3 ttl=64 time=0.403 ms
64 bytes from 192.168.12.12: icmp_seq=4 ttl=64 time=0.455 ms

--- 192.168.12.12 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 2998ms
rtt min/avg/max/mdev = 0.240/0.350/0.455/0.086 ms
```

As you can see all of our interfaces are now responding. Feel free to
validate the additional interfaces by referencing our diagram further up
in this post. All of our interfaces should respond.

So there you have it and this concludes our auto-configure OSPF setup.
Feel free to poke around and do some testing and provide feedback based
on your findings. Remember this environment is easily repaired if
something goes wrong.

And before we end make sure to commit any changes you made to your code
here. So exit out of the ssh session you have on router1 (r1) and see if any
of your code changed.

```bash
git status
....
On branch dev
Your branch is up-to-date with 'origin/dev'.
Changes not staged for commit:
  (use "git add ..." to update what will be committed)
  (use "git checkout -- ..." to discard changes in working directory)

    modified:   .vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory
    modified:   group_vars/quagga-routers

no changes added to commit (use "git add" and/or "git commit -a")

Commit changes

git commit -am "updated variables for OSPF auto-config"
....
[dev 6074ef1] updated variables for OSPF auto-config
 2 files changed, 6 insertions(+), 3 deletions(-)
```

Now push your changes to your forked repo...Again..Remember we are
working on the dev branch.

```bash
git push
....
Counting objects: 9, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (6/6), done.
Writing objects: 100% (9/9), 740 bytes | 0 bytes/s, done.
Total 9 (delta 3), reused 0 (delta 0)
To https://github.com/everythingshouldbevirtual/vagrant-ansible-routing-template.git
   1f2dc96..6074ef1  dev -> dev
```

Up next...OSPF manual-configuration..

Enjoy!
