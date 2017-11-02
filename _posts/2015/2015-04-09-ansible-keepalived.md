---
  title: Ansible - KeepAliveD
---

In this post I am creating a fictitious tenant using a vars file which
looks like below.

`tenant_1.yml`:
{% raw %}

```yaml
---
tenant_name: tenant_1
config_forward_rules_allow_spec: false

tenant_subnets:
  - { tenant_subnet: '192.168.70.0/24' }  # Web
  - { tenant_subnet: '192.168.71.0/24' }  # App
  - { tenant_subnet: '192.168.72.0/24' }  # DB

tenant_vips:
  - 10.10.10.100
  - 10.10.10.101
  - 10.10.10.102
  - 10.10.10.103
  - 10.10.10.104

# Applied on router(s) forward rules
forward_rules_allow_spec:
#  - { protocol: 'tcp', port: '8080', source: '0.0.0.0/0', destination: '192.168.70.0/24' }
#  - { protocol: 'tcp', port: '22', source: '0.0.0.0/0', destination: '192.168.71.0/24' }
#  - { protocol: 'tcp', port: '22', source: '0.0.0.0/0', destination: '192.168.72.0/24' }

forward_rules_allow_gen:
  - { source: '192.168.70.0/24', destination: '0.0.0.0/0' }
  - { source: '192.168.71.0/24', destination: '0.0.0.0/0' }
  - { source: '192.168.72.0/24', destination: '0.0.0.0/0' }

forward_rules_out_drop:
  - { source: '192.168.70.0/24', destination: '192.168.71.0/24' }
  - { source: '192.168.70.0/24', destination: '192.168.72.0/24' }
  - { source: '192.168.71.0/24', destination: '192.168.70.0/24' }
  - { source: '192.168.71.0/24', destination: '192.168.72.0/24' }
  - { source: '192.168.72.0/24', destination: '192.168.70.0/24' }
  - { source: '192.168.72.0/24', destination: '192.168.71.0/24' }

backend_rules:

# Load Balancer Setup

lb_details:
  - { name: 'web', protocol: 'tcp', listen_port: '80', tenant_vip: '10.10.10.100' }
  - { name: 'db', protocol: 'tcp', listen_port: '3306', tenant_vip: '10.10.10.100' }

lb_defs:
  - { lb_def_name: 'web', protocol: 'tcp', listen_port: '80', tenant_vip: '10.10.10.100', lb_group: 'web', server: 'ans-cloud-web01', backend_port: '80' }
  - { lb_def_name: 'web', protocol: 'tcp', listen_port: '80', tenant_vip: '10.10.10.100', lb_group: 'web', server: 'ans-cloud-web02', backend_port: '80' }
  - { lb_def_name: 'web', protocol: 'tcp', listen_port: '80', tenant_vip: '10.10.10.100', lb_group: 'web', server: 'ans-cloud-web03', backend_port: '80' }
  - { lb_def_name: 'db', protocol: 'tcp', listen_port: '3306', tenant_vip: '10.10.10.100', lb_group: 'db', server: 'ans-cloud-db01', backend_port: '3306' }
  - { lb_def_name: 'db', protocol: 'tcp', listen_port: '3306', tenant_vip: '10.10.10.100', lb_group: 'db', server: 'ans-cloud-db02', backend_port: '3306' }
```

{% endraw %}

And the goal of this is to dynamically create a KeepAliveD Failover
configuration between two Ubuntu servers functioning as routers and load
balancers. I will be using the following playbook to generate the KeepAliveD
configuration.

```yaml
---
- hosts: all
  vars_files:
    - ../vars/tenant_1.yml
  tasks:
  - name: tenant_configs | config | setting up keepalived vips
    template: src=../templates/etc/keepalived/keepalived.conf.j2 dest=/etc/keepalived/keepalived.conf owner=root group=root mode=0644
    run_once: true
    notify: restart keepalived
  handlers:
  - name: restart keepalived
    service: name=keepalived state=restarted
```

The playbook will use the following template to actually generate the
keepalived.conf file.
{% raw %}

```yaml
# {{ ansible_managed }}

#### Below is for managing the whole stack involved in Quagga HA
vrrp_script chk_zebra {
   script "killall -0 zebra"   # verify the pid existance
   interval 2                    # check every 2 seconds
   weight 2                      # add 2 points of prio if OK
}

vrrp_instance Quagga {
  state MASTER
  interface {{ keepalived_vip_int }}
  virtual_router_id {{ keepalived_router_id }}
  priority {{ keepalived_router_pri }}
  advert_int 1
  virtual_ipaddress {
    {{ keepalived_vip }}
  }
  virtual_ipaddress_excluded {
{% for vip in tenant_vips %}
          {{ vip }}
{% endfor %}
  }
  notify_master {{ notify_master_script }}
  notify_backup {{ notify_backup_script }}
}
```

