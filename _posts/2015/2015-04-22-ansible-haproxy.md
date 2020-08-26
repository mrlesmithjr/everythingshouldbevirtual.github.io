---
  title: Ansible - HAProxy
---

In
[this](https://everythingshouldbevirtual.com/ansible-keepalived "Ansible – KeepAliveD")
previous post I setup KeepAliveD using a fictitious tenant using
[Ansible](http://ansible.com "Ansible"). In this post I will be building
upon that same configuration and creating the HAProxy setup.

Below is the `vars/tenant_1.yml` file that contains the specific tenant
variables that will be used.
{% raw %}

```yaml
---
tenant_name: tenant_1
config_forward_rules_allow_spec: 'false'    # set to true to configure specific firewall rules under forward_rules_allow_spec

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

###### Firewall Setup ######
###### Note.....Firewall rules are by default dropped in default setup ######

# Port specific rules - used to add firewall rules that should be port based rules **Note...SSH is allowed by default so there is no need to specify rules for SSH.
# Another note is that when configuring load balancer setup further down...The ports that will be load balanced will be added to the firewall by default as well.
# This section will normally be used to allow a specific port to a host|subnet etc. which does not require load balancing.
# Make sure to change config_forward_rules_allow_spec at the top to true to ensure the below firewall rules are applied.
forward_rules_allow_spec:
  - { protocol: 'tcp', port: '8080', source: '0.0.0.0/0', destination: '192.168.70.0/24' }

# Generic non port specific rules to allow. Ex. allowing a host|subnet to communicate from|to one another on every port available.
forward_rules_allow_gen:
  - { source: '192.168.70.0/24', destination: '0.0.0.0/0' }
  - { source: '192.168.71.0/24', destination: '0.0.0.0/0' }
  - { source: '192.168.72.0/24', destination: '0.0.0.0/0' }

# Define specific rules below to be dropped by firewall. Ex. explicitly denying communications between hosts|subnets.
forward_rules_out_drop:
  - { source: '192.168.70.0/24', destination: '192.168.71.0/24' }
  - { source: '192.168.70.0/24', destination: '192.168.72.0/24' }
  - { source: '192.168.71.0/24', destination: '192.168.70.0/24' }
  - { source: '192.168.71.0/24', destination: '192.168.72.0/24' }
  - { source: '192.168.72.0/24', destination: '192.168.70.0/24' }
  - { source: '192.168.72.0/24', destination: '192.168.71.0/24' }


###### Load Balancer Setup ######

# Allocate vips from tenant_vips assigned above....these will be used as variables for lb_details and lb_defs. You may choose to just enter IP addresses though.
web_vip: '10.10.10.100'
db_vip: '10.10.10.101'
app_vip: '10.10.10.102'

balance_method: 'roundrobin'         # Set to one of the below types to configure load balancing method
#leastconn - The server with the lowest number of connections receives the connection
#roundrobin - Each server is used in turns, according to their weights.
#source - Source IP hashed and divided by total weight of servers designates which server will receive the request

# Defines the load balancing group setup
lb_details:
  - { name: 'web', protocol: 'tcp', listen_port: '80', tenant_vip: '{{ web_vip }}', balance_type: '{{ balance_method }}' }
  - { name: 'db', protocol: 'tcp', listen_port: '3306', tenant_vip: '{{ db_vip }}', balance_type: '{{ balance_method }}' }
  - { name: 'rabbitmq-mgmt', protocol: 'tcp', listen_port: '15672', tenant_vip: '{{ app_vip }}', balance_type: '{{ balance_method }}' }
  - { name: 'redis', protocol: 'tcp', listen_port: '6379', tenant_vip: '{{ app_vip }}', balance_type: '{{ balance_method }}' }
  - { name: 'rabbitmq', protocol: 'tcp', listen_port: '5672', tenant_vip: '{{ app_vip }}', balance_type: '{{ balance_method }}' }

# Defines the load balancing servers within the load balancing group
lb_defs:
  - { lb_def_name: 'web', protocol: 'tcp', listen_port: '80', tenant_vip: '{{ web_vip }}', lb_group: 'web', server: 'ans-cloud-web01', backend_port: '80' }
  - { lb_def_name: 'web', protocol: 'tcp', listen_port: '80', tenant_vip: '{{ web_vip }}', lb_group: 'web', server: 'ans-cloud-web02', backend_port: '80' }
  - { lb_def_name: 'web', protocol: 'tcp', listen_port: '80', tenant_vip: '{{ web_vip }}', lb_group: 'web', server: 'ans-cloud-web03', backend_port: '80' }
  - { lb_def_name: 'db', protocol: 'tcp', listen_port: '3306', tenant_vip: '{{ db_vip }}', lb_group: 'db', server: 'ans-cloud-db01', backend_port: '3306' }
  - { lb_def_name: 'db', protocol: 'tcp', listen_port: '3306', tenant_vip: '{{ db_vip }}', lb_group: 'db', server: 'ans-cloud-db02', backend_port: '3306' }
  - { lb_def_name: 'rabbitmq-mgmt', protocol: 'tcp', listen_port: '15672', tenant_vip: '{{ app_vip }}', lb_group: 'rabbitmq-mgmt', server: 'ans-cloud-app01', backend_port: '15672' }
  - { lb_def_name: 'rabbitmq-mgmt', protocol: 'tcp', listen_port: '15672', tenant_vip: '{{ app_vip }}', lb_group: 'rabbitmq-mgmt', server: 'ans-cloud-app02', backend_port: '15672' }
  - { lb_def_name: 'rabbitmq-mgmt', protocol: 'tcp', listen_port: '15672', tenant_vip: '{{ app_vip }}', lb_group: 'rabbitmq-mgmt', server: 'ans-cloud-app03', backend_port: '15672' }
  - { lb_def_name: 'redis', protocol: 'tcp', listen_port: '6379', tenant_vip: '{{ app_vip }}', lb_group: 'redis', server: 'ans-cloud-app01', backend_port: '6379' }
  - { lb_def_name: 'redis', protocol: 'tcp', listen_port: '6379', tenant_vip: '{{ app_vip }}', lb_group: 'redis', server: 'ans-cloud-app02', backend_port: '6379' }
  - { lb_def_name: 'redis', protocol: 'tcp', listen_port: '6379', tenant_vip: '{{ app_vip }}', lb_group: 'redis', server: 'ans-cloud-app03', backend_port: '6379' }
  - { lb_def_name: 'rabbitmq', protocol: 'tcp', listen_port: '5672', tenant_vip: '{{ app_vip }}', lb_group: 'rabbitmq', server: 'ans-cloud-app01', backend_port: '5672' }
  - { lb_def_name: 'rabbitmq', protocol: 'tcp', listen_port: '5672', tenant_vip: '{{ app_vip }}', lb_group: 'rabbitmq', server: 'ans-cloud-app02', backend_port: '5672' }
  - { lb_def_name: 'rabbitmq', protocol: 'tcp', listen_port: '5672', tenant_vip: '{{ app_vip }}', lb_group: 'rabbitmq', server: 'ans-cloud-app03', backend_port: '5672' }
```

{% endraw %}
Below is the `haproxy.cfg.j2` template that I will use.
{% raw %}

```raw
# {{ ansible_managed }}
global
#        log logstash    local0 #Change logstash to your naming
        log /dev/log    local0
        log /dev/log    local1 notice
#       log-send-hostname
        chroot /var/lib/haproxy
        user haproxy
        group haproxy
        daemon
        maxconn 40000
        spread-checks 3
        stats socket /run/haproxy/admin.sock mode 660 level admin
        stats timeout 30s

        # Default SSL material locations
        ca-base /etc/ssl/certs
        crt-base /etc/ssl/private

        # Default ciphers to use on SSL-enabled listening sockets.
        # For more information, see ciphers(1SSL).
        ssl-default-bind-ciphers kEECDH+aRSA+AES:kRSA+AES:+AES256:RC4-SHA:!kEDH:!LOW:!EXP:!MD5:!aNULL:!eNULL
        ssl-default-bind-options no-sslv3




defaults
        log     global
        mode    tcp
        maxconn 40000
        option  httplog
        option  dontlognull
        option redispatch
        option tcp-smart-accept
        option tcp-smart-connect
        retries 3
        timeout queue 5000
        timeout connect 50000
        timeout client 50000
        timeout server 50000
        errorfile 400 /etc/haproxy/errors/400.http
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http


userlist STATSUSERS
        group admin users admin
        user admin insecure-password admin

listen admin_page 0.0.0.0:9090
        mode http
        stats enable
        stats refresh 60s
        stats uri /
        acl AuthOkay_ReadOnly http_auth(STATSUSERS)
        acl AuthOkay_Admin http_auth_group(STATSUSERS) admin
        stats http-request auth realm admin_page unless AuthOkay_ReadOnly

{% for lb_group_def in lb_details %}
listen {{ tenant_name }}_{{ lb_group_def.name }}-{{ lb_group_def.tenant_vip }}:{{ lb_group_def.listen_port }} {{ lb_group_def.tenant_vip }}:{{ lb_group_def.listen_port }}
        mode tcp
        option tcpka
        option tcplog
        #balance leastconn - The server with the lowest number of connections receives the connection
        #balance roundrobin - Each server is used in turns, according to their weights.
        #balance source - Source IP hashed and divided by total weight of servers designates which server will receive the request
        balance {{ lb_group_def.balance_type }}
        default-server inter 2s downinter 5s rise 3 fall 2 slowstart 60s maxconn 256 maxqueue 128 weight 100
{% for item in lb_defs %}
{% if item.lb_group == lb_group_def.name %}
        server {{ item.server }} {{ item.server }}:{{ item.backend_port }} check
{% endif %}
{% endfor %}
{% endfor %}
```

{% endraw %}
And below is what we end up with as an haproxy.cfg for our setup by
using the variables and template above.

```bash
# Ansible managed: /home/administrator/ansible/projects/ans-cloud-rt01/templates/etc/haproxy/haproxy.cfg.j2 modified on 2015-04-10 12:58:14 by administrator on ansible
global
#        log logstash    local0 #Change logstash to your naming
        log /dev/log    local0
        log /dev/log    local1 notice
#       log-send-hostname
        chroot /var/lib/haproxy
        user haproxy
        group haproxy
        daemon
        maxconn 40000
        spread-checks 3
        stats socket /run/haproxy/admin.sock mode 660 level admin
        stats timeout 30s

        # Default SSL material locations
        ca-base /etc/ssl/certs
        crt-base /etc/ssl/private

        # Default ciphers to use on SSL-enabled listening sockets.
        # For more information, see ciphers(1SSL).
        ssl-default-bind-ciphers kEECDH+aRSA+AES:kRSA+AES:+AES256:RC4-SHA:!kEDH:!LOW:!EXP:!MD5:!aNULL:!eNULL
        ssl-default-bind-options no-sslv3




defaults
        log     global
        mode    tcp
        maxconn 40000
        option  httplog
        option  dontlognull
        option redispatch
        option tcp-smart-accept
        option tcp-smart-connect
        retries 3
        timeout queue 5000
        timeout connect 50000
        timeout client 50000
        timeout server 50000
        errorfile 400 /etc/haproxy/errors/400.http
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http


userlist STATSUSERS
        group admin users admin
        user admin insecure-password admin

listen admin_page 0.0.0.0:9090
        mode http
        stats enable
        stats refresh 60s
        stats uri /
        acl AuthOkay_ReadOnly http_auth(STATSUSERS)
        acl AuthOkay_Admin http_auth_group(STATSUSERS) admin
        stats http-request auth realm admin_page unless AuthOkay_ReadOnly

listen tenant_1_web-10.10.10.100:80 10.10.10.100:80
        mode tcp
        option tcpka
        option tcplog
        #balance leastconn - The server with the lowest number of connections receives the connection
        #balance roundrobin - Each server is used in turns, according to their weights.
        #balance source - Source IP hashed and divided by total weight of servers designates which server will receive the request
        balance roundrobin
        default-server inter 2s downinter 5s rise 3 fall 2 slowstart 60s maxconn 256 maxqueue 128 weight 100
        server ans-cloud-web01 ans-cloud-web01:80 check
        server ans-cloud-web02 ans-cloud-web02:80 check
        server ans-cloud-web03 ans-cloud-web03:80 check
listen tenant_1_db-10.10.10.101:3306 10.10.10.101:3306
        mode tcp
        option tcpka
        option tcplog
        #balance leastconn - The server with the lowest number of connections receives the connection
        #balance roundrobin - Each server is used in turns, according to their weights.
        #balance source - Source IP hashed and divided by total weight of servers designates which server will receive the request
        balance roundrobin
        default-server inter 2s downinter 5s rise 3 fall 2 slowstart 60s maxconn 256 maxqueue 128 weight 100
        server ans-cloud-db01 ans-cloud-db01:3306 check
        server ans-cloud-db02 ans-cloud-db02:3306 check
listen tenant_1_rabbitmq-mgmt-10.10.10.102:15672 10.10.10.102:15672
        mode tcp
        option tcpka
        option tcplog
        #balance leastconn - The server with the lowest number of connections receives the connection
        #balance roundrobin - Each server is used in turns, according to their weights.
        #balance source - Source IP hashed and divided by total weight of servers designates which server will receive the request
        balance roundrobin
        default-server inter 2s downinter 5s rise 3 fall 2 slowstart 60s maxconn 256 maxqueue 128 weight 100
        server ans-cloud-app01 ans-cloud-app01:15672 check
        server ans-cloud-app02 ans-cloud-app02:15672 check
        server ans-cloud-app03 ans-cloud-app03:15672 check
listen tenant_1_redis-10.10.10.102:6379 10.10.10.102:6379
        mode tcp
        option tcpka
        option tcplog
        #balance leastconn - The server with the lowest number of connections receives the connection
        #balance roundrobin - Each server is used in turns, according to their weights.
        #balance source - Source IP hashed and divided by total weight of servers designates which server will receive the request
        balance roundrobin
        default-server inter 2s downinter 5s rise 3 fall 2 slowstart 60s maxconn 256 maxqueue 128 weight 100
        server ans-cloud-app01 ans-cloud-app01:6379 check
        server ans-cloud-app02 ans-cloud-app02:6379 check
        server ans-cloud-app03 ans-cloud-app03:6379 check
listen tenant_1_rabbitmq-10.10.10.102:5672 10.10.10.102:5672
        mode tcp
        option tcpka
        option tcplog
        #balance leastconn - The server with the lowest number of connections receives the connection
        #balance roundrobin - Each server is used in turns, according to their weights.
        #balance source - Source IP hashed and divided by total weight of servers designates which server will receive the request
        balance roundrobin
        default-server inter 2s downinter 5s rise 3 fall 2 slowstart 60s maxconn 256 maxqueue 128 weight 100
        server ans-cloud-app01 ans-cloud-app01:5672 check
        server ans-cloud-app02 ans-cloud-app02:5672 check
        server ans-cloud-app03 ans-cloud-app03:5672 check
```

Below is the group_vars code used across the different Ansible posts
based on tenant setup

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