{% endraw %}
And what you end up with is below

```bash
#### Below is for managing the whole stack involved in Quagga HA
vrrp_script chk_zebra {
   script "killall -0 zebra"   # verify the pid existance
   interval 2                    # check every 2 seconds
   weight 2                      # add 2 points of prio if OK
}

vrrp_instance Quagga {
  state MASTER
  interface eth0
  virtual_router_id 23
  priority 101
  advert_int 1
  virtual_ipaddress {
    10.10.10.4
  }
  virtual_ipaddress_excluded {
          10.10.10.100
          10.10.10.101
          10.10.10.102
          10.10.10.103
          10.10.10.104
  }
  notify_master /opt/scripts/master.sh
  notify_backup /opt/scripts/backup.sh
}
```

And there you have it. You can now build out your KeepAliveD
configurations.

Below is the actual complete group_vars file. Which will be use for
this post and future posts based on this same scenario.

{% raw %}

```yaml
---
# Tenants
config_tenants: true

# Firewall
enable_firewall: 'true'                   # set to true to enable firewall services
nat_masquerade: 'false'

# Zabbix Monitoring
enable_zabbix_agent: 'true'

# Define configurations for each service
enable_conntrackd: true                 # set to true to enable conntrackd (connection tracking)
enable_dnsmasq: true                     # set to true to enable service
enable_haproxy: true                    # set to true to enable service
enable_keepalived: true                 # set to true to enable service
enable_quagga: true                     # set to true to enable service
enable_snmpd: true                       # set to true to enable service
enable_tftp: true                       # set to true to enable service

# Before setting these to true for config they must be set to true for enable above
config_conntrackd: true                  # set to true to config conntrackd (setup connection tracking sync - Works with KeepAliveD)
config_dnsmasq: true                     # set to true to do custom config
config_glusterfs: true                   # config glusterfs - sets up GlusterFS mounts, bricks and mounts various mountpoints if configured
config_haproxy: true                    # set to true to do custom config
config_hosts: '{{ config_glusterfs }}'   # update /etc/hosts true|false - needs to be set to true for glusterfs if DNS hostnames do not exist for nodes being setup
config_keepalived: true                 # set to true to do custom config
config_lvm: true                         # set to true to do custom config
config_quagga: true                     # set to true to do custom config
config_snmpd: true                       # set to true to do custom config
config_tftp: true                        # set to true to do custom config

# Before setting these to true for sync they must be set to true for config above
sync_dnsmasq: true                       # set to true to sync /var/lib/misc which contains dhcpleases
sync_haproxy: true                       # set to true to sync between nodes using GlusterFS
sync_keepalived: true                    # set to true to sync between nodes using GlusterFS
sync_quagga: true                       # set to true to sync between nodes using GlusterFS
sync_tftp: true                          # set to true to sync between nodes using GlusterFS

# Define LVM configuration for additional disk added for GlusterFS
create: true
create_vgname: 'glusterfs-vg'
create_lvname: 'glusterfs-lv'
create_lvsize: '100%FREE'
new_mntp: '/mnt/gluster'
new_disk: '/dev/sdb'
filesystem: xfs

# Reconfigure networking on ititial setup
pre_bootstrap_change_ip: true            # changes ip of each node based on host_vars ip setting

# Bootstrap setup
reboot: true                             # reboot after changing hostname to match inventory_hostname - set to false if you do not want to reboot
root_password: '$1$lDZD3dKn$ZRoapxlZOzivK/sQWqdFg/'             # MD5 hash of your root password
administrator_password: '$1$WcS2byl4$hU0dn5RtQsCbRWiorXsTL.'    # MD5 hash of your administrator password - if used
remote_password: '$1$D3pAE14D$HSv0wd9jK9P.bt/vMcOv6.'           # MD5 hash of your remote password - used for Ansible

# Rsyslog setup
syslog_server_1: 'logstash.everythingshouldbevirtual.local'
syslog_server_port_1: '514'
syslog_server_2: 'logstash-dev.everythingshouldbevirtual.local'
syslog_server_port_2: '514'

# Timezone setup
change_timezone: true
timezone: 'America/New_York'             # change to your preferred timezone - UTC|EST5EDT|America/New_York

# SNMPD configurations
snmpd_ro_community: 'everythingshouldbevirtual'   # set to your snmp RO (read only) community string
snmpd_authorized_network: '10.0.101.0/24'         # set to your network in which your snmp monitoring server(s) reside

# sets up networking - used througout various settings for additional services if they can be
pri_bind_interface: 'eth0'          # should be set to your primary interface
pri_interface_method: 'static'      # set to static or dhcp
pri_netmask: '255.255.255.0'        # set to the netmask for your primary interface (eth0)
pri_netmask_cidr: '24'              # set to the CIDR format for your netmask (255.0.0.0=8,255.255.0.0=16,255.255.255.0=24, or other)
pri_network: '10.10.10.0'           # set to the network that your primary interface resides...last octet 0 for network address match
pri_gateway: '10.10.10.3'           # set to the primary interface gateway...this will also setup the ospf default route
pri_dns: '10.0.101.111'             # set to your primary dns server
sec_bind_interface: 'eth1'          # should be set to your secondary interface - which is providing DHCP and TFTP services
sec_interface_method: 'static'      # set to static or DHCP - this interface will|should be disabled in /etc/network/interfaces...used for VLAN(s) config - Inside interface
sec_netmask: '255.255.255.0'        # should be left unconfigured
sec_network: ''                     # should be left unconfigured
sec_gateway: ''                     # should be left unconfigured
sec_dns: '10.0.101.112'             # set to your secondary dns server
pri_domain_name: 'everythingshouldbevirtual.local'           # set to your primary domain name
dnsmasq_domain: 'cloud.everythingshouldbevirtual.local'      # use for provisioning of secondary domain for your cloud - can be the same as pri_domain_name
dns_nameservers: '{{ pri_dns }} {{ sec_dns }}'                 # add your dns servers for upstream resolution
dns_search: '{{ pri_domain_name}} {{ dnsmasq_domain }}'      # sets your dns search suffix

#### Below is for setting up /etc/network/interfaces
interfaces_lo:
  - { int: lo, method: loopback, ip_address: '{{ quagga_ospf_routerid }}/32' }      # configures the loopback adapter for the OSPF router ID

# Sets up /etc/network/interfaces
interfaces_config:
  - { int: '{{ sec_bind_interface }}', method: '{{ sec_interface_method }}' }
  - { int: '{{ conntrackd_sync_int }}', method: 'static' }

#### Below is for both /etc/network/interfaces.d and quagga - add your VLAN(s) and IP addresses for routing
vlan_config:
  - { vlan: vlan110, method: manual, raw_device: '{{ sec_bind_interface }}', ip_address: '10.0.110.1/24', network: '10.0.110.0/24' }
  - { vlan: vlan700, method: manual, raw_device: '{{ sec_bind_interface }}', ip_address: '192.168.70.1/24', network: '192.168.70.0/24' }
  - { vlan: vlan701, method: manual, raw_device: '{{ sec_bind_interface }}', ip_address: '192.168.71.1/24', network: '192.168.71.0/24' }
  - { vlan: vlan702, method: manual, raw_device: '{{ sec_bind_interface }}', ip_address: '192.168.72.1/24', network: '192.168.72.0/24' }
  - { vlan: vlan703, method: manual, raw_device: '{{ sec_bind_interface }}', ip_address: '192.168.73.1/24', network: '192.168.73.0/24' }

#### Below is for setting up quagga
passive_int:
  - { int: default }      # sets all interfaces to passive in OSPF config by default

no_passive_int:
  - { int: lo }                           # sets to no passive so OSPF will communicate
  - { int: '{{ pri_bind_interface }}' }   # sets to no passive so OSPF will communicate

ospf_area_config:
  - { network: '{{ pri_network }}/{{ pri_netmask_cidr }}', area: '{{ quagga_ospf_area }}' }      # configures OSPF network area
  - { network: '{{ quagga_ospf_routerid }}/32', area: '{{ quagga_ospf_area }}' }

ospf_redistribute:
  - connected
  - kernel
  - static
# - isis
# - rip

### Enter default route for networks not found in routing table
default_route: '0.0.0.0/0 {{ pri_gateway }}'      # configures the default route for Quagga...This will typically be set to your default gateway for your pri interface

# Define TFTP settings
tftp_bind_address: "10.0.110.1"                 # set to an interface IP address on your inside network - vlan700 IP address here
tftpboot_dir: '/var/lib/tftpboot'                 # set to the tftpboot directory
tftpboot_home: '{{ tftpboot_dir }}'               # this should be the same as tftpboot_dir....used for GlusterFS mount
tftpboot_mnt: '{{ gfs_dir5 }}-{{ app_name }}'     #
tftpboot_backup_dir: '/var/lib/tftpboot.backup'

# Define DNSmasq settings
no_dhcp_bind_int: '{{ pri_bind_interface }}' # change to interface that you do not want DHCP listening on - should be outside interface (eth0)
time_server: '10.10.10.1'
domain_suffix_search: '{{ pri_domain_name }},{{ dnsmasq_domain }}'
dhcp_boot: 'pxelinux.0,{{ inventory_hostname }},{{ tftp_bind_address }}'
netboot_url: 'http://archive.ubuntu.com/ubuntu/dists/trusty-updates/main/installer-amd64/current/images/netboot/'
netboot_file: 'netboot.tar.gz'
dnsmasq_misc_backup_dir: '/var/lib/misc.backup'
dnsmasq_misc_mnt: '{{ gfs_dir6 }}-{{ app_name }}'
dnsmasq_misc_home: '/var/lib/misc'
dnsmasq_nameservers:
  - '{{ pri_dns }}'
  - '{{ sec_dns }}'

# Define preseed.cfg defaults (TFTP)
#domain_name: '{{ pri_domain_name }}'              # your primary domain name or secondary domain name - whichever suits your requirements
domain_name: '{{ dnsmasq_domain }}'              # comment out above and uncomment this line if you want to use your cloud domain for provisioning vs. primary domain name
root_pw: '$1$6Rpvg6ad$VUpMRxLIDXbNp3of9RjGE0'     # use echo "password" | mkpasswd -s -5
bind_address: '{{ tftp_bind_address }}'            # sets to your sec_bind_address - change to pri_bind_address if you are setting TFTP on your primary interface

# Define DNSmasq DHCP ranges
dhcp_range:
  - { start: '192.168.70.128', end: '192.168.70.224', netmask: '255.255.255.0' }
  - { start: '192.168.71.128', end: '192.168.71.224', netmask: '255.255.255.0' }
  - { start: '192.168.72.128', end: '192.168.72.224', netmask: '255.255.255.0' }
  - { start: '192.168.73.128', end: '192.168.73.224', netmask: '255.255.255.0' }

# Define app name to use for setup of GlusterFS
app_name: 'ans-cloud-rt'

# Define names for GlusterFS directories and volumes
gfs_dir1: 'loadbalancers'
gfs_vol1: '{{ app_name }}'
gfs_dir2: 'routers'
gfs_vol2: '{{ app_name }}'
gfs_dir3: 'scripts'
gfs_vol3: '{{ app_name }}'
gfs_dir4: 'interfaces'
gfs_vol4: '{{ app_name }}'
gfs_dir5: 'tftpboot'
gfs_vol5: '{{ app_name }}'
gfs_dir6: 'dnsmasq_misc'
gfs_vol6: '{{ app_name }}'

# Define folders to create for GlusterFS
create_gluster_bricks:
  - { name: '{{ gfs_dir1 }}', owner: root, group: root }
  - { name: '{{ gfs_dir2 }}', owner: root, group: root }
  - { name: '{{ gfs_dir3 }}', owner: root, group: root }
  - { name: '{{ gfs_dir4 }}', owner: root, group: root }
  - { name: '{{ gfs_dir5 }}', owner: root, group: root }
  - { name: '{{ gfs_dir6 }}', owner: root, group: root }

# Define GlusterFS volumes to create
create_gluster_volumes:
  - { name: '{{ gfs_dir1 }}-{{ gfs_vol1 }}', brick: '{{ gluster_brick_dir }}/{{ gfs_dir1 }}/{{ gfs_vol1 }}', rebalance: yes, replicas: 2 }
  - { name: '{{ gfs_dir2 }}-{{ gfs_vol2 }}', brick: '{{ gluster_brick_dir }}/{{ gfs_dir2 }}/{{ gfs_vol2 }}', rebalance: yes, replicas: 2 }
  - { name: '{{ gfs_dir3 }}-{{ gfs_vol3 }}', brick: '{{ gluster_brick_dir }}/{{ gfs_dir3 }}/{{ gfs_vol3 }}', rebalance: yes, replicas: 2 }
  - { name: '{{ gfs_dir4 }}-{{ gfs_vol4 }}', brick: '{{ gluster_brick_dir }}/{{ gfs_dir4 }}/{{ gfs_vol4 }}', rebalance: yes, replicas: 2 }
  - { name: '{{ gfs_dir5 }}-{{ gfs_vol5 }}', brick: '{{ gluster_brick_dir }}/{{ gfs_dir5 }}/{{ gfs_vol5 }}', rebalance: yes, replicas: 2 }
  - { name: '{{ gfs_dir6 }}-{{ gfs_vol6 }}', brick: '{{ gluster_brick_dir }}/{{ gfs_dir6 }}/{{ gfs_vol6 }}', rebalance: yes, replicas: 2 }

# Define KeepAliveD settings (router_id for nodes is in host_vars)
keepalived_vip: '10.10.10.4'                     # Should be setup on the same network as your pri_interface resides
keepalived_vip_int: '{{ pri_bind_interface }}'   # Set to primary bind interface
keepalived_router_id: '23'   # make sure to set these to different values so keepalived does not attempt to join additional vrrp domains on the same subnet
notify_master_script: '/opt/scripts/master.sh'
notify_backup_script: '/opt/scripts/backup.sh'
notify_fault_script: '/opt/scripts/fault.sh'
scripts_mnt: '{{ gfs_dir3 }}-{{ app_name }}'
scripts_home: '/opt/scripts'

# Define conntrackd settings for use with KeepAliveD
conntrackd_ignore_addresses:
  - '{{ pri_bind_address }}'
  - '{{ conntrackd_sync_ip }}'
  - '{{ keepalived_vip }}'
conntrackd_sync_int: 'eth2'                 # Use a separate interface than used for primary and VLAN interface
conntrackd_sync_netmask: '255.255.255.0'    # Netmask for sync network

# Define GlusterFS settings
glusterfs_client: true        # true|false - should be set to true in most cases
glusterfs_server: true        # true|false - should be set to true in most cases
glusterfs_repl_int: '{{ pri_bind_interface }}'
gluster_brick_dir: '{{ new_mntp }}'
cluster_hosts: 'ans-cloud-rt01a,ans-cloud-rt01b'      # make sure to change these to match your inventory_hostname values of your hosts in site_hosts and also add them to a site_group

# Define HAProxy settings
haproxy_backup_dir: '/etc/haproxy.backup'
haproxy_home: '/etc/haproxy'
haproxy_mnt: '{{ gfs_dir1 }}-{{ app_name }}'

# Define Quagga settings
# if the mgmt int is changed you need to update playbooks/glusterfs.yml under updating /etc/hosts to reflect the correct eth device until a variable solution can be found
quagga_mgmt_int: '{{ pri_bind_interface }}'           # should be set to your outside interface (eth0)
quagga_mgmt_method: '{{ pri_interface_method }}'      # static|dhcp
quagga_mgmt_netmask: '{{ pri_netmask }}'              # primary interface netmask
quagga_mgmt_gateway: '{{ pri_gateway }}'              # primary interface gateway
quagga_mgmt_nameservers: '{{ dns_nameservers }}'      # upstream dns servers
quagga_mgmt_dns_search: '{{ dns_search }}'            # dns search suffix
quagga_enable_zebra: 'yes'                            # enables main Quagga service
quagga_enable_ospfd: 'yes'                            # enables Quagga OSPF service
quagga_enable_vtysh: 'yes'                            # enables vty terminal acesss
quagga_backup_dir: /etc/quagga.backup                 # where to move the existing /etc/quagga directory
quagga_home: '/etc/quagga'                            # where to mount the new quagga home directory
quagga_hostname: quagga-rt01                          # set to the hostname to be configured within Quagga router
quagga_password: quagga                               # set to your preferred password for Quagga router login
quagga_enable_password: quagga                        # set to your preferred enable password for Quagga router configurations
quagga_ospf_routerid: '172.16.0.41'                   # IP address to assign as the OSPF router ID...This IP will be bound to the loopback adapter
#quagga_ospf_distribute: 'connected'                   # can be set to the following options  - babel|bgp|connected|isis|kernel|rip|static
quagga_ospf_area: '0'                                 # set to the desired area mapping for OSPF routing with upstream OSPF routers
quagga_mnt: '{{ gfs_dir2 }}-{{ app_name }}'
net_config_dir: '/etc/network/interfaces.d'

# Interfaces GlusterFS - for replicating network VLAN interfaces *currently not working*
interfaces_home: '/etc/network/interfaces.d'
interfaces_mnt: '{{ gfs_dir4 }}-{{ app_name }}'
sync_interfaces: false
```

{% endraw %}

Enjoy!
